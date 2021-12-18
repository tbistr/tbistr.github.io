---
title: "Vagrantでデスクトップ環境はいらないけど、GUIアプリを使いたい"
description: "x11 clientをVagrantで立てたubuntu VMに設定し、windows上のx11 serverで描画させます。"
author: "tbistr"
date: 2021-12-18T17:01:15+09:00

tags: ["vagrant", "linux"]
# categories: ["themes", "syntax"]
# series: ["Themes Guide"]
---
# 何がしたいか
Vagrantで立てたVMについて、デスクトップ環境(GNOMEとかLXDEとか)を導入して、GUI環境を実現させる方法は様々な紹介されている。  
しかし、デスクトップ環境の導入をすると、プロビジョニングのステップで`apt-get install ubuntu-desktop`などする必要があり、めちゃめちゃ時間がかかる。  
ここでは、VM内のアプリケーションをwindows上のウィンドウとして描画させることを目指す。

# 仕組み
## 用語
- X Window System (X11)
    - linuxがデファクトで使っている、ウィンドウ表示プロトコル。
- X11クライアント
    - 画面を描画してもらうアプリケーション。
    今回の場合、ゲストOS上で動くアプリケーション。
- X11サーバー
    - 画面を描画する側。
    今回の場合、ホストOSのウィンドウシステム。
- vcXsrv
    - 今回使うwindows向けX11サーバー。
    X11サーバーは色々種類があるが、最近はこれが主流らしい。

## 実際の動き
下図のように、SSH経由で送信。
![動作イメージ](/images/vagrant_x11.drawio.png)

# 実践
## 準備
Chocolateyを使っているので、`choco install vcxsrv`でvcXsrvをインストール。  
(https://sourceforge.net/projects/vcxsrv/ からもインストールできる)

起動すると設定項目が出る。
すべて初期設定で問題ない。

タスクバーにX11のロゴが出ていたら準備完了。(簡単！！！)

## Vagrantファイルの用意
最終的に、以下のようなVagrantファイルを用意する。

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.10"

  config.ssh.forward_x11 = true

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade
  SHELL
  
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    echo "export DISPLAY=10.0.2.2:0" >> ~/.bash_profile
  SHELL
end
```

そこまで解説することはないが、少しだけ。

`config.ssh.forward_x11 = true`は、X11を有効化するオプション。
これをtrueに設定することで、`vagrant ssh`で接続したときに自動でx11を転送してくれる。

`echo "export DISPLAY=10.0.2.2:0" >> ~/.bash_profile`では、X11でどこに描画するかを設定する。
virtual boxでは、ゲストからホストは10.0.2.2でアクセスすることができるので、このように設定する。

`privileged: false`はプロビジョニングを非ルートユーザー(デフォはvagrantユーザー)で実行する設定。
これで~/がユーザーのホームになる。
