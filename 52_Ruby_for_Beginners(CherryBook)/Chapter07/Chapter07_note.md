## チェリー本 Chapter07 「クラスの作成を理解する」

---

<BR><BR>

### 経緯について  
---

「Rubyプログラムをオブジェクト指向的なアプローチでリファクタする」解説を聞いた。  
せっかくの機会なので、これを機会にチェリー本の該当箇所を読み、Ruby自体の理解を深めたい。  

また、Ruby自体、そもそもProgateで２周したのはよいが、綺麗に忘れているので、  
欲を言えば、７章に限らず、どこかで勉強する機会を設けたい。  

<BR>

### ハッシュだと管理しきれない
---

クラスを使うプログラミングと、使わない場合の違いについて説明があった。  

- 苗字・名前・年齢といった３つのカテゴリを持つデータを扱いたいとする。  
- Rubyの場合、ハッシュを使ってユーザーのデータを都度作っていくことができる。

```rb
users = []
users << {first_name: 'Alice', last_name: 'Ruby', age: 20 }
users << {first_name: 'Bob', last_name: 'Python', age: 30 }
```

これでもよいが、勘違いやバグが発生しやすくなってしまう。

#### バグの事例 その１
---

以下の例の場合、afeと打ってしまったためにnilとなっているのだが、その打ち間違いに気がつかないと、  
アリスさんが年齢不詳であるためにnilになったと勘違いしてしまうかもしれない。  

```rb
users[0][:afe] #=> nil
```

#### バグの事例 その２
---


以下の例の場合、設計上不要であるにもかかわらず、性別に関するカテゴリーが作られてしまう。  

```rb
users[0][:sex] = 'F'
```

#### バグの事例 その３
---

以下の例の場合、BobさんをRobertさんに書き換えたかったのだが、うっかりミスのため、  
AliceさんがRobertさんに変更されてしまう。（0とするところを1としてしまった）

```rb
users[1][:first_name] = 'Robert'
```

<BR>

### クラスを使うとミスしづらい
---

以下のようにクラスを定義すると、その型に合致してくれるか判断されるため、  
ガッチリと堅牢なプログラムを作る上では有効である。  

このようにクラスを作ると、先ほどのようなミスは防止できる。  

```rb
class User
  attr_reader :first_name, :last_name, :age

  def initialize(first_name, last_name, age)
    @first_name = first_name
    @last_name = last_name
    @age = age
  end
end
```

<BR>

### 用語について
---

- クラス
- オブジェクト
- インスタンス
- レシーバ
- メソッド
- メッセージ
- ステート
- 属性（attributes, properties)

すごく雑にまとめます。  

<dl>
  <dt>クラス</dt>
  <dd>データの型。そのクラス内のオブジェクトの属性を定めたり、そのクラスが使えるメソッドを定める。</dd>
  <dt>オブジェクト、インスタンス、レシーバ</dt>
  <dd>クラスに属するデータ。</dd>
  <dd>レシーバは、メソッドの受け皿というニュアンスを強調する場合に使われる。</dd>
  <dt>メソッド、メッセージ</dt>
  <dd>そのクラスに属するデータが使える処理。</dd>
  <dd>メッセージは、レシーバに対して送るというニュアンスを強調する場合に使われる。</dd>
  <dt>ステート</dt>
  <dd>オブジェクトごとに保持されるデータ。先ほどのUserクラスでいうと「Alice」とか「20」とか。</dd>
  <dd>一応、Aliceの属性であるfirst_nameもステートには入るらしい。</dd>
  <dt>属性</dt>
  <dd>オブジェクトから取得できる値。Userクラスでいうと「first_name」とか「age」とか。</dd>
</dl>

<BR>

### initializeメソッド
---

initializeメソッドを使うと、例えば User.newとした時に、  
そのメソッドが自動的に実行される。  

また、initialize(name, age)のように引数をつけると、  
単純にUser.newではエラーとなる。  

その場合、必ずUser.new(Alice, 20)のように書くこと。  

<BR>

### インスタンス変数・ローカル変数
---

<dl>
  <dt>インスタンス変数</dt>
  <dd>@で始まるやつ。「@user = 」とか。</dd>
  <dd>クラスを超えて参照することができる。</dd>
  <dd>自作アプリを作るとき、とりあえず@を付けがち。</dd>
  <dt>ローカル変数</dt>
  <dd>@で始まらないやつ。「user = 」とか。</dd>
  <dd>クラスを超えて参照できない。</dd>
</dl>

<BR>

### クラスを超えてインスタンス変数を参照する
---

以下のように書くと、外部から @name を参照できる。

```rb
class User
  # User.newすると、nameを引数として求める。
  # nameはインスタンス変数として扱われる。
  def initialize(name)
    @name = name
  end

  # @nameを外部から参照するためのメソッド
  def name
    @name
  end
```

これで、nameが外部から参照できる。  

```rb
user = User.new('Alice')
user.name #=> Alice
```

また、以下のように書くと、外部から @nameを書き換えることができる。

```rb
class User
  def initialize(name)
    @name = name
  end

  # @nameを外部から書き換えるためのメソッド
  def name=(value)
    @name = value
  end
```

これで、nameが外部から書き換えられる。  

```rb
user = User.new('Bob')
# nameメソッドを使って、BobをRobertに変更
user.name = 'Robert'
# user.nameを出力すると、Robertになっている！
user.name #=> Robert
```

<BR>

### アクセサメソッド
---

先ほどの事例で分かったとおり、外部からインスタンス変数を参照したり、  
書き換えを行ったりするのは、わざわざメソッドを書くのが面倒くさい。  

そこで、そんな面倒なあなたのために、attr_accessorがある。  

これを使うと、以下の両方のメソッドを足してくれる。  
- 外部から参照するためのメソッド
- 外部から書き換えるためのメソッド

参照するだけでよい場合は、attr_readerを使う。  
書き換えだけしたい場合は、attr_writerを使う。  

attr_accessorは属性を複数指定することができる。  

複数の属性を外部から参照・書き換えしたい場合、特に威力を発揮する。  
以下のとおり、こんなに短くコードが書ける。  

【Before】
```rb
class User
  def initialize(name, age)
    @name = name
    @age = age
  end

  def name
    @name
  end

  def name=(value)
    @name = value
  end

  def age
    @age
  end

  def age=(value)
    @age = value
  end
end
```

【After】
```rb
class User
  attr_accessor :name, :age

  def initialize(name, age)
    @name = name
    @age = age
  end
end
```

<BR>

### クラスメソッド
---

インスタンスメソッドを使う場合、User.new しなければならない。  

そのオブジェクトを使って、その属性の値を活用したい場合はそうせざるを得ないのだが、  
使わなくてもよい場合、クラスメソッドが使える。

例えば、そのクラスのインスタンスの値を使って、「〜さんは〜歳です」と出力したい場合、
以下のshow_ageメソッドのようなインスタンスメソッドを使わざるを得ない。

```rb
class User
  attr_accessor :name, :age

  def initialize(name, age)
    @name = name
    @age = age
  end

  def show_age
    "#{@name}さんは、#{@age}歳です"
  end
end
```

ただ、Userオブジェクトに属するAliceさんやBobさんとか、もしくは新たに追加されるかもしれない  
Charlieさんとかの属性を持ってきて処理する必要がない場合、クラスメソッドが使える。  

そもそも、Userクラスに含めるかという問題もあるが、例えば「こんにちは」と出力するメソッドを  
追加したい場合、インスタンスメソッドだと無駄なコードを書く必要があり、面倒くさい。  

```rb
class User
  def hello
    "こんにちは"
  end
end

user = User.new
user.hello #=> こんにちは
```

この場合、素直に puts "こんにちは" してろよという話なのかもしれないが、  
理由によりメソッド化したい場合、クラスメソッドにすると若干面倒が省ける。  

```rb
class User
  # クラスメソッドのときは、self.を追加する
  def self.hello
    "こんにちは"
  end
end

user.hello #=> こんにちは
```

若干だけど 笑  

あと、意味がない userインスタンスの作成が省けるというのは大きいのかもしれない。  

なお、先ほどのような書き方とは変わって、このようにクラスメソッドを書くことができる。  

```rb
# self.と書かなくてよいので、たくさんクラスメソッドを作る時にはよいかも。  

class User
  class << self
    def クラスメソッド
    end
  end
end
```

<BR>

### 定数
---

また、クラスの中には定数を定義することもできる。  
定数を定義せずとも、その定数部分に値を直接入れてしまえばよいので、絶対必要なものではない。  

ただ、以下のような観点から定数は有効である。
- 定数の名前から、プログラムの意味・意図が通りやすくなる 
- 定数名で参照できる（直接書いた場合、同じ値をまた書く必要があり、ミスにつながる）
- なお、クラス内では定数名で参照できるし、クラス外でもクラス名::定数名で参照できる


定数を使った場合

```rb
class Product 

  # 定数は大文字とアンダースコアで書くこと
  # デフォルト価格を0円と定義
  DEFAULT_PRICE = 0

  attr_reader :name, :price

  def initialize(name, price = DEFAULT_PRICE)
    @name = name
    @price = price
  end

end
```

定数を使わない場合  

```rb
class Product 

  attr_reader :name, :price

  # 時間が経つと、なぜ price=0 としているのか失念してしまうかも
  # 0のところを1とタイプミスしたら。。。
  # 定数だとタイプミスしたらエラーとなって気がつく
  def initialize(name, price = 0)
    @name = name
    @price = price
  end

end
```

<br>

### 第７章の構成について  
---

さて、これ以降は例題を使いながら勉強していく。

#### 設定  

梅田、十三（じゅうそう）、三国の３つの駅がある。  
切符は、150円と190円の切符がある。  

どこで乗車して、どこで下車したかによって、料金が変わる。  
駅間が１駅だと150円、２駅だと190円。

買った切符で改札が通れるか判断する。

・・・以上のようなRubyのプログラムを作りながら、クラスについて学んでいく。  
また、テストプログラムも書きながら、確認を行っていく。  

--- 今日は７章の40％ぐらい進んだ ---

