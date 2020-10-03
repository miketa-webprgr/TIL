# MySQLノート

## MySQLのバージョン

５系と８系は違う。  
ネットで検索する場合も、その違いを意識すること。  
記事のとおりやっても動かない場合、バージョンが違う可能性がある。  

## よく使うコマンド

```text
mysql.server start
mysql.server stop
```

## 魔法のコマンド

今度、魔法のコマンドについて、改めてちゃんと理解しよう。  
初期設定でパスを設定する。  

```text
bundle config --local build.mysql2 "--with-cppflags=-I/usr/local/opt/openssl/include"
bundle config --local build.mysql2 "--with-ldflags=-L/usr/local/opt/openssl/lib"
```

- [【Rails】MySQL2がbundle installできない時の対応方法 \- Qiita](https://qiita.com/fukuda_fu/items/463a39406ce713396403)

## ユーザー確認

ルートユーザーでアクセスし、ユーザーの一覧を確認する。

```text
mysql -u root -p
#=> パスワードを入力

select user, host, password from mysql.user;
```

## MySQLの初期設定

自分で書いたQiita。

- [【MacOS \- Homebrew版】MySQLの設定方法（インストール→MySQLサーバー立ち上げ→パスワード設定等） \- Qiita](https://qiita.com/miketa_webprgr/items/ba7210ac57e2086fc5b6#3-mysql%E3%81%AE%E3%83%91%E3%82%B9%E3%83%AF%E3%83%BC%E3%83%89%E8%A8%AD%E5%AE%9A%E7%AD%89%E3%82%92%E8%A1%8C%E3%81%86)

ユーザー作成の際に参考になった。

- [ユーザーに権限を設定する\(GRANT文\) \| MySQLの使い方](https://www.dbonline.jp/mysql/user/index6.html)

## その他の基本的なコマンド

```text
show databases;
show database データベースA;
show tables;
show table テーブルA;
```