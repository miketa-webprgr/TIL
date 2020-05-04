
## 現場Rails Chapter01 「RailsのためのRuby入門」
---

<BR>
<BR>
### クラスとインスタンス
---
クラス = メソッドのカテゴリ
インスタンス = クラスに属する事例

クラスとインスタンスは互い違いの関係ある。
  - 例：１のクラスはinteger
    - １というオブジェクトは、integerというクラスのインスタンス
      ```
      1.class
      => Interger
      ```
  - 例：田中太郎のクラスは人間（イメージです。。。）
    - 田中太郎というオブジェクトは、人間というクラスのインスタンス
      ```
      田中太郎.class
      => 人間
      ```

<BR>
### クラスは自分で作れる
---
なお、Integerのように既に定義されているクラスもあるが、自らクラスを作ることもできる。
```
Class Tanaka
  def firstname
    "太郎"
  end
end      
```
以上のとおり、Tanakaクラスとfirstnameメソッドを作成した。
tanakaというTanakaクラスに属するオブジェクトを作成した場合、firstnameメソッドを使えば、太郎と出力される。
```
tanaka = Tanaka.new
tanaka.firstname
=> 太郎
```
なお、tanakaオブジェクトでなくとも、Tanakaクラスになってfirstnameメソッドを使った時点で太郎と出力される。
```
abdefghijkl = Tanaka.new
abdefghijkl.firstname
=> 太郎
```

<BR>
### 変数は固定するために使う
---
以下はあくまでイメージである。
同じ文字であっても、以下のとおり違うオブジェクトとして扱われてしまう。

```
”田中”.object_id
=> 147891654
”田中”.object_id
=> 489481648
```

同じオブジェクトとして扱うためには、変数に代入していく必要がある。
```
tanaka = ”田中”
tanaka.object_id
=> 147891654
tanaka.object_id
=> 147891654
```

<BR>
### ローカル変数とインスタンス変数
---
ローカル変数は、使い捨てである。
メソッドをまたいで、その変数は引き継がれることがない。
```
Class Tanaka
  def tanaka_1
    tanaka = "田中"
  end
  def tanaka_2
    tanaka
  end
end
```

以上のように、Tanakaクラスにtanaka_2メソッドを適用した場合、田中と出力されるのを期待してしまう。
だが、tanaka_1メソッドから変数が引き継がれないので、エラーとなってしまう。

```
Class Tanaka
  def tanaka_1
    @tanaka = "田中"
  end
  def tanaka_2
    tanaka
  end
end
```
そこで、以上のように「＠」を使ってインスタンス変数とすることが必要である。

<BR>
### Attr_accessor
---
以下の二つはイコールである。

```
class User
  attr_accessor :name
end
```
```
class User
    def name= (name)
      @name = name
    end

    def name
      @name
    end
end
```
なお、インスタンス変数に代入するメソッドをセッターという。
また、インスタンス変数の内容を参照するメソッドをゲッターという。

<BR>
### メソッドのメソッド
-----

メソッドからメソッドを使うことができる。
その時は、以下のように書くことができる。

```
class User
    attr_accessor :name, :address
    def profile
      "#{name}(#{address})"
    end
end
```

<BR>
### 演算子
---

よく把握していなかったものを記載する。
- 「%　→　剰余」
- 「||　→　or」
- 「&&　→　and」
- 「^　→　XOR」
- 「!　→　not」
- 「!=　→　等しくないか調べる」

なお、「+」「-」「*」「/」は、知ってのとおりであるが、
  - 「+」は文字列や配列の連結にも使える
  - 「*」で文字列や配列を繰り返し連結するのにも使える

<BR>
### 配列とハッシュ
-----
Array＝配列
  - << で配列に追加できる

ハッシュ
  - 内部的にデータとキーを結びつけおく
  - 配列には色々なメソッドがある

````
{tokyo: 13636222, kanagawa: 9145572}
# 他の記法もある
````

ハッシュから値を取得する時は[:tokyo]とする

````
options = {tokyo: 13636222, kanagawa: 9145572}
puts options[:tokyo] 
````

なお、puts keyで tokyoとkanagawaが出力できる。
また、puts value で136…と914…が出力できる。

<BR>
### 引数にデフォルトを設定する
-----

```` 
def name(full, with_age)
  n = if full
        “#{family_name}” #{given_name}”
      else
        #{given_name}”
      end
  n<< “(#{age})” if with_age
  n
end
````

というメソッドのにおいてフルネームと年齢を表示したい場合、
下記のような表記をする必要がある。

````
person.name(true, true)
````

ただ、毎回設定するのが面倒なので、下記のようにデフォルトを設定できる。
````
def name( full = true, with_age = true)
````

この場合、下記のような表記でフルネームも名前も表示される。

````
person.name
````

キーワード引数の場合だと、以下のような表記になる。
````
def name (full: true, with_age: true)
````

キーワード変数の場合、このようにして下の名前だけを表示することができる。

````
person.name(full:false, age:false)
````
<BR>
### クラスとモジュール
-----

クラスやモジュールを使うことで、共通するメソッドを使い回すことができる。

クラスを使い回す（継承する）場合、以下のように記載する。
例としてはかなり微妙だが、firstnameメソッドが引き継がれる。

```
class Taro
  def firstname
    @taro = "太郎"
  end
end

Class Taro < Tanaka
  def name
    tanakataro =  "田中" + @taro
    tanakataro
  end
end
```
クラスは、小クラスに共通するメソッドを記載する際に役立つ。
単にクラスの部品として書きたいような場合は、モジュールを使うとよい。
