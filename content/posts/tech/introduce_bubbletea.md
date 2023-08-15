---
title: "簡単にTUIが作れる！Bubble Teaのススメ(アーキテクチャ編)"
description: "Bubble Teaは、TUIを作成するためのGo言語のライブラリです。この記事では、Bubble Teaのアーキテクチャを紹介します。"
author: "tbistr"
date: 2023-08-14T20:58:51Z

# tags: ["emoji"]
categories: ["golang", "bubbletea"]
# series: ["Themes Guide"]
---

## Bubble Teaとは

[Bubble Tea](https://github.com/charmbracelet/bubbletea)は、Text User Interface(TUI)を作成するためのGo言語のライブラリ(フレームワーク)です。
以下のようなCLIアプリケーションを、比較的簡単に作成できます。

<img src="https://stuff.charm.sh/bubbletea/bubbletea-example.gif" width="100%" alt="Bubble Tea Example">

[Bubble TeaのGitHubリポジトリより引用](https://github.com/charmbracelet/bubbletea/blob/master/README.md)

Bubble Teaはフレームワーク的な部分があるため、アーキテクチャを理解することが重要です。
(理解しないと使えません。)
この記事では、Bubble Teaのアーキテクチャについて紹介します。

## Elmアーキテクチャ

Bubble Teaの中心となる考え方は、Elmアーキテクチャに基づいています。
[Elmアーキテクチャ](https://guide.elm-lang.jp/architecture/)は、Elmという関数型のAltJS言語で考案された、Webフロントエンドのアーキテクチャです。

![Elmアーキテクチャ](/images/elm_arch.png)

Elmアーキテクチャは、以下の3つの要素から構成されます。

- Model: アプリケーションの状態を表すデータ
- View: Modelの状態を表示するための関数(ElmではHTMLを生成)
- Update: Modelを更新するための関数

ユーザが何か入力したり、イベントが発生すると、ランタイムはUpdateを呼び出します。
このとき渡されるmsgは、入力やイベントを表すデータ構造です。
Updateはmsgの種類・内容に応じてModelを更新します。
(Updateはイベントハンドラのようなものですね。)

Update, Viewが純粋関数であることにより、テストが容易になったりというメリットがありそうです。
この辺は関数型っぽいですね。

## Bubble Teaのアーキテクチャ

Bubble TeaはElmアーキテクチャをCLIのためにGo言語で実装したものと考えてもらってOKです。

Bubble Teaアプリとして実行するためには、以下のintefaceが実装されている必要があります。

```golang
type Model interface {
    Init() Cmd
    Update(Msg) (Model, Cmd)
    View() string
}
```

Viewを見ると、ElmのViewと同じようにModelから文字列(CUI)を生成する関数であることがわかります。
一方、UpdateはElmのものと少し異なり、Msgを受け取ってModelとCmdを返しています。
Modelを返しているのは、Updateが値呼びの関数であるためです。
変更後のModelを返すことでModelの変更を実現しています。

### CmdとMsg

Cmd、Msgの定義は以下の通りです。

```golang
type Msg interface{}
type Cmd func() Msg
```

Msgはinterface{}なので、何でもアリです。
CmdはMsgを返す関数なので、これも何でもアリの関数になります。
型にはほぼ情報が無いですね。

実際には、Cmdは主にI/O処理を実行するために使われます。
Updateから返されたCmdは、Bubble Teaランタイムによってgroutineとして非同期に実行されます。

Msgは、Updateの引数になっていることからもわかるように、ユーザの入力やイベントを表すデータです。

具体的な例を見てみましょう。

#### Msgの種類

Bubble Teaでは、以下のようなMsgが定義されています。

- tea.KeyMsg: キー入力
- tea.WindowSizeMsg: ウィンドウサイズ変更
- tea.MouseMsg: マウスイベント

これらのMsgは、ランタイムからUpdateに渡されます。
一方ユーザが定義するMsgには、以下のようなものがあります。

- I/O処理(Cmd)の結果を表すMsg
- エラーの発生を表すMsg
- 時間経過によるTickを表すMsg

#### Cmdの種類

Cmdは、基本的にはユーザが定義する関数です。
以下のようなCmdが考えられます。

- HTTPリクエストを送信、結果のMsgを返すCmd
- ファイルを読み込み、grepした結果のMsgを返すCmd
- 現在時刻を取得し、Msgにして返すCmd

また、Bubble Teaが提供するCmdもあります。

- tea.Batch: 複数のCmdを実行するCmd
- tea.Exec: 別のプログラムを実行するCmd
- tea.Quit: プログラムを終了するCmd

### Updateの構造

CmdとMsgの説明を踏まえると、Updateの構造は図のようになります。
Updateに入力されるMsgは、ユーザの入力やイベントを表すMsgと、I/O処理の結果を表すMsgの2種類があります。

![Bubble Tea Update](/images/bubbletea_update.png)

このCmd、Msgのループを考えると、Initの意味もわかると思います。
Initはアプリケーション開始時にどんなCmdを実行するかを指定します。

例えば、ページングがあるAPIを実行したい場合、ページ0のデータを取得するCmdをInitに指定します。
API実行結果のMsgを受け取ったら、次のページを取得するCmdを返すようにUpdateに定義します。
これで、毎回完了を待ちつつページングを順番に実行できます。

## サンプルアプリ

Bubble Teaのサンプルアプリを作ってみました。

```golang
package main

import (
    "fmt"

    tea "github.com/charmbracelet/bubbletea"
)

type model struct {
    counter  int
    typedKey string
}

var _ tea.Model = model{}

func (m model) Init() tea.Cmd {
    return nil
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch msg.String() {
        case "ctrl+c":
            return m, tea.Quit
        default:
            m.typedKey = msg.String()
            m.counter++
        }
    }

    return m, nil
}

func (m model) View() string {
    return fmt.Sprintf("You typed: %s\nCounter is: %d\nCtrl+C to exit", m.typedKey, m.counter)
}

func main() {
    m := model{}
    p := tea.NewProgram(m)
    p.Run()
}
```

実行すると、入力したキーと、入力の回数が表示されます。
詳細な説明は宿題にしてみますが、Bubble Teaのコードリーディングには以下のようなコツがあります。

- まずModelが保持している状態と、Viewの処理内容を見る
- Updateを見て、処理するMsgの種類を把握する
- Msgの型ごとに処理を見ていく

## まとめ

Bubble Teaでは、こんな感じで状態を管理しつつCmdを裏で非同期実行することで、カクツキのないCUIアプリケーションを作ることができます。
一方、トップに載せたGifようなカラフルでアニメーションがあるアプリを作るのは、この記事で説明した機能だけでは難しいです。

Bubble Teaの開発チームであるCharmが作っている、Lipglossというライブラリを使うことで、文字や背景に色をつけたり枠線を書いたりと、美麗なテキストをViewで生成できます。
また、Bubblesというコンポーネント集も提供されており、組み合わせることでリッチなUIを簡単に実現で行きます。
この辺の説明は、次回以降の記事でやっていきたいと思います。

今後以下のような記事を書いていく予定です。

- デモアプリを使ったコード中心の説明
- Lipglossの紹介
- Bubblesの紹介
- モデル定義のコツ

### 先行記事

日本語でのBubble Teaの紹介記事があまりなかったのでこの記事を書いてみました。
少ないですが、以下のような記事があります。

- https://motemen.hatenablog.com/entry/2022/06/introduction-to-go-bubbletea
- https://zenn.dev/yuzuy/articles/95e522a39a5423f5bff4
