# Railsテストに対する解説

## 問題 1

ActiveRecordについて正しい記述はどれか

- バリデーション(検証)もActiveRecordの機能の一つである
- データベースをオブジェクト指向スタイルで操作するものである
- RailsのモデルはActiveRecordを継承している
- Railsにおける基本的なルールとして、モデル名は複数形、テーブル名は単数形とする必要がある

### ActiveRecordの機能

Railsガイドに書いてある。  
ORM(Object-Relational Mapping)というものらしい。  

難しいことは分からないが、RubyとSQLをつなぐための仕組みと捉えればよいと思われる。  
RailsではActiveRecordだが、他のプログラミング言語にも似たようなものがあるらしい。  

ActiveRecordの機能は、以下に書いてあるとおり。  
詳しくは、Railsガイドを参照しよう。  

- モデルおよびモデル内のデータを表現する
- モデル同士の関連付け(アソシエーション)を表現する
- 関連付けられているモデル間の継承階層を表現する
- **データをデータベースで永続化する前にバリデーション(検証)を行なう**
- **データベースをオブジェクト指向スタイルで操作する**

- [ORMフレームワークとしてのActive Record](https://railsguides.jp/active_record_basics.html#orm%E3%83%95%E3%83%AC%E3%83%BC%E3%83%A0%E3%83%AF%E3%83%BC%E3%82%AF%E3%81%A8%E3%81%97%E3%81%A6%E3%81%AEactive-record)  

### RailsのモデルはActiveRecordを継承している

開発しているRailsのアプリを開き、適当なモデルのファイルを開いてみよう。  
こうなっているはず。継承してい流ことが分かる。  

```rb
class Product < ApplicationRecord
end
```

### Railsにおける命名ルール

- モデルのクラス - 単数形、語頭を大文字にする (例: BookClub)
- データベースのテーブル - 複数形、語はアンダースコアで区切られる (例: book_clubs)

面白いのは、Mouseクラスを作成した場合、きちんとデータベースはmiceにしなければならないところ。  
こういった不規則なものについても対応している。  

- [命名ルール](https://railsguides.jp/active_record_basics.html#%E5%91%BD%E5%90%8D%E3%83%AB%E3%83%BC%E3%83%AB)  

## 問題 2

モデルについて正しい記述はどれか

- ActiveRecordを継承している
- 複数形で定義する
- 対応するテーブルは単数形で定義する
- クラス名はケバブケースで書く

### ケバブケース・キャメルケース・スネークケース

| 名称         | 用例       | 主な言語                |                名称の由来                |
| ----------- | ---------- | --------------------- | --------------------------------------- |
| キャメルケース | camelCase  | Java, JavaScript      |   ラクダのコブみたいだから（大文字のがコブ）   |
| スネークケース | snake_case | PHP, Python, Ruby     |   ヘビみたいに下を這っているから             |
| ケバブケース  | kebab-case | HTML, CSS, Lisp       |    ケバブみたいに串刺しだから（焼き鳥的な）    |

気合で覚えると大変。以下のように覚えよう。  
多分、RubyやRailsでケバブケースって使わない。  

### 解説 2

基本的には、問題１で解説した内容でカバーしている。  

繰り返しになるが、モデルはキャメルケースで単数系。  
例えば、`QuizProblem`のようにすること。  

テーブルはスネークケースで複数形。  
例えば、`quiz_problems`のようにすること。  

## 問題 3

以下のうち正しい記述はどれか

- Railsではデータベースのテーブル名を探索するときにモデルのクラス名を複数形にした名前で探索する
- クラス名はケバブケースで書く
- 外部キーはテーブル名の複数形_idで定義する
- 主キーはオートインクリメントされる

### 解説 3

基本的には、問題１や問題２で解説したとおり。  
以下、補足を記載する。  

### データベースのテーブル名の探索方法

Railsでデータベースのテーブル名を探索する場合、どうなっているか。  
例えば、usersテーブルのデータを全て取得したい場合、Railsでどう書くか考えてみよう。  

```rb
# すべてのユーザーのコレクションを返す
users = User.all
```

このように、Rails上ではモデルのクラス名（単数系）をそのまま書くが、  
裏では、モデルのクラス名を複数形にしたusersテーブルを探している。  

### 外部キーはテーブル名の複数形_idで定義する？

Railsガイドに書いてあるとおり。  
テーブル名の単数系_idで定義する必要がある。  

Railsでテーブル間の紐付けをする場合、「`belongs_to`と`has_many`を書いて終わり」  
というようなことが多いので意識しづらいが、覚えておくようにしよう。  

例えば、usersとpostsというテーブルが紐づいている場合、  
postsテーブルにおいて指定されている外部キーは`user_id`になる。  
間違っても、`users_id`ではない。  

- [2.2 スキーマのルール](https://railsguides.jp/active_record_basics.html#%E3%82%B9%E3%82%AD%E3%83%BC%E3%83%9E%E3%81%AE%E3%83%AB%E3%83%BC%E3%83%AB)  

### オートインクリメント

そもそも聞くことがあまりない言葉かと思う。  
idの自動採番のことを言うらしい。  

主キーは、railsの方で自動的に付与するidになるので、  
**主キーはオートインクリメントされる。**

調べていないが、こうするとidを1000から始めることができるらしい。  

- [auto\_incrementの初期値を設定したい \- Qiita](https://qiita.com/w-tdon/items/fd147120eebf3cc22edd)  

## 問題 4

以下のうち文法的に正しいものを選べ。  

```rb
User.create(name: "David", occupation: "Code Artist")
```

```rb
user = User.new
user.name = "David"
user.occupation = "Code Artist"
user.save
```

```rb
user = User.new(name: "David", occupation: "Code Artist")
user.save
```

```rb
user = User.new(name="David", occupation="Code Artist")
user.save
```

### 解説 4

Railsガイドを見れば、全てが書いてある。  

- [5 CRUD: データの読み書き](https://railsguides.jp/active_record_basics.html#crud-%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AE%E8%AA%AD%E3%81%BF%E6%9B%B8%E3%81%8D)  

注意点だけ、以下に記す。  

- createメソッドは、newインスタンスを作り、saveするメソッド。  
- Rubyの話になるが、インスタンスの属性（attributes）を書き換える場合は4番目の選択肢のような書き方はしない。  
  - シンボルである`:name`に文字列である`"David"`を代入している
  - 厳密には違うようだが、ハッシュのようなものなので、本来的？な書き方はこうなる

```rb
user = User.new(:name => "David", :occupation => "Code Artist")
```

## 問題 5

以下のうちエラーになるものはどれか

```rb
users = User.where(name: "taro")
users.name
```

```rb
user = User.first
```

```rb
david = User.find(name: 'David')
```

```rb
users = User.where(name = 'David', occupation = 'Code Artist')
```

### 解説 5

ActiveRecordについて、Railsガイドに解説が書いてある。  

- [Active Record クエリインターフェイス](https://railsguides.jp/active_record_querying.html)  

また、各メソッドについて調べる場合、Railsドキュメントを活用するとよい。  

- [where \| Railsドキュメント](https://railsdoc.com/page/model_where)  
- [first \| Railsドキュメント](https://railsdoc.com/page/model_first)  
- [find \| Railsドキュメント](https://railsdoc.com/page/find)  

なお、以上の選択肢の内、３つ目の選択肢がエラーになる。  

詳細はRailsドキュメントのとおりだが、findメソッドはidしか引数として受け付けず、  
id以外のものである場合は例外処理となる。（例外処理にしたくない場合、find_byを使うとよい）  

### whereメソッド

- 条件を指定し、当てはまるレコードを全て取得するメソッド
- 条件については文字列で指定したり、ハッシュで指定することができる
- その他、配列で指定することができるので、ドキュメントで確認してみるとよい

### firstメソッド

- モデルの先頭のレコードを取得するメソッド
- 先頭のレコードを３つ取得することや、並び替えてから先頭のレコードを取得することもできる

### findメソッド

- IDを指定してレコードを取得
- 複数してしたり、配列で指定して取得することもできる

## 問題 6

名前を更新する処理として正しいものはどれか

```rb
user = User.find_by(name: 'David')
user.name = 'Dave'
user.save
```

```rb
User.find_by(name: 'David').update(name: 'Dave')
```

```rb
user = User.find(name: 'David')
user.name = 'Dave'
user.save
```

```rb
user = User.find_by(name: 'David')
user.update(name: 'Dave')
```

### 解説 6

Railsガイドに書いてある。  

- [5 CRUD: データの読み書き(Update)](https://railsguides.jp/active_record_basics.html#crud-%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AE%E8%AA%AD%E3%81%BF%E6%9B%B8%E3%81%8D)  

また、各メソッドについて調べる場合、Railsドキュメントを活用するとよい。  

- [find\_by \| Railsドキュメント](https://railsdoc.com/page/find_by)  
- [find \| Railsドキュメント](https://railsdoc.com/page/find)  

なお、以上の選択肢の内、３つ目の選択肢がエラーになる。  

詳細はRailsドキュメントのとおりだが、findメソッドはidしか引数として受け付けず、  
id以外のものである場合は例外処理となる。（例外処理にしたくない場合、find_byを使うとよい）  
（解説5でも説明したとおりである）  

## 問題 7

ユーザーを全削除する記述として正しいものはどれか

- `User.all.destroy`
- `User.destroy.all`
- `User.destroy_all`
- `User.delete!`

### 解説 7

Railsガイドに書いてある。  

- [5 CRUD: データの読み書き(Delete)](https://railsguides.jp/active_record_basics.html#delete)  

また、各メソッドについて調べる場合、Railsドキュメントを活用するとよい。  

- [destroy \| Railsドキュメント](https://railsdoc.com/page/model_destroy)  
- [destroy\_all \| Railsドキュメント](https://railsdoc.com/page/model_destroy_all)  
- [delete \| Railsドキュメント](https://railsdoc.com/page/model_delete)
- [delete\_all \| Railsドキュメント](https://railsdoc.com/page/model_delete_all)  

ただ、Railsドキュメントだと情報量が少ない部分があるので、英語になるが、RailsAPIを参照する方がよいかもしれない。  
（例えば、destroyメソッドなど）

なお、destroyとdeleteの違いは以下のとおりである。  

- destroy系はモデルを通すため、depedentしているモデルのデータについても併せて削除する  
- delete系はモデルを通さず、直接データベース上のデータを削除するため、dependentの影響を受けない  

また、それぞれのメソッドは、以下のとおりとなっている。  
なお、以下ではdestroyについて書いたが、deleteの場合も同様である。  

- destroy_allは、データを全て削除する場合に使う。
- destroy_byは、条件に合致するデータを全て削除する場合に使う。  
- destroyは、データを一つ削除する場合に使う
  - `User.first.destroy`といった書き方が一般的
  - `User.destroy(1)`といった書き方も可能である
  - `User.destroy([5,6])`とすれば、複数のデータを削除することもできる。  

よって、今回の場合、以下のとおりとなる。

- `User.all.destroy`
  - User.allのデータは一つではないのでエラーとなる
- `User.destroy.all`
  - クラスに対してdestroyする場合、引数がないのでエラーとなる。その後に`all`と書くのもおかしい。
- `User.destroy_all`
  - 正しい
- `User.delete!`
  - クラスに対するdeleteも、destroyと同様に引数にidがないとエラーとなる。

## 問題 8

以下のような定義がある場合の挙動として正しいものはどれか

```rb
# このようなクラスを前提とする
class User < ApplicationRecord
  validates :name, presence: true
end
```

```rb
user = User.new
user.save  # => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```

```rb
user = User.new
user.save! # => false
```

```rb
user = User.new
user.name = "DHH"
user.save # => true
```

```rb
User.create(name: "DHH") # =>  #<User:0x00007f7f8a4a3cb8 id: ~~~
```

### 解説 8

Railsガイドに書いてある。  

- [6 バリデーション（検証）- Railsガイド](https://railsguides.jp/active_record_basics.html#%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%EF%BC%88%E6%A4%9C%E8%A8%BC%EF%BC%89)  

> save、updateメソッドは、バリデーションに失敗するとfalseを返します。  
> このとき実際のデータベース操作は行われません。  
>
> 上のメソッドにはそれぞれ破壊的なバージョン (save!、update!) があり、  
> こちらは検証に失敗した場合にさらに厳しい対応、つまりActiveRecord::RecordInvalid例外を発生します。  

最初の２つの選択肢については、validationに抵触するため、  
非破壊的なメソッドの場合はfalseとなり、破壊的メソッドの場合は例外処理となるはずである。  
だが、いずれのメソッドも逆の結果を示しているので、挙動として正しいものではない。  

最後の２つの選択肢については、問題なくsaveやcreateができるケースであり、挙動として正しいものである。  
