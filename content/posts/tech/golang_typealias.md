---
title: "Type aliasの使い所を提案"
description: "Go言語の謎機能である、Type aliasの使い所を説明"
author: "tbistr"
date: 2023-03-06T21:04:34Z

tags: ["golang"]
# categories: ["themes", "syntax"]
# series: ["Themes Guide"]
---

## Type alias

Type aliasはGo 1.9で追加された機能です。
以下のようなsyntaxで既存の型に別名を定義できます。

```golang
type aliasT = string
```

普通のDefined typeと何が違うのかというと、定義された型と元になる型との同一性が異なります。
具体的にDefined typeでは以下のように独自メソッドを生やすことができる一方、Type aliasにはメソッドを追加できません。

```golang
type defT string

func (d defT) SomeMethod() {
    // これは出来る
}

type aliasT = string

func (d aliasT) SomeMethod() {
    // これは出来ない!
}
```

これはaliasT型がstringと完全に同じとみなされているからで、以下のようなコードはコンパイルが通ります。

```golang
type aliasT = string

func f() string {
    var a aliasT
    return a
}
```

## 使い所

導入経緯的には、大規模コードのAPI変更をサポートするために作られた機能だそうで、普通のコードベースだと一生体験する機会がなさそうです。
そんな感じで微妙にどこに使えば良いか迷うこの機能ですが、最近書いたコードで良さそうなユースケースを見つけました。

### 導入対象

対象となるプロジェクトは自作の[AtCoder関連のライブラリ「atcoder-go」](https://github.com/tbistr/pig)で、注目したい部分は以下のようなファイル群です。

```plaintext
atcodergo
    |---model // 外部に公開する構造体を定義
    |   |---model.go
    |---parse // HTMLパーサー、セレクタを使ってHTML→構造体の変換
    |   |---task.go
    |   |---(etc.go...)
    |---task.go // 外部から操作するメソッドを定義
    |---(etc.go...)
```

各ファイルの内容はざっくり、こんな感じです。

```golang
// model/model.go
type Task struct {
 Name   string
 IdName string
 ID     string
}

// parse/task.go
func Task(doc pig.Node) []*model.Task {
    tasks := []*model.Task{}
    // pig(自作ライブラリ)を使ってHTMLをパース
    return tasks
}

// /task.go
func Task(doc pig.Node) []*model.Task {
    tasks := []*model.Task{}
    // 通信してページを取得
    tasks = parse.Task(doc)
    return tasks
}
```

以下のようなことを考慮してこの構成になりました。

- parseだけするパッケージを分けたい→`parse/`追加
  - テストを書く場合parse関数は分離するべき
  - `parse~()`みたいなprefixを付けた関数が量産されるのは嫌
- `parse/`の中の関数で返す型と、`atcodergo/`で返す型は共通化したい→`model/`追加

### 導入

`atcodergo/task.go`をこのように変更しました。

```golang
// /task.go

type Task = model.Task

func Task(doc pig.Node) []*Task {
    tasks := []*Task{}
    // 通信してページを取得
    tasks = parse.Task(doc)
    return tasks
}
```

## まとめ

この書き方には以下のような利点があります。

- 外部から見える型が`model.Task`のように一段深くならない
- Defined typeと異なり型変換が不要

型変換については、もしDefine typeを使った場合には`unsafe.Pointer`や`unsafe.Slice`を使用してキャスト処理を書く必要があります。
この処理自体はそんなに長くもないのですが、やはり元の型との整合性を自分で担保しないといけないのはちょっと怖いです。

一方この書き方には、型のドキュメント(godoc)が空になり、基になる型の情報がわかりにくくなるというデメリットがあります。

こんな感じで、外部に公開したい型を参照しやすくするという用途では結構使えるんじゃないかと思っています。

## おまけ(Alias typeの導入経緯)

Type aliasの導入経緯は[このページ](https://go.dev/talks/2016/refactor.article)から読むことが出来ます。
ざっくり言うと大規模コードベースでパッケージを整理する際、一気にすべてのコードを変更しなくて済むように`type newT = oldT`のように書くことを想定しています。
