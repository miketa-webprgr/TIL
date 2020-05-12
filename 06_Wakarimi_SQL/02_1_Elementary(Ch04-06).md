## 第II部 初級編（partI)  
第４章 select文, 第５章 group by 句, 第６章 便利な機能  
---
<br>

### ● 第４章 select文
---

何らかの句を出力するには、select句を使う。  
出力方法は、以下のとおり。  

```sql
select 'Hello';
```

テーブルを指定してその中身を出力するには、from句を使う。  
テーブルの全ての行を出力する場合は以下のとおり。  
（テーブル名は「members」とする）  

```sql
select *
from members;
```

なお、列名を選択して出力することも可能。  

```sql
-- genderとnameは列名
select gender, name
from members;
```

ある特定の行を選ぶ場合には、where句で指定する。
等号だけでなく、不等号も使える。  

```sql
select *
from members
where name = 'ミカサ';
```

where句では、andやorなどを使うこともできる。

```sql
-- 該当部分のみ抜粋
where gender = 'M' and height >= 170;
```

整列するには、「order by」句を使う。  
order by 句をつけないと順番は不定のため、なるべく使うこと。  

```sql
select *
from members
order by name;
```

原則として昇順にならぶが、降順にする場合はdescを加える。
また複数条件を指定する場合、以下のように指定する。

並び替える場合、重複しない値を使うこと。  

```sql
-- 背が高い順に並べ、同じ場合はidが小さい順に並べる
order by height desc, id;
```

先頭の行からn行だけ出力するには、limit句を使う。  
先頭の行からn行だけ出力しない場合には、offset句を使う。  
limitとoffsetを組み合わせる場合、まずoffsetが適用される。  

```sql
-- 先頭から３行スキップされ、４行目だけを出力する
select * from members order by id
limit 1 offset 3;
```

なお、where句と「order by」句を使う場合、whereを先に使うこと。  
順番を間違えるとエラーになる。  

また、SQLは基本的に上から順番に実行されるが、select句は最後に実行される。  
複雑なコードであっても、このことに留意して読み解く。  

練習問題も無事終了。  

<br>

### ● 第５章 group by 句

---

#### 集約関数（Aggregate Function)  
---  

- 合計する (sum())
- 平均する (avg())
- 最大値を調べる (max()) 
- 最小値を調べる (min()) 
- 行数を数える (count())

小数点以下の桁数を指定する場合、以下のように書く。  

- to_char(avg(height), ’999.99’)
  - これで小数点２桁までの表記になる
  - ’999.99’が書式指定

##### 書き方の事例  
---

```sql
# selectする際に、集約関数を使える
# そもそも列が選択できるわけではなく、式を選択するもの
# genderというのも、=genderという簡単な式を使っているだけ

select gender, avg(height)
from members
group by gender;
```

```sql
select gender, max(height), min(height), sum(height)
             , count(*), to_char(avg(height), '999.99')
from members
group by gender
order by gender desc;
```

##### その他  
---

countについて
count(*) = 行数をカウントする
count(height) = 身長がnullでないものをカウントする

集約関数の戻り値は、データ型が集約関数ごとに異なる。
- avgは、整数を受け取っても、戻り値のデータ型は実数になる。
  - ３と５を受け取っても、戻り値は４ではなく4.0となる。

#### having句  
---

having句を使うと、グループをフィルターできる。

```sql
-- フィルター前

 gender | max | min | sum | count | to_char 
--------+-----+-----+-----+-------+---------
 M      | 175 | 163 | 676 |     4 |  169.00
 F      | 170 | 158 | 328 |     2 |  164.00
(2 rows)
```

having avg(height) > 168　を加えると、以下のとおり。  

```sql
-- フィルター後

 gender | max | min | sum | count | to_char 
--------+-----+-----+-----+-------+---------
 M      | 175 | 163 | 676 |     4 |  169.00
(1 row)
```

なお、order by の後に加えると syntax errorになる。  
順番に注意すること。  

#### SQLの処理について注意すること  
---

ポイント:
- グループ化される前は「行」が操作対象
  - なので where 句は行をフィルタする
- グループ化後は「グループ」が操作対象
  - なので having 句や order by 句や select 句はグループを操作する
  - また「select gender」の gender は、 行の列名ではなくグループのキー名を表す

また、group byを使うと、select句で指定できるのは「group化のキー」と「集合関数」だけ。  
要は、select句とgroup by句を合わせないと、エラーが起きるということ。  

例えば、以下はエラーが起きる。  

```sql
-- select genderとしてgroup by と合わせないとエラーになる

select name, avg(height)
from members
group by gender;
```

order byでも同様にselect句との組み合わせが悪いとエラーとなる。  

```sql
-- nameでは並びかえできない。
-- genderか、avg(height)でしか並べ替えられない。  

select gender, avg(height)
from members
group by gender
order by name;
```

group by では式が指定できる。

```sql
select length(name), count(*)
from members
group by length(name); 
```

<br>

### ● 第６章 便利な機能
---

<br>

#### コメントと別名（Alias）  
---

SQL には、２種類のコメントがあります。  
- 範囲コメント ... 「/*」から「*/」までを読み飛ばす。
- 行コメント ... 「--」から行末までを読み飛ばす。

SQL では、列や式に別名 (Alias) をつけられる。  
以下のとおり。

```sql
-- 早速、コメントの練習！
-- asは省略可能

select name as 名前, height as 身長, gender as 性別 
from members;

   名前   | 身長 | 性別 
---------+------+------
 エレン   |  170 | M
 ミカサ   |  170 | F
 アルミン |  163 | M
 ジャン   |  175 | M
 サシャ   |  168 | M
 コニー   |  158 | F
(6 rows)

```


なお、参照用の別名（SQL操作用のラベル）とヘッダ用の別名（表示用）を変えるとよい。  
変えるには、「導出テーブル」を使う必要がある。  
（導出テーブルは今後解説があるとのこと）  

SQL では、テーブルにも別名 (Alias) をつけられる。  
よくテーブルに短い別名を付けて、その別名を列名の接頭辞として使うことがある。  

1. SQLのキーワード（例：selectなど）と同じ列名を使う場合、  
   キーワードを使おうとしているのではないことを分かりやすくするため。  
2. 複数のテーブルを扱う場合、どのテーブルなのか明示するため。 

```sql
select m.id, m.name, m.height
from members as m 
where m.gender = 'F'
order by m.id;
```

なお、where句では別名は参照できず、order by句では参照できる。  

<br>

#### 演算子  
---

- 「+」「-」「*」「/」の四則演算
- 「=」といった等号や、「>」「>=」「<」 「<=」の比較など
  - 「!=」は「<>」（等しくないを意味する）
- 「(」「)」のようなカッコ類
- 「in」「and」「join」「is nulll」などの単語

「and」「or」の注意すべき点として、「and」の優先順位が高いことが挙げられる。
- 「x=1 or y=1 and z=1」は、「x=1」を満たしているもしくは「y=1とz=1」を満たしているか。

<br>

##### 文字列結合演算子：「||」
---

<br>

##### パターンマッチ演算子：「like」
---
- 文字列があるパターンにマッチしているか調べて、true/falseを返す
- 「%」や「_」を組み合わせることが多い
- not likeという逆パターンもある
- 大文字小文字を区別しない場合、ilikeを使う
- whereなどと組み合わせることが多い

```sql
-- サで終わればtrue
select 'ミカサ’ like '%サ';
```

```sql
-- サで終わる３文字であればtrue
select 'ミカサ' like '__サ';
```

```sql
-- membersテーブルから「ン」で終わるデータを取得
select name
from members
where name 'like %ン' ;
```
<br>

##### in演算子
---

複数の値を列挙して、その中に探しているものが含まれているか探す。  
含まれていないか確認する場合は、not inを使う。  

```sql
-- 7があればtrue 
select 7 in (3,7,9);
```

<br>

##### null
---

値がないこと。空であること。  

なお、nullであることを条件に検索をかけても、そのデータは取得できない。  
<b>nullを使った計算式はどれも結果がnullになる。</b>  

is null と is not null を使うと、true/false が確認できる。  

nullである場合に別の値に変換したい時、coalesce( )関数を使う。
複数の引数を指定することができる。nullでない最初の引数が返される。  

```sql
-- xがnullなら0を返す
coalesce(x, 0)

-- xがnullでなければxを返す
-- xがnullならyを返す
-- yもnullならzを返す
coalesce(x, y ,z)
```

```sql
-- movie_idがnullのものを未登録に変換して出力

select id, coalesce(movie_id||'','未登録') as movie_id, name, gender
from characters order by id;

 id  | movie_id |   name   | gender 
-----+----------+----------+--------
 401 | 93       | ナウシカ | F
 402 | 94       | パズー   | M
 403 | 94       | シータ   | F
 404 | 94       | ムスカ   | M
 405 | 95       | さつき   | F
 406 | 95       | メイ     | F
 407 | 未登録   | クラリス | F
(7 rows)
```

なお、null を使った論理演算式は、基本的に結果はnullだが、  
「false and null」はfalse、「true or null」はtrueとなります。  

<br>

##### 複合値（Composite value）
---

数値や文字列や真偽値を集めて組にしたもの。  
```sql
(10,20,30)
(10, 'A', true)
```

複合値は比較できる。
データ型が異なっても比較できる。

```sql
-- まず、１番目の値で比較して判断
-- １番目と２番目の値が同じである場合、２番目の値で比較
-- 以下の例の場合、trueになる

select(1,2,3)<(1,4,3);
select
('2019-03-11'::date, 9) > ('2019-03-10'::date, 9);
```
