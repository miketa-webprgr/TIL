## 第I部 導入編
---
<br>

### ● 第１章 データベース
---

データベース
- RDBは、データの種類ごとにテーブルに格納し、必要に応じてテーブルを連結したり加工する
- SQLは、RDBで使う専門の言語

リレーショナルデータベース
- データ(テーブル)とロジック(SQL)が分離している
- 細かくテーブルが分かれているので、データの組み合わせを柔軟に変えられる
- データの整合性を保てる

リレーショナルデータベースのテーブル
- テーブルは行と列から構成される
- 行と列の交わった箇所はフィールド。
- テーブルは行の集合
  - 列の集合ではない
  - テーブルの行には順番がない
- 行を識別するのは主キーの値
- 列を識別するのは名前(列名)
  - 列を増やすのは大変
  - テーブルやSQLの変更が必要

<br>

### ● 第２章 PostgreSQLのインストール
---

この章では、SQLの勉強用にPostgreSQL をインストールする方法を説明。  
なお、本番環境で使うようなインストール方法ではないとのこと。  

とりあえず、Postgres.appをインストールすればよいということなので、  
カチカチと突き進んでいく。  

Homebrewを使ったインストールについてもHPで記述があったが、  
ここは本に素直に従うことにする。  


<a href="https://gyazo.com/c816f489290da6c12cf0b96079846150"><img src="https://i.gyazo.com/c816f489290da6c12cf0b96079846150.png" alt="Image from Gyazo" width="550"/></a>

<br>

ポートが空いていないらしい。。。  
きっとRailsでPostgreSQLを使っているからだろう。  

<a href="https://gyazo.com/5a07803805315afc337451d382cbf18b"><img src="https://i.gyazo.com/5a07803805315afc337451d382cbf18b.png" alt="Image from Gyazo" width="550"/></a>

<br>

Server Settingでポート番号を１つずらしてみることに。  
Railsをやる際にトラブルにならないといいけど。  

<br>

### ● 第３章 基本操作
---

コマンドラインの挙動がおかしく、だいぶテンパる。  
１時間半近くして、やっと法則性が分かった。  

```
user=# select * from testtable1 order by name
```

「;」を入れ忘れたためか反応がない。
もう一度入力。

```
user-# select * from testtable1 order by name;
ERROR:  syntax error at or near "select"
LINE 2: select * from testtable1 order by name;
 ```

このuser「-」って何！？  
間違ってないはずなのにエラーになっている！！！

・・・！  

これがあると、２行に分けて打っているということか！  

```
user=# select *
user-# from 
user-# testtable1
user-# ;
 id | name | age 
----+------+-----
(0 rows)
```

無駄にトリッキーなことをして挙動を確かめてみた。  

```
user=# create table testtable1 (
user(# id integer primary key
user(# , name text not null
user(# , age integer
user(# );
```

このように「(」で終われば、「user(」に変わると。  
なるほど。もはやSQLの勉強ではないが。  

ここからはサクサクと進む。  
第３章も無事終了。以下、ノート。  

---

（例) select * testtable1 set age=22 where name = Alice;
- まず特定のテーブルから必要なデータを選ぶ
- 処理を入力する
- 条件を指定する
- 最後のセミコロンを忘れない

新しい行の挿入
```
insert into testtable1(id, name, age)
values(101, 'Alpha', 20)
    , (102, 'Blavo', 25);
```

更新する(年齢を+1)
```
update testtable1
set age = age + 1;
```

削除する
```
# 条件を指定しないと全てのデータが削除されるので注意
delete from testtable1;
```

テーブルを削除
```
drop table testtable1;
```

<br>

### ● その他
---

VSCode関係に脱線し、マルチカーソルを知る。  
これは便利だ！  

[Qiita：VSCodeのマルチカーソル練習帳](https://qiita.com/TomK/items/3b1f5be07d708d7bd6c5)