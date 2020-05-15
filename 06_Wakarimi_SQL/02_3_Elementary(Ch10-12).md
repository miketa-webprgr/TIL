## 第II部 初級編（partIII)  
第10章 制約、主キー、外部キー, 第11章 テーブルの関連, 第12章 インデックス
---
<br>

### ● 第10章 制約、主キー、外部キー
---

<br>

#### NOTNULL制約
---

列に null が入るのを禁止する制約を指す。  
NOTNULL制約がついているか調べるには、psqlコマンドで「\dt テーブル名」  

```sql
create table movies (
  movie_id  integer   primary key
  -- 「not null」がNOTNULL制約
, title     text      not null unique
);
```

<br>

#### 一意制約
---

列の値に重複があってはならない制約を指す。  
一意制約がついているか調べるには、psqlで「\d テーブル名」  

```sql
create table movies (
  movie_id  integer   primary key
  -- 「unique」が一意制約
, title     text      not null unique
);
```

<br>

#### 主キーと主キー制約
---

主キー (Primary key)とは、テーブル内の行を特定するための列。  
主キーがどれかを調べるには、psql コマンドで「\d テーブル名」  

主キーには特徴があり、  
- 値が一意でなければならない（特定できなくなるため）  
- 値がnullであってはならない（NULL制約）  
- 主キーの値は変わってはならない  

```sql
create table movies (
  -- 「primary key」が主キーを指定している
  movie_id   integer   primary key
, title      text      not null unique );
```

<br>

#### 外部キーと外部キー制約
---

外部キー (Foregin key) とは、他テーブルの主キーを参照している列。  
外部キーを調べるには、psqlコマンドで「\d テーブル名」  

外部キー制約 (Foreign key constraint)とは、参照先のテーブルにない値が外部キーに入るのを禁止する制約。  
他から参照されている主キーを変更しようとすると、エラーになる。  

```sql
create table characters (
  id         integer   primary key
  -- 「references movies(movie_id)」が外部キーを指定している
, movie_id   integer   references movies(movie_id)
, name       text      not null
, gender     char(1)   not null
);

```

<br>

#### 複合主キーと複合外部キー
---

複数の列から構成された主キーは複合主キー (Composite primary key) という。 
テーブルに対して主キーがどれかを指定する。

参照されている複数の外部キーは、複合外部キーという。  

```sql
create table writings (
  book_id    integer   not null references books(id)
, author_id  integer   not null references authors(id)
-- 「primary key(book_id, author_id)」が主キーを複数指定している

, primary key(book_id, author_id)
);
```

<br>

#### 練習問題
---

##### NOT NULL 制約 

- NULLにできないカラム（列）
- 必ず値が入っていないといけない場合に使う

##### 一意制約

- 他の行のデータと値が重複してはいけない
- 他のデータと必ず区別させたい時に使う

##### 主キー

- 行を特定するための列
- 条件があり、① not null ② 一意 ③ 値が変わらないこと

##### 外部キー

- 外部キーとは、他のテーブルの主キーを参照するための列
- 外部キーには参照先にある値しか入れることができない（紐付けできない）
- 外部キーが参照している値は変更できない