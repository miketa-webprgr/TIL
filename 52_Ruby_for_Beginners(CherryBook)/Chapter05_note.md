# Chapter05 ハッシュやシンボルを理解する

## ハッシュ

こういうやつ。  
取り出しが高速でできるらしい。  

この矢印はハッシュロケットという。  
使うと古くさいらしい。  

h = { 'japan' => 'yen', 'us' => 'dollar', 'india'=> 'rupee'}

なお、ブロックの`{}`とは違う。 

### 要素の追加、変更、取得

ハッシュのキーを追加する。  

```rb
h['italy'] = 'euro'
h =  { 'japan' => 'yen', 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro'}
```

ハッシュのキーを変更する。  

```rb
h['japan'] = 'YEN!!!'
h =  { 'japan' => 'YEN!!!', 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro'}
```

ハッシュを取り出す。  

```rb
h =  { 'japan' => 'YEN!!!', 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro'}
h['us'] #=> "dollar"
```

存在しないキーを選ぶと`nil`が返る。  

```rb
h =  { 'japan' => 'YEN!!!', 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro'}
h['france'] #=> nil
```

### ハッシュを使った繰り返し処理

`each`を使うと、キーと値の組み合わせを順に取り出すことができる。  
キーと値は可能した順に取り出せる。  

```rb
h =  { 'japan' => 'YEN!!!', 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro'}

h.each do |key, value|
  puts "#{key}: #{value}"
end

#=>
japan: YEN!!!
us: dollar
india: rupee
italy: euro
```

ブロック引数を1つにすると、キーと値が配列に格納される。  

```rb
h =  { 'japan' => 'YEN!!!', 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro'}

h.each do |h|
  # h は ['japan', 'YEN!!!'], ['us', 'dollar'], ['inidia', 'rupee'], ['italy', 'euro']となっている
  key = h[0]
  value = h[1]
  puts "#{key}は#{value}という通貨を使用している"
end

#=>
# japanはYEN!!!という通貨を使用している
# usはdollarという通貨を使用している
# indiaはrupeeという通貨を使用している
# italyはeuroという通貨を使用している
```

### ハッシュの同値比較、要素数の取得、要素の削除

ハッシュ同士を`==`を比較すると、同じハッシュかどうかチェックできる。  
順番がことなろうとも、キーと値が同じであればtrueが返ってくる。  

```rb
h = { 'japan' => 'YEN!!!', 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro' }
i = { 'italy' => 'euro', 'japan' => 'YEN!!!', 'us' => 'dollar', 'india'=> 'rupee' }
h == i #=> true
```

ハッシュの要素の数を調べることや、要素の削除をすることもできる。  

```rb
h = { 'japan' => 'YEN!!!', 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro' }
h.size #=> 4
h.delete('japan') #=> "yen
h #=> "{ 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro' }
```

## シンボル

こういうやつ。  
コロンを最初につける。  

```rb
:apple
```

`'apple'`は、Stringクラスである。  
`:apple`は、Symbolクラスである。  

Stringクラスだと、同じ文字であっても違うオブジェクトとして管理される。
Symbolクラスだと、同じ文字である場合同じオブジェクトとして管理される。  

```rb
'apple'.object_id #=> 70216948842760
'apple'.object_id #=> 70216948462140
'apple'.object_id #=> 70216948584180

:apple.object_id #=> 2030108
:apple.object_id #=> 2030108
:apple.object_id #=> 2030108
```

また、シンボルの場合、中身が整数であるため高速で処理できるらしい。  

あと、シンボルはイミュータブル（変更不可能）なので、破壊的メソッドは使うことができない。  
Stringクラスの値で上書きしてしまえば、破壊的メソッドを使うことができる。  

```rb
fruit = :apple
fruit.upcase! #=> NoMethodError (undefined method `upcase!' for :apple:Symbol)

# 'orange'
fruit = 'orange'
fruit.upcase! #=> ORANGE
```

シンボルの場合、これはこう書き換えられる。  
なお、正確にはStringクラスのオブジェクトをSymbolクラスに書き換えているので、厳密には微妙に異なる。  

```rb
# h =  { 'japan' => 'YEN!!!', 'us' => 'dollar', 'india'=> 'rupee', 'italy' => 'euro'}
h =  { :japan => 'YEN!!!', :us => 'dollar', :india => 'rupee', :italy => 'euro'}

# h =  { :japan => 'YEN!!!', :us => 'dollar', :india => 'rupee', :italy => 'euro'}
h = { japan: 'YEN!!!', us: 'dollar', india: 'rupee', italy: 'euro'}

# h = { japan: 'YEN!!!', us: 'dollar', india: 'rupee', italy: 'euro'}
h = { japan: :YEN, us: :dollar, india: :rupee, italy: :euro}


h.each do |h|
  # h は ['japan', 'YEN!!!'], ['us', 'dollar'], ['inidia', 'rupee'], ['italy', 'euro']となっている
  key = h[0]
  value = h[1]
  puts "#{key}は#{value}という通貨を使用している"
end

#=>
# japanはYEN!!!という通貨を使用している
# usはdollarという通貨を使用している
# indiaはrupeeという通貨を使用している
# italyはeuroという通貨を使用している
```

## 続・ハッシュについて

### キーや値に異なるデータ型を混在させる

- キーについても混在させることができるが、基本的にはデータ型は揃える方がよい。  
- 一方では、ハッシュに格納する値については、データ型が混在することが多い。  

```rb
person = {
  # 値が文字列
  name: 'Alice',
  # 値が数値
  age: 20,
  # 値が配列
  friends: ['Bob', 'Carol'],
  # 値がハッシュ
  phones: { home: '123-4567', mobile: '090-1234-5678'}
}

person[:friends] #=> ["Bob", "Carol"]
person[:phones] #=> {:home=>"123-4567", :mobile=>"090-1234-5678"}
person[:phones][:home] #=> "123-4567"

person[:phones].each do |type, phone|
  puts "#{type}: #{phone}"
end

#=>
# home: 123-4567
# mobile: 090-1234-5678

person[:phones].each do |phone|
  puts phone[1]
end
#=>
# 123-4567
# 090-1234-5678
```

### メソッドのキーワード引数とハッシュ

メソッドに引数を渡す場合、どの引数がどんな意味を持つのか分かりづらい場合がある。  
例えば、以下のような場合、メソッドのキーワード引数を使うと可読性が上がる。  

```rb
# 変更前
def buy_burger(menu, drink, potato)
  # ハンバーガーを購入
  if drink
    # ドリンクを購入
  end
  if potato
    # ポテトを購入
  end
end

buy_burger('cheese', true, true)
```

```rb
# 変更後
def buy_burger(menu, drink: true, potato: true)
  # ハンバーガーを購入
  if drink
    # ドリンクを購入
  end
  if potato
    # ポテトを購入
  end
end

buy_burger('cheese', drink: true, potato: true)
```

なお、キーワード引数にはデフォルト値があるので、引数を省略することができる。  

```rb
# 変更後
def buy_burger(menu, drink: true, potato: true)
  # 省略
end

# drinkについては、デフォルト値の true のままでよいので引数を省略する
buy_burger('cheese', potato: false)
```

また、デフォルト値なしのキーワード引数を指定することができる。  

```rb
def buy_burger(menu, drink:, potato:)
  # 省略
end
```

キーワード引数を使うメソッドを呼び出す場合、キーワード引数に一致するハッシュ  
（キーはシンボル）を引数として渡すこともできる。  

```rb
params = { drink: true, potato: false }
buy_burger ('fish', params)
# buy_burger('fish', drink: true, potato: false)とイコール
```

## ハッシュについて詳しく

### ハッシュでよく使うメソッド

- keysメソッド
  - ハッシュのキーを返す
- valuesメソッド
  - ハッシュの値を配列として返す
- has_key?/key?/include?/member? メソッド
  - has_key?メソッドは、指定されたハッシュの中にキーがあるか`t/f`を返す
  - その他のメソッドは、エイリアスメソッド

### **でハッシュを展開させる

`**`をハッシュの前に付けると、ハッシュリテラルないで他のハッシュのキーと値を展開できる。  

```rb
h = { us: 'dollar', india: 'rupee' }
# ハッシュの中に、`h`というハッシュを入れ子にできる
# mergeメソッドを使って、同じことができる
{ japan: 'yen', **h } #=> {:japan => "yen", :us => "dollar", :india => "rupee"}
{ japan: 'yen'}.merge(h) #=> {:japan => "yen", :us => "dollar", :india => "rupee"}
```

### ハッシュを使った擬似キーワード

これを見たら、キーワード引数と同じようなものだと理解すること。  
キーワード引数はRuby2.0から導入されたため、古いコードだと見ることがあるらしい。  

- `def buy_burger(menu, options ={})`

### 任意のキーワードを受ける`**`引数

これはエラーになる。

```rb
def buy_burger(menu, drink: true, potato: true)
  # 省略
end

# saladというキーワード引数がないので、Argumentエラーになる
buy_burger('fish', drink: true, potato: false, salad: true)
```

けど、こうしてやれば、キーワード引数がないものも受け取れる

```rb
# othersは記入しなくてもよい
def buy_burger(menu, drink: true, potato: true, **others)
  # 省略
end

# エラーにならない
buy_burger('fish', drink: true, potato: false, salad: true)
```

### メソッド呼び出し時の`{ }`の省略

最後の引数がハッシュであれば、ハッシュリテラルの`{ }`を省略できる。  
これから書き換え前と書き換え後の事例を示す。

なお、最後の引数ではない場合、ハッシュリテラルの`{ }`は省略できない。  

```rb
def buy_burger(menu, options = {})
  puts options
end

buy_burger('fish', {drink: true, potato: false})
#=> {:drink=>true, :potato=>false}
```

```rb
def buy_burger(menu, options = {})
  puts options
end

buy_burger('fish', drink: true, potato: false)
#=> {:drink=>true, :potato=>false}
```

### ハッシュリテラルの`{ }`とブロックの`{ }`
