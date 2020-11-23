# 自己結合を使って、カレーセットとカレーを関係付けてみた

## GitHubレポジトリ

以下のとおりです。  

- [miketa\-webprgr/gozenapp: カレーセットを表現するためだけの練習アプリ](https://github.com/miketa-webprgr/gozenapp)

git cloneをした後、以下を実行してください。  

```rb
rails db:create
rails db:migrate
rails db:seed
```

その後、rails consoleでガチャガチャ試すことができます。  
なお、View側は何も作っていません。。。

## はじめに

先日、東京でオフラインのもくもく会が開催されました。  

もくもく会では、主に「テーブル設計」に関する議論を行いました。  
餃子の王将の注文書を見ながらテーブル設計を行うなど、実践的な練習を行いました。  

そこで、少し話題となったのが「華御前」などのセットものが入った場合のテーブル設計です。  

「親子関係が入ると途端に難しくなるから」ということで、この領収書に関するテーブル設計や実装は話題に  
なりませんでしたが、自己結合を使ってもいけそうといった話をして終わりになりました。  

<img src="https://github.com/miketa-webprgr/TIL/blob/master/99_etc/20201111_curry_set_app.png?raw=true" width="600"/><br>  

自己結合は、フォローとアンフォローの関係を表現する場合によく使われます。  
ユーザー同士の関係性を表すので、「自己結合」と言うらしいです。  
（ただ、中間テーブルを介する形なので、正確には自己結合とは言わないかも）  

Railsガイドでは、「1つのデータベースモデルに全従業員を格納しておきたいが、  
マネージャーと部下(subordinate)の関係も追えるようにしておきたい場合」が例として挙げられています。  
（これは、中間テーブルを介さない純粋な自己結合に該当する）  

- [Active Record の関連付け \- Railsガイド](https://railsguides.jp/association_basics.html#%E8%87%AA%E5%B7%B1%E7%B5%90%E5%90%88)

「難しくなるから」と言われると、なんとなく挑戦したくなります。  

ということで、「1つのデータベースモデルに全メニューを格納しておきたいが、  
『華御前』と『前菜』『焼物』『かに冷し鉢』・・・の関係も追えるようにしておきたい場合」の  
テーブル設計とアソシエーションの実装に挑戦してみました。  
（通常であれば、あまりセットメニューと単品メニューを同じテーブルに格納しないのでしょうけど）  

そこで、アソシエーション部分のみですがなんとか実装できたので、みなさんに共有します。  

## 単純化してカレーセットとコーヒーセットしかない喫茶店を想定して考えてみる

難しいことを考えると頭がパンクするので、以下のメニューを提供している喫茶店を想定してみました。  
なお、カレーセットは「カレー・サラダ・コーヒー」、ケーキセットは「ケーキ・コーヒー」で構成されるものとします。  

|メニュー   |値段     |
|----------|-------:|
|カレーセット|   1,500|
|ケーキセット|     600|
|カレー     |   1,200|
|サラダ     |     300|
|コーヒー   |     300|
|ケーキ     |     400|

自己結合の考え方を使って、カレーセットをカレー・サラダ・コーヒーに紐づけることができます。  
具体的には、こんな感じです。  

itemsテーブル  

|ID |メニュー   |値段     |
|---|----------|-------:|
|1  |カレーセット|   1,500|
|2  |ケーキセット|     600|
|3  |カレー     |   1,200|
|4  |サラダ     |     300|
|5  |コーヒー   |     300|
|6  |ケーキ     |     400|

relationshipsテーブル  

|ID |item_id(1) |item_id(2) |
|---|----------:|----------:|
|1  |1          |          3|
|2  |1          |          4|
|3  |1          |          5|
|4  |2          |          5|
|5  |2          |          6|

少し解説すると、item_id(1)にはカレーセットなどのセットメニューに関するidが入ります。  
item_id(2)にはカレーやサラダなどのセットメニューに紐づくidが入ります。  

このようなテーブルを作ることによって、カレーセットに紐づく単品メニューのid [3, 4, 5] を取得し、  
そのメニュー名である「カレー・サラダ・コーヒー」を取得することができます。  

逆に、コーヒーに紐づくセットメニューを取得することもできます。  
コーヒーのidは5なので、item_id(2)に5が入っているレコードを参照します。  

すると、item_id(1)には [1, 2] が入っているので、  
コーヒーに紐づくメニューとして、「カレーセット・ケーキセット」を取得することができます。  

なお、SQLを発行しなければいけないというパフォーマンス上の問題がありますが、  
セットメニューであるかの判別は、item_id(1)カラムへのデータの有無で判断できます。  
もしくは、逆の発想で、item_id(2)カラムへのデータの有無でも判断できます。  

## Railsアプリにおけるassociationの実装について

Schemaは下記のとおりとしました。  
命名は、ちょっと分かりづらいかったかもしれないです。  

なお、本来は外部キー制約を設ける方がベターなんですが、設定方法が少し特殊で勉強が必要であったため、  
とりあえず今回は外部キー制約を設けない形で実装を進めました。（自己結合は色々と面倒です・・・）  

```rb
ActiveRecord::Schema.define(version: 2020_11_07_134240) do

  create_table "items", force: :cascade do |t|
    t.string "menu"
    t.integer "price"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

  create_table "relationships", force: :cascade do |t|
    t.integer "set_menu_id", null: false
    t.integer "food_id", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["food_id"], name: "index_relationships_on_food_id"
    t.index ["set_menu_id"], name: "index_relationships_on_set_menu_id"
  end

end

# なお、以下を参照すると外部キー制約も設定できそうです。  
# [Suinさんのブログ（「同じモデルに複数外部キー制約をつける場合」）](https://suin.io/547)
```

次にItemモデルとRelationshipモデルですが、以下のとおり実装しました。  

```rb
class Item < ApplicationRecord

# 【前段部分】

  # カレーセットに属するfood_idを取得するassociation
  has_many :food_relationships, class_name: 'Relationship', foreign_key: 'set_menu_id', dependent: :destroy

  # コーヒーが属するset_menu_idを取得するassociation
  has_many :set_menu_relationships, class_name: 'Relationship', foreign_key: 'food_id', dependent: :destroy

# 【後段部分】

  # カレーセットからカレー・サラダ・コーヒーを取得する
  has_many :foods, through: :food_relationships

  # カレーからカレーセットを取得する
  has_many :set_menus, through: :set_menu_relationships
end
```

```rb
class Relationship < ApplicationRecord
  belongs_to :food, class_name: 'Item'
  belongs_to :set_menu, class_name: 'Item'
end
```

itemsテーブル  

|ID |メニュー   |値段     |
|---|----------|-------:|
|1  |カレーセット|   1,500|
|2  |ケーキセット|     600|
|3  |カレー     |   1,200|
|4  |サラダ     |     300|
|5  |コーヒー   |     300|
|6  |ケーキ     |     400|

relationshipsテーブル  

|ID |set_menu_id |food_id |
|---|----------:|----------:|
|1  |1          |          3|
|2  |1          |          4|
|3  |1          |          5|
|4  |2          |          5|
|5  |2          |          6|

## Associationの説明

前段部分と後段部分に分けて説明します。  

前段部分とは、`has_many :food_relationships`と`has_many :set_menu_relationships`のことです。  
後段部分とは、`has_many :food`と`has_many :set_menu`のことです。  

### relationshipsテーブルのレコードを取得するassociation

通常、has_manyにてassociationを定義する場合、relationshipsテーブルにitem_idという  
association元の外部キーを持たせることが一般的です。  

この場合、`has_many :relationships`と書くだけでいいので、シンプルに実装することができます。  

ただ、今回の場合、relationshipsテーブルには外部キーを２つ設けており、  
それぞれをitem_idという命名にするわけにいきません。  

そこで、外部キーをset_menu_idとfood_idと命名しました。  

外部キーがassociation元のモデル名と異なるので、foreign_keyが何かを明示する必要があります。  

そこで、セットメニューから単品メニューのidを取得するassociationについては、  
`foreign_key: 'set_menu_id'`と明示しました。  

逆についても然りで、単品メニューからセットメニューのidを取得するassociationについては、  
`foreign_key: 'food_id'`と明示しました。

また、どちらのassociationについても、`has_many :relationships`と書いてしまう  
わけにはいかないので、それぞれ`has_many :food_relationships`及び`has_many :set_menu_relationships`  
といった形で別名をつけました。  

別名をつけると、どこのモデルを参照しているか不明になるため、`class_name: 'Relationship'`と明示しました。  

### Relationshipモデルのアソシエーション設定

アソシエーション元であるRelationshipモデルについも、定義してあげる必要があります。  

通常であれば、`belongs_to :items`とするだけで問題ありません。  
ただ、今回の場合、書き分けてあげないとrelationshipのレコードからassociationを取得することができません。  
（`Relationship.find(1).item`というようなアソシエーションを書いた場合のことを考えてみよう）  

ということで、以下のとおり設定しましょう。  

```rb
class Relationship < ApplicationRecord
  belongs_to :food, class_name: 'Item'
  belongs_to :set_menu, class_name: 'Item'
end
```

food_idを外部キーとした場合にアソシエーション元のレコードを取得する場合、  
`@relationship.food`で取得できます。  

set_menu_idを外部キーとした場合にアソシエーション元のレコードを取得する場合、  
`@relationship.set_menu`で取得できます。  

なお、`belongs_to :tanpin_menu`とすることもできますが、その場合、その命名から外部キーを推測できないので、  
`belong_to :tanpin_menu, class_name: 'Item', foreign_key: 'food_id'`とする必要があります。  

### Relationshipを通じて、紐づくmenusレコードを取得するassociation

これまで実装したassociationのみだと、紐づくidしか取得できず、本当に必要なメニュー名や値段を取得することができません。  
そこで、`has_many :foods, through: :food_relationships`と記述する必要が出てきます。  

ここでは、既に実装したfood_relationshipsを使って、food_idに紐づくitemsテーブルのレコードを取得します。  

例えば、`Item.find_by(menu:'カレーセット').foods`とrails consoleで打ち込んだ場合、以下のSQLが発行されます。  

なお、LIMITについてはRailsの仕様なのか自動で入るようです。  
また、11というのはレコードの数を指しているようです（推測）。  

```sql
SELECT "items".*
  FROM "items"
  WHERE "items"."menu" = ?
  LIMIT ?  [["menu", "カレーセット"], ["LIMIT", 1]]
SELECT "items".*
  FROM "items"
  INNER JOIN "relationships"
  ON "items"."id" = "relationships"."food_id"
  WHERE "relationships"."set_menu_id" = ?
  LIMIT ?  [["set_menu_id", 1], ["LIMIT", 11]]
```

ここでやっている作業としては、INNER JOINとWHEREです。  
INNER JOIN をした後に、WHERE句で条件をかけて範囲を絞っています。  

#### INNER JOINについて

私は、以下のような作業が行われていると想定していました。  

1. food_relationshipsを使って必要なrelationshipsテーブルのレコードを取得
2. そのレコードとitemsテーブルの全レコードをINNER JOINしてitemsテーブルから必要なレコードを取得

しかし、実際には逆の順序で、まずINNER JOINを行った後に、WHEREで条件を設けて絞る作業を行っていました。  
それは先ほど記載したSQLのとおりです。  

とはいっても、SQLだけではイメージしづらいので、順を追って具体的に確認しましょう。  
rails consoleでこのコマンドを打ち込むと、INNER JOIN部分だけが確認できます。  

```rb
Item.joins('INNER JOIN "relationships" ON "items"."id" = "relationships"."food_id"')
```

すると、以下のfood_idカラムにあるidに紐づく、itemsテーブルのレコードを返します。  
（正確には、itemsテーブルにあるitem_idとrelationshipsテーブルにあるfood_idが一致するものだけjoinsして、  
  itemsテーブルのカラムの値を全てSELECTしています）  

|food_id    |
|----------:|
|          3|
|          4|
|          5|
|          5|
|          6|

結果、返り値として以下を取得できます。  
これは、カレーセットに紐づく単品とケーキセットに紐づく単品の一覧になります。  

```rb
<ActiveRecord::Relation [
  <Item id: 3, menu: "curry", price: 1200, created_at: "2020-11-07 18:11:36", updated_at: "2020-11-07 18:11:36">,
  <Item id: 4, menu: "salad", price: 300, created_at: "2020-11-07 18:11:36", updated_at: "2020-11-07 18:11:36">,
  <Item id: 5, menu: "coffee", price: 300, created_at: "2020-11-10 16:43:31", updated_at: "2020-11-10 16:43:31">,
  <Item id: 5, menu: "coffee", price: 300, created_at: "2020-11-10 16:43:31", updated_at: "2020-11-10 16:43:31">,
  <Item id: 6, menu: "cake", price: 400, created_at: "2020-11-07 18:11:37", updated_at: "2020-11-07 18:11:37">
]>
```

#### WHEREについて

これにWHERE句をかけて、カレーセットに紐づく単品に限定します。  
`Item.find_by(menu:'カレーセット')`をして得られた`'relationships.set_menu_id = ?', 1`を活用します。  

```rb
Item.joins('INNER JOIN "relationships" ON "items"."id" = "relationships"."food_id"').where('relationships.set_menu_id = ?', 1)
```

relationshipsテーブルの内、以下のレコードに紐づくものに限定して、itemsテーブルからレコードを取得します。  

|ID |set_menu_id |food_id |
|---|----------:|----------:|
|1  |1          |          3|
|2  |1          |          4|
|3  |1          |          5|

結果、カレーセットに紐づく、以下の３つのレコードが取得できます。  

```rb
<ActiveRecord::Relation [
  <Item id: 3, menu: "curry", price: 1200, set_menu: false, created_at: "2020-11-07 18:11:36", updated_at: "2020-11-07 18:11:36">,
  <Item id: 4, menu: "salad", price: 300, set_menu: false, created_at: "2020-11-07 18:11:36", updated_at: "2020-11-07 18:11:36">,
  <Item id: 5, menu: "coffee", price: 300, set_menu: false, created_at: "2020-11-10 16:43:31", updated_at: "2020-11-10 16:43:31">,
]>
```

#### 補足： throughについて

なお、`has_many: foods`という命名ではなく、例えば`has_many: tanpin`としたい場合、  
`has_many :tanpin, through: :food_relationships, source: :food`と書く必要があります。  

`has_many :tanpin`とすると、参照元となる`belongs_to :tanpin`が見つからないので、  
エラーとなってしまいます。そこで、`source: :food`と書き、参照元を明示してあげる必要があるのです。  

以下の説明が分かりやすいかったです。

- [Understanding :source option of has\_one/has\_many through of Rails \- Stack Overflow](https://stackoverflow.com/a/4632456)
- [【初心者向け】丁寧すぎるRails『アソシエーション』チュートリアル【幾ら何でも】【完璧にわかる】🎸 \- Qiita](https://qiita.com/kazukimatsumoto/items/14bdff681ec5ddac26d1#source)

### カレーがメニューから消えたとき、カレーセットも自動的に削除したい！  

さて、改めてassociationの設定状況を確認しましょう。  

```rb
class Item < ApplicationRecord
  # カレーセットに属するfood_idを取得するassociation
  has_many :food_relationships, class_name: 'Relationship', foreign_key: 'set_menu_id', dependent: :destroy

  # コーヒーが属するset_menu_idを取得するassociation
  has_many :set_menu_relationships, class_name: 'Relationship', foreign_key: 'food_id', dependent: :destroy

  # カレーセットからカレー・サラダ・コーヒーを取得する
  has_many :foods, through: :food_relationships

  # カレーからカレーセットを取得する
  has_many :set_menus, through: :set_menu_relationships
end
```

```rb
class Relationship < ApplicationRecord
  belongs_to :food, class_name: 'Item'
  belongs_to :set_menu, class_name: 'Item'
end
```

以上のとおり、`dependent: :destroy`と設定しているので、以下のように作動します。  

- カレーが削除されたら、カレーとカレーセットの関係性に関するレコードも自動的に削除
- カレーセットが削除されたら、カレーセットとカレー・サラダ・コーヒーとの関係性に関するレコードも自動的に削除

ただ、ここで問題となるのが、カレーが消えた場合のカレーセットの存在です。  
通常であれば、カレーが消えた場合、カレーセットも売れなくなってしまうので、カレーセットを自動的に削除したいかと思います。  

#### through先にもdependent: :destroyしてみる

そこで、下記のとおり設定してみました。  

```rb
  # カレーからカレーセットを取得する
  has_many :set_menus, through: :set_menu_relationships, dependent: :destroy
```

この設定をしてカレーのレコードを削除したところ、カレーセットは削除されることはありませんでした。  
調べたところ、`has_many through`しているレコードに`dependent: :destroy`は機能しないようです。  

- [参考ブログ](https://aikawame.hateblo.jp/entry/2019/03/07/has_many_%3Athrough%E6%99%82%E3%81%AEdependent%3A_%3Adestroy%E3%81%AE%E6%8C%99%E5%8B%95)
- [stackoverflow]https://stackoverflow.com/questions/10039880/has-many-through-association-dependent-destroy-under-condition-of-who-called-des

#### コールバックで対応する

そこで、コールバックを使えばよいのではないかと考え、以下のように実装してみました。  

```rb
class Item < ApplicationRecord
  before_destroy :destroy_related_set_menus, unless: :set_menu?

  # カレーセットに属するfood_idを取得するassociation
  has_many :food_relationships, class_name: 'Relationship', foreign_key: 'set_menu_id', dependent: :destroy

  # コーヒーが属するset_menu_idを取得するassociation
  has_many :set_menu_relationships, class_name: 'Relationship', foreign_key: 'food_id', dependent: :destroy

  # カレーセットからカレー・サラダ・コーヒーを取得する
  has_many :foods, through: :food_relationships

  # コーヒーからカレーセットやケーキセットを取得する
  has_many :set_menus, through: :set_menu_relationships

private

  def destroy_related_set_menus
    set_menus.destroy_all
  end

  def set_menu?
    foods.exists?
  end
end
```

ただ、残念ながらこれもうまくいきませんでした。  
削除されるのは、relationshipsテーブルのレコードのみでした。  

調べたところ、以下のような記事が出てきました。  

- [Ruby on Rails: how to efficiently delete a mass of associated objects by has\_many :through \| by Xiuzhong LI \| Medium](https://medium.com/@lxz.tty/ruby-on-rails-how-to-effectively-purge-associated-objects-by-has-many-through-d597850fb03f)

読んでみたところ、そもそも孫レコードにあたるカレーセットを削除する前に、子コードである  
カレーとカレーセットの関係性に関するrelationshipsテーブルのレコードが削除されてしまうので、  
孫レコードが参照できない、と書いてありました。  

#### コールバックで発火するメソッドを修正してみる

そこで、以下のとおりコールバック関数を利用したらうまくいきました。  

```rb
class Item < ApplicationRecord
  before_destroy :destroy_related_set_menus, unless: :set_menu?

  # カレーセットに属するfood_idを取得するassociation
  has_many :food_relationships, class_name: 'Relationship', foreign_key: 'set_menu_id', dependent: :destroy

  # コーヒーが属するset_menu_idを取得するassociation
  has_many :set_menu_relationships, class_name: 'Relationship', foreign_key: 'food_id', dependent: :destroy

  # カレーセットからカレー・サラダ・コーヒーを取得する
  has_many :foods, through: :food_relationships

  # コーヒーからカレーセットやケーキセットを取得する
  has_many :set_menus, through: :set_menu_relationships

private

  def destroy_related_set_menus
    Item.where(id: set_menu_ids).destroy_all
  end

  def set_menu?
    foods.exists?
  end
end
```

アソシエーションは難しいですね。。。  

## 最後に

狙ったことは実装できましたが、以下については議論の余地がありそうです。  

- そもそもこのようなテーブル設計でいいのか
- コールバックはあまり乱用しないほうがよいと聞くことがあるので、これでよかったのか

以上については今後の課題にして、とりあえず終わりとしたいと思います。  
