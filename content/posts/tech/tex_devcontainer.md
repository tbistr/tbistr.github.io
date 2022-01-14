---
title: ""
description: ""
author: "tbistr"
date: 2022-01-14T05:20:52Z

tags: ["tex", "devcontainer", "docker"]
# categories: ["themes", "syntax"]
# series: ["Themes Guide"]
---

### 概要
卒論のために久しぶりにTeX環境を作る必要があったので、TeX Liveがfullでインストールされたコンテナイメージを探していたのですが、無かったので作りました。
通常のインストール手順ではハマるポイントがあったのでパッケージマネージャを使わない方法でインストールしました。

devcontainer化することで、vscodeから立ち上げるだけで補完、プレビューが使えるようになっています。

### devcontainer
devcontainerとは、vscodeの（拡張）機能の1つで、これを利用することで用意したコンテナの中にワークスペースフォルダをマウントし、コンテナ内のコマンド、環境を使うことができます。
また、拡張機能、設定についても指定したものを使うことができます（ローカルのvscodeにはインストールされません）。
これにより、環境の分離、チーム内での共通化ができます。

参考：[devcontainerのチュートリアル](https://code.visualstudio.com/docs/remote/containers-tutorial)

### ネット上に存在するTeX用Dockerイメージ
ネット上には、フルインストール版、日本語に必要なモジュールに限定した版の両方について、様々なイメージが存在しています。
~~しかし、日本語環境がうまく入らない、texindex（フォーマッタ）が動作しないなど、日本語環境で常用するには不具合が発生しました。~~
<font color="tomato">執筆中に試してみたら、公式が配布している[`texlive/texlive`イメージ](https://hub.docker.com/r/texlive/texlive)はちゃんと全部動きました。。。~~まあ、見なかったことにします。~~</font>
また、fullと謳っていながら実際にはモジュールに不足があるイメージもあります（主に日本語関係のモジュールがない）。

## 制作内容
全容は[Githubのリポジトリ](https://github.com/tbistr/texlive-full-devcontainer)から参照できます。

### tex.profile
texを非対話的に（ビルド時にユーザーがオプション等を選択しない）インストールするのに必要な設定ファイルです。
今回は、このファイルによるスキーマ指定がしたかったため、aptは使わずに、ソースからインストールしています。

`selected_scheme scheme-full`でfullバージョンを指定しています。

その他で肝要な部分は、`option_doc 0`と`option_src 0`です。
fullでインストールすると、ドキュメントとソースを含むだけで2GB近く消費してしまいます。
そのため、このオプションで除外しておきます。

```bash
selected_scheme scheme-full

TEXDIR /opt/texlive/2021
TEXMFCONFIG ~/.texlive2021/texmf-config
TEXMFHOME ~/texmf
TEXMFLOCAL /opt/texlive/texmf-local
TEXMFSYSCONFIG /opt/texlive/2021/texmf-config
TEXMFSYSVAR /opt/texlive/2021/texmf-var
TEXMFVAR ~/.texlive2021/texmf-var

binary_x86_64-darwin 0
binary_x86_64-linux 1
binary_win32 0
option_doc 0
option_src 0
option_adjustrepo 0
```

余談ですが、この設定ファイルの書式、TeX Liveのリファレンスにも見つけられませんでした（一度デフォルト設定でビルドすると出力されるから、それを改変しろ。ということらしい。）

### Dockerfile
本体となるDocekrfileです。
ベースにはmicrosoftが公開しているdevcontainer用に設定されたイメージを使用しました。
基本的なutil系コマンドがインストール済みです。

それ以降の処理はコメントに記載してあるとおりです。

```Dockerfile
FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-21.04

COPY tex.profile ./

# 日本のミラーサーバからTeX Live 2021をダウンロードしてくる
RUN wget https://texlive.texjp.org/2021/tlnet/install-tl-unx.tar.gz \
    && mkdir install-tl \
    && tar xzf install-tl-unx.tar.gz -C install-tl --strip-components 1 \ # 解凍
    && ./install-tl/install-tl \ # インストールスクリプトを起動
    --profile ./tex.profile \ # 先述のプロファイルを指定する
    --repository http://texlive.texjp.org/2021/tlnet \ # 依存のDL元も日本のミラーサーバ
    && rm -r install-tl-unx.tar.gz install-tl

# latexindentを使うために必要なperlのモジュールをaptで取得する。
# cpan（perlのパッケージマネージャ）でのインストールはperl外の依存（makeとか）を必要とするため、対応するものをaptで探した。
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    libyaml-tiny-perl \
    libfile-homedir-perl \
    libunicode-linebreak-perl \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

USER vscode

# 環境変数の更新
RUN echo 'export PATH="$PATH:/opt/texlive/2021/bin/x86_64-linux"' >> ~/.bashrc
```

また、本番リポジトリではこのDockerfileをビルド、Dockerhubにアップしたものをpullしてくる様になっています。
そのため、通常のtexliveのインストール、ビルドよりも大幅に短期間でセットアップできます。
（一度pullしておけば再起動は爆速。）

### その他
latexmkrc（texからPDFへのビルド手順を定義したファイル）と、settings.json（vscodeの設定）については、[このリポジトリ](https://github.com/bana118/latex-container-template)を参考にしています。

## まとめ
以上の内容をテンプレートリポジトリとして[こちら](https://github.com/tbistr/texlive-full-devcontainer)にアップしました。
補完、フォーマットも効いて、エラーは該当ラインに表示、ファイルの保存時に自動的にビルドするなど、手間の割になかなか便利に使えると思います。
是非使ってみてください。
