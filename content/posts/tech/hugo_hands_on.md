---
title: "hugoお試ししてみた。"
description: "hugo + github pagesで静的サイト公開してみる。"
author: "tbistr"
date: 2021-11-23T09:09:26+09:00

tags: ["hugo", "github"]
# categories: ["themes", "syntax"]
# series: ["Themes Guide"]

draft: false
---

## 今回使ったツールなど

### hugo
hugoは、go言語で書かれた静的サイトジェネレータ。
markdownで記事を書くことができ、有志が開発したテーマによって、簡単にキレイなサイトを作ることができる。

https://gohugo.io/about/what-is-hugo/

### github pages
githubのリポジトリに置いたファイル群を、専用のURL(独自ドメインもおk)に紐づけてホスティングしてくれるサービス。

https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages

### github actions
githubに対するpush, pull requestなどのアクションをトリガにして、ビルドや他サービスへのデプロイ、テストなどのアクションを実行してくれるサービス。(CI/CDとかいうやつですかね。)

https://docs.github.com/ja/actions

## やってみる

### hugo
https://gohugo.io/getting-started/quick-start/

linuxbrewに対応しているので、`brew install hugo`でインストールした。
windowsだとchocolatey、macだとhomebrewでインストールするのが楽だと思う。

公式のクイックスタートガイドに沿って、ローカルで使い方を確認する。

### テーマ選び
hugoでは複数テーマを好きなときに切り替えられるようになっているが、設定項目も全然違うので、気軽に切り替えるようなもんでもない。
そのため、ローカルで多少吟味してから決めると良さそう。

今回は、以下の項目を評価して決めた。
- 必須の設定項目が多すぎない。
- リポジトリやホームページに情報がちゃんと載ってる。
- exampleサイトの実装がある。(情報がカスでも、最悪写せる。)
- 最新の更新が新しめ。

上3つを考慮してanankeを考えていたが、更新が途絶えていたためpaper-modを選択した。

### (おまけ)テーマの適用方法について
ネット上の記事ではgit submoduleを使ってテーマを管理する方法が主流になっているが、最新版ではhugo modulesという機能を使うことができる。

詳細は以下のURLを参考にして欲しいが、モジュール初期化後にconfig.ymlにインポートしたいパッケージ名を書けば良い。
(載ってないが、テーマの指定部分にパッケージ名を書くだけでいける。)

中身はgo modulesを使っているらしい。


### github
普通にリポジトリを作成してpagesを設定すると、URLは`htps://[user_name].github.io/[repo_name]/`になる。
自分のホームページ的な運用をしたいのであれば、リポジトリ名を`user_name.github.io`にすることで、`https://[user_name].github.io/`でホスティングしてくれる。

とりあえず好きな方でリポジトリを作成し、hugoのディレクトリとして`hugo new site`してプッシュする。

### github actions
今回は、mainブランチにpushしたときにビルドを走らせて、ホスティング対象となるファイルをgh-pagesブランチにcommitしてくれるactionsを採用した。
先人が必要な記述を用意してくれているので、ほぼそのままパクる。

https://github.com/peaceiris/actions-hugo#getting-started を参考に、`.github/worlflows/gh-pages.yml`を作成して、中身はそのままコピペする。
適当にコミットしてみて、リポジトリのactionsタブから、正常にビルドができていることを確認する。(コケてたら、`hugo-version`をローカルと合わせてみると多分うまくいく。)

### github pages
作成したリポジトリのsettingsタブから、pages項目を開き、`Source`のブランチを`gh-pages`、ディレクトリを`/root`に設定する。
(先述のactionsによるビルドが成功していれば、ブランチは自動で作成されているはず。)

### hugoの設定をイジる
必須な項目として、`config.yml`の`baseURL`を、pagesが対象にするURLに合わせる必要がある。
このサイトなら`https://tbistr.github.io/`、pagesタブで`Your site is published at https://ほげ`とされているURL。

あとはローカルの`hugo server`で確認しつつ好きなように設定する。
