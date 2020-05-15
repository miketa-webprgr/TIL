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

#### 方針転換
---

進捗具合が芳しくなかったため、ここから勉強の進め方・ノートの取り方で方針転換を図る。  

従来
1. 読みながらマークダウンでノートを取る
2. 事例は逐一ノートを取りつつ、コマンドラインで試す 

今後
1. とりあえず概要が理解できるまで読む
2. 逆にいえば、細かいところは理解しない
3. そのあとメモをマークダウンで記載
4. 重要だと思ったところだけコマンドラインで試す
5. あとは、実戦で試す機会があるはずなので、割り切る！

<br>

#### サブクエリの中にサブクエリ
---

サブクエリの中にサブクエリを使える。  

<br>

#### exists 演算子
---

in 演算子であれば、以下に対してtrue/falseを判断し、結果を返す。  

```sql
select 7 in (3,7,9);
```

exits演算子は、条件が合う行があるか調べて、１行でもあればtrueを返す。  
なければfalseを返す。  

select exits (サブクエリ)　といった形で使う。  


<br>

#### all 演算子
---

サブクエリが返した値がすベて条件に合致していればtrueを返す。  
一つでも合致しなければfalseを返す。  

```sql
--この場合、サブクエリの結果が全て40以下であればtrueを返す

select 40 <= all(サブクエリ)
```

<br>

#### any 演算子
---

any 演算子は、all 演算子と異なり、一つでも合致していればtrue。
何も合致しなければfalseを返す。

<br>

#### with 句 (CTE)
---

with句は、サブクエリに別名（Alias）をつけるのに使う。  
以下のような形で使う。  

なお、列名も別名を命名できる。  

```sql
with 別名（列名にも命名できる） as (サブクエリ)
本体SQL
```

<br>

#### with 句を使ったリファクタリング
---

リファクタリングとは、機能は同じままにコードを分かりやすく書き換えたりすること。  
リファクタリングでは、with句を活用するとよい。  

<br>

#### サブクエリを一時テーブルとみなす
---

同じサブクエリを２回使う場合、with句を活用すると省略できる。  

<br>

#### with 句の注意点
---

with句を使うと、SQLの動作が遅くなることがある。  
場合によっては、with句を使わずにまとめることも検討すべき。  

<br>

#### with 句の練習
---

ここは重要であると思われるので、SQLで実践してみる。  
以下を解読＋リファクタリングしてみる。  

```sql
-- 書き換え前
-- 教科＋教科ごとの平均点のテーブルを作成し、そこから平均点が最も低いデータの行を抽出

select subject, avg_score from (
    select subject, avg(score) as avg_score
    from test_scores
    group by subject
) avg_scores
-- avg_scores = 教科＋教科ごとの平均点のテーブル
where avg_score = (
select min(x.avg_score) from (
select subject, avg(score) as avg_score from test_scores
group by subject
)x );
-- x = avg_scores
```

```sql
-- 書き換え後

with avg_scores(subject, avg_score) as (
    select subject, avg(score) as avg_score
    from test_scores
    group by subject
) 
-- avg_scores = 教科＋教科ごとの平均点のテーブル

select subject, avg_score
from avg_scores
where avg_score = (
    select min(avg_score) from avg_scores
);
```

<br>

### ● 第８章 テーブル結合（基礎編）
---

<br>

#### from句とwhere句でテーブル結合
---

以下のようなコードにより、  

- moviesテーブルに格納されている４行のデータ
- charactersテーブルに格納されている７行のデータ

以上が掛け合わされて、２８行のテーブルが出力される。  

```sql
select movies.*, characters.*
from movies, characters;
```

ここで、主キーであるmovies_idでwhere句を使って一致させることにより、  
正しい映画とキャラクターの正しい組み合わせのテーブルが出力される。  

```sql
select movies.*, characters.*
from movies, characters;
where movies.movies_id = characters.movies_id;
```

<br>

#### join演算子
---

where句は、条件を指定するもの。  
ただ、組み合わせてテーブルを結合させるための条件と、そこからの抽出条件が混ざるとややこしい。  

そこでjoin演算子。  
joinを使い、足したいテーブルを記載。  
その後にonを使って、テーブル結合の条件を記載。  

以下が事例。  

```sql
select *
from movies m
  join characters c 
    on m.movie_id = c.movie_id
where c.gender = 'F';
```

<br>

#### テーブル結合によってできること
---

テーブル結合を使うと、あるテーブルを検索するときに別のテーブルを利用できる。

<br>

#### サブクエリとの比較
---

サブクエリを使っても、あるテーブルを検索するときに別のテーブルを利用できる。  
テーブル結合からサブクエリの書き換え、その逆の書き換えに慣れるとよい。  

<br>

#### using
---

joinで結合する際には、onではなくusingが使える。  
usingを使うと短く書けるが、制限も出てくる。  

以下がonからusingへの書き換えの事例。  

```sql
-- 書き換え前
select *
from movies m
  join characters c 
    on m.movie_id = c.movie_id

-- 書き換え後
select *
from movies m
  join characters c 
    using (movie_id)
```

usingは、movie_idと〜idと~idが一致するときのような、  
複数の列名を使って結合するときには、省略がかなりできるため、その効果を発揮する。  

なお、natural joinというより省略した書き換えがあるらしい。  

<br>

### ●第９章 内部結合と外部結合
---

以下のjoinの性質と使い方を学んだ。  
- inner join(省略されてjoinとも言われる)
- left join
- right join
- full outer join(省略されてfull joinとも言われる)
- cross join

<br>

#### テーブル結合の注意点
---

joinを使うと、当然であるが、共通のカラムで紐付けを行ってテーブルを作成するので、  
共通のカラムの値がnullであったり、値があっても他方のテーブルにその値が存在しないと、  
作成されるテーブルにそのデータは表れてこない。  

```sql
id | movie_id |  name   | gender
---+----------+---------+--------
401 |      93 | ナウシカ  | F
402 |      94 | パズー    | M 
403 |      94 | シータ    | F 
404 |         | クラリス  | F  ← こういったデータはjoinしても表れない
405 |      96 | メイ     | F  ← もう一方のテーブルに96番がないと紐付けられない
```

<br>

#### leftjoin と rightjoin
---

inner joinしても表れないデータを表示させるため、  
left join と right join を使うことができる。  

left joinを使うと、join元のテーブルで表れないデータも表示させることができる。  
right joinを使うと、join先のテーブルで表れないデータも表示させることができる。  

<br>

#### 完全外部結合
---

full outer join(省略されてfull join)を指す。  
full outer join を使うと、join先とjoin元のテーブルで表れないデータの両方を表示させることができる。  

<br>

#### クロス結合
---

cross joinを指す。  
join元とjoin先のデータの全ての組み合わせで構成されるテーブルを出力する。  

<br>

#### テーブル結合によるフィルタ
---

様々なjoinを活用して、条件にあった行だけを選んだり、除いたりすることができる。  

inner joinの活用。  
映画テーブルとキャラクターテーブルがある場合、キャラクターが登録されていないテーブルや  
映画が登録されていないキャラクターのみを抽出することができる。  

left joinの活用。  
映画テーブルとキャラクターテーブルがある場合、映画が登録されていないキャラクターのみを  
取り除くことができる。その際、where ~ is null を組み合わせる。  

<br>

#### 練習問題
---

難易度が少し高かったものはつまづいたので、丁寧に確認。  

<br>

##### 問題:映画ごとの出演キャラクター数(難易度:★★☆)
映画ごとに出演者の数を集計し、映画 ID 順に表示してください。  
もし出演者がいない場合は 0人として集計してください。  

##### 考え方

① left joinにより、映画IDにて紐付けて、映画テーブルとキャラクターテーブルを  
合体させてものをテーブルとして出力（出演キャラクターがいないものも含む）

```sql
select * from movies m 
  left join characters c 
    on m.movie_id= c.movie_id;

 movie_id |      title       | id  | movie_id |   name   | gender 
----------+------------------+-----+----------+----------+--------
       93 | 風の谷のナウシカ | 401 |       93 | ナウシカ | F
       94 | 天空の城ラピュタ | 402 |       94 | パズー   | M
       94 | 天空の城ラピュタ | 403 |       94 | シータ   | F
       94 | 天空の城ラピュタ | 404 |       94 | ムスカ   | M
       95 | となりのトトロ   | 405 |       95 | さつき   | F
       95 | となりのトトロ   | 406 |       95 | メイ     | F
       96 | 崖の上のポニョ   |     |          |          | 

```

② 出力項目として、映画ごとの出演者の数がある。count()と group by句を活用する。  
m.movie_idでグループ化し、c.idもしくはc.nameをカウントする。  

```sql
-- 結果のみ表示

 movie_id |      title       | count 
----------+------------------+-------
       93 | 風の谷のナウシカ |     1
       94 | 天空の城ラピュタ |     3
       95 | となりのトトロ   |     2
       96 | 崖の上のポニョ   |     0
```
