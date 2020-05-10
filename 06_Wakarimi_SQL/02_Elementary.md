## 第II部 初級編
---
<br>

### ● 第４章 select文
---

何らかの句を出力するには、select句を使う。  
出力方法は、以下のとおり。  

```
select 'Hello';
```

テーブルを指定してその中身を出力するには、from句を使う。  
テーブルの全ての行を出力する場合は以下のとおり。  
（テーブル名は「members」とする）  

```
select *
from members;
```

なお、列名を選択して出力することも可能。  

```
# genderとnameは列名
select gender, name
from members;
```

ある特定の行を選ぶ場合には、where句で指定する。
等号だけでなく、不等号も使える。  

```
select *
from members
where name = 'ミカサ';
```

where句では、andやorなどを使うこともできる。

```
# 該当部分のみ抜粋
where gender = 'M' and height >= 170;
```

整列するには、「order by」句を使う。  
order by 句をつけないと順番は不定のため、なるべく使うこと。  

```
select *
from members
order by name;
```

原則として昇順にならぶが、降順にする場合はdescを加える。
また複数条件を指定する場合、以下のように指定する。

並び替える場合、重複しない値を使うこと。  

```
# 背が高い順に並べ、同じ場合はidが小さい順に並べる
order by height desc, id;
```

先頭の行からn行だけ出力するには、limit句を使う。  
先頭の行からn行だけ出力しない場合には、offset句を使う。  
limitとoffsetを組み合わせる場合、まずoffsetが適用される。  

```
# 先頭から３行スキップされ、４行目だけを出力する
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

集約関数（Aggregate Function)  

- 合計する (sum())
- 平均する (avg())
- 最大値を調べる (max()) 
- 最小値を調べる (min()) 
- 行数を数える (count())

小数点以下の桁数を指定
- to_char(avg(height), ’999.99’)
  - これで小数点２桁までの表記になる
  - ’999.99’が書式指定

```
select gender, avg(height)
from members
group by gender;
```

```
select gender, max(height), min(height), sum(height)
             , count(*), to_char(avg(height), '999.99')
from members
group by gender
order by gender desc;
```

countについて
count(*) = 行数をカウントする
count(height) = 身長がnullでないものをカウントする

集約関数の戻り値は、データ型が集約関数ごとに異なる。
- avgは、整数を受け取っても、戻り値のデータ型は実数になる。
  - ３と５を受け取っても、戻り値は４ではなく4.0となる。

having句を使うと、グループをフィルターできる。

```
 gender | max | min | sum | count | to_char 
--------+-----+-----+-----+-------+---------
 M      | 175 | 163 | 676 |     4 |  169.00
 F      | 170 | 158 | 328 |     2 |  164.00
(2 rows)
```

having avg(height) > 168　を加えると、以下のとおり。  
なお、order by の後に加えると syntax errorになる。  

```
 gender | max | min | sum | count | to_char 
--------+-----+-----+-----+-------+---------
 M      | 175 | 163 | 676 |     4 |  169.00
(1 row)
```

ポイント:
- グループ化される前は「行」が操作対象
  - なので where 句は行をフィルタする
- グループ化後は「グループ」が操作対象
  - なので having 句や order by 句や select 句はグループを操作する
  - また「select gender」の gender は、 行の列名ではなくグループのキー名を表す

また、group byを使うと、select句で指定できるのは「group化のキー」と「集合関数」だけ。  
要は、select句とgroup by句を合わせないと、エラーが起きるということ。  

例えば、以下はエラーが起きる。  

```
# select genderとしてgroup by と合わせないとエラーになる

select name, avg(height)
from members
group by gender;
```

order byでも同様にselect句との組み合わせが悪いとエラーとなる。  

```
# nameでは並びかえできない。
# genderか、avg(height)でしか並べ替えられない。  

select gender, avg(height)
from members
group by gender
order by name;
```

group by では式が指定できる。

```
select length(name), count(*)
from members
group by length(name); 
```

