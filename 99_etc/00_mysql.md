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

## MySQLのバージョン起因？の問題

エラーメッセージは以下のとおり。  

```text
SELECT list is not in GROUP BY clause and contains nonaggregated column … incompatible with sql_mode=only_full_group_by
```

解決策。  
詳細はまだ理解していない。  

```text
SET GLOBAL sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
```

- [MySQL5\.7でONLY\_FULL\_GROUP\_BYをOFFにする方法 \- Qiita](https://qiita.com/shiutk/items/9176e6d76c1707bc36c7)
- [StackOverFlow](https://stackoverflow.com/questions/41887460/select-list-is-not-in-group-by-clause-and-contains-nonaggregated-column-inc)

## MySQLの初期設定

自分で書いたQiita。

- [【MacOS \- Homebrew版】MySQLの設定方法（インストール→MySQLサーバー立ち上げ→パスワード設定等） \- Qiita](https://qiita.com/miketa_webprgr/items/ba7210ac57e2086fc5b6#3-mysql%E3%81%AE%E3%83%91%E3%82%B9%E3%83%AF%E3%83%BC%E3%83%89%E8%A8%AD%E5%AE%9A%E7%AD%89%E3%82%92%E8%A1%8C%E3%81%86)

ユーザー作成の際に参考になった。

- [ユーザーに権限を設定する\(GRANT文\) \| MySQLの使い方](https://www.dbonline.jp/mysql/user/index6.html)

## その他の基本的なコマンド

- [別ノートを参照すること](../11_Rails_Intensive_Training/00_mysql_note.md)
