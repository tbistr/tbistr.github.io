---
title: "リッチなターミナル描画を実現！ Lip Glossのススメ(ざっくりできること編)"
description: ""
author: "tbistr"
date: 2023-10-15T14:53:26Z

# tags: ["emoji"]
categories: ["golang", "bubbletea", "lipgloss"]
# series: ["Themes Guide"]

---

この記事では、Lip Glossでできること、その方法をざっくり紹介します。

## Lip Gloss(lipgloss) とは

Lip Glossは、Goで書かれたTUI用のスタイリングライブラリです。
以下のような装飾をターミナルのテキストに適用できます。

<img src="https://camo.githubusercontent.com/5ed63e6b61ddcea9575c0b0eefd373c8f085f4594e323fe60c39ce3bbcc27d81/68747470733a2f2f73747566662e636861726d2e73682f6c6970676c6f73732f6c6970676c6f73732d6578616d706c652e706e67" width="100%" alt="Lip Gloss Example">

[Lip GlossのGitHubリポジトリより引用](https://camo.githubusercontent.com/5ed63e6b61ddcea9575c0b0eefd373c8f085f4594e323fe60c39ce3bbcc27d81/68747470733a2f2f73747566662e636861726d2e73682f6c6970676c6f73732f6c6970676c6f73732d6578616d706c652e706e67)

以下のような項目のスタイリングに対応しています。

- 文字色
- 背景色
- 文字寄せ
- 文字囲み
- 特定幅での折返し
- などなど...

以前紹介したBubble Teaを提供しているCharmというチームが開発しています。
そのため、Bubble Teaを用いたTUIアプリのスタイリングに凝りたい場合、Lip Glossが第一候補になります。

うまく使いこなすと、こんな感じのアプリを作れます。
(GitHubのリポジトリを検索するTUIアプリ)

<img src="https://raw.githubusercontent.com/tbistr/github-explorer/main/demo.gif" width="100%" alt="GitHub Explorer">

## Lip Glossの各機能の紹介

できることと、そのコード例をセットで紹介します。

### `Style`型

`Style`型はlipglossのメインの型です。
メソッドチェインでスタイルを設定します。
`Render`メソッドを呼び出すと、渡した文字列にスタイルが適用されます。

(各スタイルについては後述します)

```go
fmt.Println("# lipglossでは、style型を使って文字列を装飾します")
var style = lipgloss.NewStyle().
    Bold(true).
    Italic(true).
    Foreground(lipgloss.Color("#0079FF")).
    Background(lipgloss.Color("#F6FA70")).
    Width(40).
    Align(lipgloss.Center)

fmt.Println("## style型のRenderメソッドに文字列を渡すと、装飾された文字列が返ります")
fmt.Println(style.Render("Hello, lipgloss!"))
```

結果はこのようになります。

![style.Renderの結果](/images/introduce_lipgloss/style.png)

### 文字、背景色の設定

`Color`型はlipglossで色を表現する型です。
`Style`型の`Foreground`メソッド、`Background`メソッドに渡すことで、文字色、背景色を設定できます。

基本的にはカラーコードで指定するとエディタのハイライトが効いたりと便利ですが、ANSIの4bit, 8bitカラーも使えます。

```go
fmt.Println("# lipglossを使って文字、背景に色をつけることができます")

fmt.Println("## Color型を使って色を作成できます")
blue := lipgloss.Color("#0079FF")
// 24bitカラー以外にも、4bit, 8bitのカラーも使えます
// lipgloss.Color("5")

fmt.Println("## style型のForeground, Backgroundメソッドには、Color型で色を指定します")
fmt.Println(
    lipgloss.NewStyle().
        Foreground(lipgloss.Color("#F6FA70")).
        Background(blue).
        Render("Colored text!"),
)
```

結果はこのようになります。

![文字、背景色の設定](/images/introduce_lipgloss/color.png)

### テキストの形式設定

テキストをイタリック体、反転、太字などにできます。
それぞれ、`Italic`, `Reverse`, `Bold`, `Underline`メソッドを使います。
(フォントが対応していない場合、通常表示になります)

```go
fmt.Println("# テキストの形式を指定することもできます")
fmt.Println(lipgloss.NewStyle().
    Bold(true).Render("Bold text."))
fmt.Println(lipgloss.NewStyle().
    Italic(true).Render("Italic text."))
fmt.Println(lipgloss.NewStyle().
    Reverse(true).Render("Reversed text."))
fmt.Println(lipgloss.NewStyle().
    Underline(true).Render("Underlined text."))
```

結果はこのようになります。

![text_style](/images/introduce_lipgloss/text_style.png)

### パディング、マージンの設定

CSSのように、テキストの周りに余白を設けることができます。
`Padding`メソッド、`Margin`メソッドを使います。

`Padding~`, `Margin~`メソッドを使うと、上下左右別々に値を指定できます。
また、`Padding`, `Margin`メソッドにCSSと同じ書式でまとめて指定できます。
(例以外にも色々なパターンで渡せます)

```go
fmt.Println("# CSSのように、パディングやマージンを指定できます")

// 上下左右に2文字分パディング
fmt.Println(lipgloss.NewStyle().Background(blue).
    Padding(2).Render("Padding text."))
// 上下左右に2文字分マージン
fmt.Println(lipgloss.NewStyle().Background(blue).
    Margin(2).Render("Margin text."))

fmt.Println("## 上下左右に別々の値を指定することもできます")
fmt.Println(
    lipgloss.NewStyle().Background(blue).
        PaddingTop(1).
        PaddingRight(2).
        PaddingBottom(1).
        PaddingLeft(2).
        Render("Complex padding text."),
)
fmt.Println(
    lipgloss.NewStyle().Background(blue).
        Margin(1, 2, 1, 2). // まとめることもできます(CSSと同じ書式)
        Render("Complex margin text."),
)
```

結果はこのようになります。

![Paddin, margin](/images/introduce_lipgloss/padding.png)

### 高さ、幅の設定

`Width`, `Height`メソッドを使うと、文字列を描画する領域のサイズを指定できます。
文字列がはみ出ると自動で折り返されます。

また`MaxWidth`, `MaxHeight`メソッドを使うと、文字列を描画する領域の最大サイズを指定できます。

```go
fmt.Println("# 文字列を描画する領域のサイズを指定できます")
fmt.Println(
    lipgloss.NewStyle().Background(blue).
        Width(15).
        Height(3).
        Render("hoge"),
)

fmt.Println("## 文字列がはみ出ると自動で折り返されます(しかも英語の場合単語単位でいい感じに)が、高さは変わります")
fmt.Println(lipgloss.NewStyle().Background(blue).
    Width(15).
    Height(3).
    Render("lolem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua."),
)
```

結果はこのようになります。

![text size](/images/introduce_lipgloss/size.png)

### 文字寄せの設定

高さ、幅を指定した場合、その中での文字寄せを設定できます。
`Align`メソッドに`lipgloss.Center`等の定数を指定します。

複数設定すると、左下、中央上などの組み合わせも指定できます。

```go
fmt.Println("# 文字列の描画位置を指定できます")
fmt.Println(
    lipgloss.NewStyle().Background(blue).
        Width(30).
        Align(lipgloss.Center).
        Render("Centered text."),
)
fmt.Println(
    lipgloss.NewStyle().Background(blue).
        Width(30).
        Align(lipgloss.Right).
        Render("Right-aligned text."),
)

fmt.Println("## 上下の位置も指定できます")
fmt.Println(
    lipgloss.NewStyle().Background(blue).
        Width(30).
        Height(3).
        Align(lipgloss.Bottom).
        Render("Bottom-aligned text."),
)
```

結果はこのようになります。

![align text](/images/introduce_lipgloss/align.png)

### 文字囲みの設定

`Border`メソッドを使うと、文字列を囲む枠線を設定できます。

```go
fmt.Println("# 文字を囲う枠線を書くこともできます")

fmt.Println(
    lipgloss.NewStyle().
        Border(lipgloss.NormalBorder()).
        Render("Bordered text."),
)
fmt.Println("## 枠線には色々な種類がありますし、自分で作ることもできます")
fmt.Println(
    lipgloss.NewStyle().
        Border(lipgloss.DoubleBorder()).
        Render("Bordered text."),
)
```

![border](/images/introduce_lipgloss/border.png)

### 高さ、幅を測りたい

`Width`, `Height`関数を使うと、文字列を描画したときの高さ、幅を測れます。
`Size`関数を使うと一緒に取得できます。

裏ではmattnさんの[runewidth](https://github.com/mattn/go-runewidth)を使っているようです。

```go
fmt.Println("# 文字列の幅、高さを取得できます")
str := "Lip Gloss is great.👍\n And cute.🐱"
fmt.Println(str)

width := lipgloss.Width(str)
height := lipgloss.Height(str)
fmt.Printf("width: %d, height: %d\n", width, height)

width, height = lipgloss.Size("You can get the size of the string.")
fmt.Printf("width: %d, height: %d\n", width, height)
```

結果はこのようになります。

![measure size](/images/introduce_lipgloss/measure.png)

## まとめ

Lip Glossの機能をざっくり紹介しました。
他にも色々な機能があり、[Lip GlossのGitHubリポジトリ](https://github.com/charmbracelet/lipgloss)や、[Lip Glossのドキュメント](https://pkg.go.dev/github.com/charmbracelet/lipgloss)に詳しく書かれています。

今後TUIアプリを作る機会があれば、Lip Glossを使ってみてください。

## おまけ

実は、Lip GlossはBubble Teaがよく使っている絵文字を正しく表示できません!
[Bubblesというリポジトリ](https://github.com/charmbracelet/bubbles)で使われている'🫧'という絵文字は[最近Unicodeに追加された](https://lets-emoji.com/bubbles-emoji/)もので、runewidthがまだ対応していないためです。

![bubble](/images/introduce_lipgloss/bubble.png)

以下の結果は`1, 2, 1`になってしまいます。

![bug](/images/introduce_lipgloss/bug.png)

ただし、相当レアケースでほとんどの場合心配しなくても大丈夫です。
(このバグで結構な時間を溶かしたので、共有してみました)

## チートシート全文

この記事で紹介した機能が確認できるコード(チートシート)です。
そのまま実行できますから、手元で色々試してみてください。

<script src="https://gist.github.com/tbistr/c1f72c1fbeb938ee18d4263a44c716d9.js"></script>
