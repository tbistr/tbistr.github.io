---
title: "MySQLのチュートリアルでコケた"
description: ""
author: "tbistr"
date: 2022-08-19T13:48:10Z

tags: ["mysql"]
# categories: ["themes", "syntax"]
# series: ["Themes Guide"]

# draft: true
---

## 流れ

今作ってるbotでMySQLを使いたくなったので、使ってみる。
作ってるのは<https://github.com/tbistr/gs-linker>。

## 参考

MySQLのチュートリアルサイト? <https://www.mysqltutorial.org/getting-started-with-mysql/> を参考にした。

環境はVagrantで作った。
Boxは`bento/ubuntu-20.04`。

## インストール

チュートリアルサイトでの[インストールの手順](https://www.mysqltutorial.org/install-mysql-ubuntu/)では、
`sudo mysql_secure_installation`でどんなパスワードを設定しても

```plaintext
 ... Failed! Error: SET PASSWORD has no significance for user 'root'@'localhost' as the authentication method used doesn't store authentication data in the MySQL server. Please consider using ALTER USER instead if you want to change authentication parameters.
```

のエラーで失敗する。

そのため、`sudo mysql_secure_installation`の前に、

```bash
sudo mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '適当なパスワード';
mysql> exit
```

を実行した。

あとは手順通り、すべて`y`でOK。

一応確認、

```bash
sudo mysql -u root -p

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.02 sec)
```

できとるがね。(河村市長)
