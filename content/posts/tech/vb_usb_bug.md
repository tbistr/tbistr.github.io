---
title: "Virtualbox Manager (GUI)が起動しないバグと解決方法"
description: "virtualboxが起動しないバグに1年くらい苦しめられたが、USBデバイスが原因であることがわかった。"
author: "tbistr"
date: 2022-04-26T07:35:27Z

tags: ["virtualbox"]
# categories: ["themes", "syntax"]
# series: ["Themes Guide"]

# draft: true
---

# 概要
原因はUSBでつながっていたXInputのコントローラ。
デバイスを抜くことで解決。

# 状況
## 症状
- VirtualboxのGUIが起動しない
- `vagrant up`が失敗する
- `VBoxManage.exe`コマンドに応答がない
  - helpは表示できるが、vmのリスト等は出ない

## 環境
- i7-12700
- 32gb
- windows11 pro
- `virtualbox 6.1.32`
  - chocolateyでインストール
- wsl, dockerがインストールされている
- ハイパーバイザプラットフォーム有効化済み

# 確認したこと、やってみたこと
- タスクマネージャの確認
- virtualboxの再インストール
- vagrantの再インストール
- OSのクリーンインストール

## タスクマネージャの確認
「バックグラウンドプロセス」のタブにこの3つのプロセスが存在した。
- `Virtualbox Interface`
- `Virtualbox Global Interface`
- `Virtualbox Manager`

本来は、`Virtualbox Manager`は「アプリ」のタブにあるべき。
起動を試みるたびに、`Virtualbox Manager`は増えていき、ゾンビプロセスになった。

## virtualbox, vagrantの再インストール
chocolatey版、msiインストーラ版両方試したが効果なし。

## OSのクリーンインストール
しばらくは問題が起きなくなったが、ある日突然再発。

# 原因と解決方法
VM起動時にエラーが発生し、いつもどおり原因をググっていたところ、同じような問題がOracleのフォーラムに投稿されていることに気がついた。

https://forums.virtualbox.org/viewtopic.php?t=97261

昨年10月に、このような投稿があった。

>Re: GUI not appearing
>
>Postby LeonJi » 4. Oct 2021, 04:11
>
>I also got this problem this morning.  
>Finally, I found an usb device(wireless adaptor for game controller) was the root cause.  
>After removing this device, the GUI appeared immediately.  
>So I guess , VB detects the usb devices and can not recogize it , some how it comes into an infinite loop before completing the initialization process.

要約すると、
「根本的な原因はゲームのコントローラが接続されていることだった。アプリの初期化時にUSBデバイスを見ており、コントローラを抜いたら解決した。」
と言っている。

**私のPCにもUSBゲームコントローラが有線接続しっぱなしになっていたので、取り除いてvirtualboxを起動してみたところ、正常に起動した。**

やはり原因としてはフォーラムの投稿にあるように、「初期化時にUSBデバイスをチェックするが、ゲームのコントローラに対応できずに無限ループになる。」ということであると考えられる。

タイムアウト処理か、起動後にチェックするようにして欲しいが、バグ報告はOracle会員にならないといけないし、フォーラム経由で既に開発チームに認識されているっぽいので放置。

# おまけ
使っていたコントローラはこれ。

https://www.amazon.co.jp/gp/product/B08NPJ1GTC

普通に質が良く、ジャイロ対応でSwitchでもPCでも安定して使えるのでオススメ。
