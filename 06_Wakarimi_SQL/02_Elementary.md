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