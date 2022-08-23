---
title: "DockerComposeで環境変数のエイリアスを作りたい"
description: ""
author: "tbistr"
date: 2022-08-23T12:10:26Z

tags: ["docker"]
# categories: ["themes", "syntax"]
# series: ["Themes Guide"]

# draft: true
---

## 経緯

相変わらずDocker上のMySQLをいじっている。

MySQLサーバに対して、毎回コンテナに入ってmysqlプロンプトを出してコマンドを打つのがめんどくさいので、webベースのGUIサーバを追加することにした。
このとき、`.env`で指定しているMySQL用のユーザ情報について、同じ値を別の名前で環境変数として指定する必要があった。

## 環境

docker-composeファイルは以下の通り。

```yaml
version: '3.8'

services:
  db:
    container_name: db
    image: mysql:8.0
    environment:
      - LANG=ja_JP.UTF-8
    tty: true
    volumes:
      - type: volume
        source: test_volume
        target: /var/lib/mysql
    networks:
      - test_network

volumes:
  test_volume:
    name: test_volume

networks:
  test_network:
    external: true
```

`.env`はこの内容。

```
MYSQL_DATABASE=test_database
MYSQL_USER=test_user
MYSQL_PASSWORD=password
MYSQL_ROOT_PASSWORD=root_password
```

## 課題

今の環境に、phpmyadminというMySQLのGUIツールを新しいサービスとして導入する。
具体的に以下の内容を追加したい。

```yaml
  db-gui:
    container_name: db-gui
    image: phpmyadmin/phpmyadmin
    environment:
      - LANG=ja_JP.UTF-8
    tty: true
    ports:
      - 8080:80
    depends_on:
      - db
    networks:
      - test_network
```

このとき、phpmyadminコンテナには環境変数`PMA_HOSTS`, `PMA_USER`, `PMA_PASSWORD`を渡して接続先と認証の情報を教える必要がある。
しかし、`PMA_HOSTS`はサービス名(`db`)を直書きすれば良いとして(networkが設定されてるので、サービス名←→アドレスができる)、他の2つの値は`.env`の`MYSQL_USER`, `MYSQL_PASSWORD`と同一である。

愚直な方法として、`.env`に`PMA_USER`, `PMA_PASSWORD`を追加しても良い。
が、同じ値(しかも認証情報!)を2回書くのはメンテナンス性も悪いし、ミスを誘発しやすい。

## 解決方法

今回、環境変数にエイリアスを設定することでこの問題を解決した。

このように、`db-gui`サービスのパラメータに`environment:`を設定して、値を`${envで設定された変数名}`とすることで、`db-gui`で変数が読まれる際には`MYSQL_USER`の値で置き換えられる。

```yaml
    environment:
      - LANG=ja_JP.UTF-8
      - PMA_ARBITRARY=1
      - PMA_HOSTS=db
      - PMA_USER=${MYSQL_USER}
      - PMA_PASSWORD=${MYSQL_PASSWORD}
```

## おまけ

環境変数の評価タイミング的に、`environment`設定時に展開されるのか、参照時に展開されているのかが気になったので確認してみた。
つまり、`PMA_USER="${MYSQL_USER}"`なのか、`PMA_USER="test_user"`かということである。

試すのは簡単で、

```bash
docker compose exec db-gui bash
> echo $PMA_USER
test_user
> MYSQL_USER=hoge
> echo $PMA_USER
test_user
```

上書き後の最後の結果が`test_user`なので、`environment`に設定した変数はコンテナ作成時に評価されることが分かる。
更に複数の手段で指定される環境変数が、どの順番で読み込まれるかも分かる。
例えば、`.env`に`TEST=$PMA_HOSTS`などと`environment`で指定された変数を利用しようとしても、デフォルト値の`""`として評価される。

一応確認してみるとこのようになる。

```bash
docker compose exec db-gui bash
> echo $TEST

> echo $PMA_HOSTS
db
```

逆に応用テクとしてこんな風に`.env`の内容を利用して複雑な変数を用意することもできたりする。
```
COMPLEX=${MYSQL_PASSWORD}concat_ENVS${MYSQL_PASSWORD}hogehuga
```
