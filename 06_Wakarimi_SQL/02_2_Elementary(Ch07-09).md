## 第II部 初級編（partII)  
第７章 サブクエリ文, 第８章 テーブル結合（基礎）, 第９章 内部結合と外部結合
---
<br>

### ● 第７章 サブクエリ文
---

サブクエリとは、SQLの中にSQLを埋め込む機能である。  

例えば、このような形でサブクエリは使われる。  

```sql
select *
from members
-- ここをサブクエリで置き換える
where height > 167.3
-- サブクエリで置き換えてみた後
where height > ( select avg(height) from members ) 
```

サブクエリ(つまり埋め込む SQL)がどんな結果を返すのかによって、サブクエリの使い方が決まる。  

・・・といってもイメージしづらいが、SQLの実行結果は４パターンに分けられて、
それぞれ場面に合った使い方をしていく必要がある。  

- 単一列単一行　→　単一値の代わり
- 複数列単一行　→　複合値の代わり
- 単一列複数行　→　in 演算子とともに使う　
- 複数列複数行　→　テーブルの代わり

<br>

#### 単一列単一行のサブクエリ
---

単一値の代わりに使う。  
要は、「10」だとか「'paper'」だとかの単一のものの代わり。
単一であれば、数値でも文字列でも日付でも真偽値（t/f）でもよい。

例えば、以下は国語の最高得点を出力するSQLである。  

```sql
select max(score)
from test_scores
where subject = '国語'
```

単一値を出力するはずなので、下記のような場面においてサブクエリとすることができる。  
ちなみに、以上のSQLでは80点という点数が出力されるとする。  

```sql
select t.student_id, t.score from test_scores t
where t.subject = '国語'
-- ここで、先ほどのSQLをサブクエリとして使える
and t.score = 80
order by t.student_id;
```

<br>

#### 複数列単一行のサブクエリ
---

複合値の代わりに使う。  

複合値のイメージは、Rubyでいうところのハッシュである。  
(subject, score) = (’ 国語’, 80)がその具体例である。  

例えば、以下はこのような結果を出力するSQLである。  
実行結果：(subject, score) = (’ 国語’, 80)

```sql
select subject, max(score) from test_scores
where subject = '国語' 
group by subject;
```

これをサブクエリとして使う。  
```sql
select t.student_id, t.score
from test_scores t
-- ここで、先ほどのSQLをサブクエリとして使える
where (t.subject, t.score) = ('国語', 80)
order by t.student_id;
```

なお、実際には複数列単一行のSQLは,１行だけのテーブルの代わりに使うことがほとんど。  
詳細は後ほど確認していくとのこと。  

<br>

#### 単一列複数行のサブクエリ
---

in 演算子の右側に指定するサブクエリとして使う。  
・・・？  

まず、in 演算子の復習から。  

> 複数の値を列挙して、その中に探しているものが含まれているか探す。  
> 含まれていないか確認する場合は、not inを使う。  

> ```sql
> -- 7があればtrue 
> select 7 in (3,7,9);
> ```

なるほど。  
( , , )　←　これに該当する部分をサブクエリで置き換えられるということかな。  

具体的に事例を見ていく。  
以下は、何らかの教科で100点満点をとった生徒のIDを全て表示するSQLである。  

```sql
# 単一列（student_id）に対して、複数行（student_id）の結果が出力

select student_id
from test_scores
where score = 100;
```

これをサブクエリとして使う。  
```sql
select s.*
from students s
-- ここで、先ほどのSQLをサブクエリとして使える
where s.id in (203, 204, 204)
order by s.id;
```

なお、単一列に対して、複数行の結果が出力されるような形式であればよく、  
実際の結果が1行や0行であってもエラーとならない。  

例えば、先ほどの事例において何らかの教科で5000万点とった生徒のIDを出力するSQLを  
サブクエリとして使ったとしてもエラーは起きない。  

<br>

#### 複数列複数行のサブクエリ
---

テーブルがわりのサブクエリとして使える。
導出テーブルとも言われる。

以下は各教科とその平均点のテーブルを出力するSQLである。

```sql
select subject, avg(score) as avg_score
from test_scores
group by subject;
```

これをサブクエリとして使う。  
具体的にはfrom句の後に使える。  

サブクエリに別名（Alias）が必要なので注意すること。  
先ほどのテーブルから、平均70点を下回るデータのみを出力する。  

```sql
select x.subject, x.avg_score
from(
    select subject, avg(score) as avg_score
    from test_scores
    group by subject
) x
where x.avg_score < 70
```

<br>

#### サブクエリの中にサブクエリ
---

