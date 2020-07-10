# MySQL操作ノート

## ここを見るとよい

- [【MySQL, SQL】データベースを扱う基本SQL一覧 \- Qiita](https://qiita.com/knife0125/items/bb095a85d1a5d3c8f706)
- [【SQL実践】- TechEssentials](https://tech-essentials.work/courses/12)

## よく使うコマンド

### MySQLサーバーに接続・停止

mysql.server start
mysql.server stop

### データベースに接続

mysql -u root -p

-u ----- 「ユーザを指定する」というオプション
-u root ----- 「rootユーザで」みたいな意味
-p ----- 「パスワードを指定してログインする」というオプション

### 作成済のデータベース一覧を確認する

show databases;

### 使用するデータベースの選択

USE [データベース名];

### テーブル一覧の表示

SHOW TABLES;

### `users`テーブルから全てのデータを取得（users以外でも可能）

SELECT * FROM `users`;
