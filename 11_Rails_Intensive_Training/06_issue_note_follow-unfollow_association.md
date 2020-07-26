# フォロー・アンフォローのアソシエーション

## 2.1 relationshipsテーブルについて

まず、Qiita記事やRailsチュートリアルに書かれているとおりだが、  
relationshipsテーブルについて理解できているか確認する。  

relationshipsテーブルは、いいね機能実装でも取り扱った中間テーブルであり、  
ここには関係性を示すため、フォローするユーザーidとフォローされるユーザーidが  
カラムとなることを抑える必要がある。  

relationshipsテーブル（created_atなどは省略）

|  id  | user_id（フォローする人） | user_id（フォローされる人）|
| ---- | ---------------------- | ----------------------- |
|   1  |          1             |            2            |
|   2  |          1             |            3            |
|   3  |          3             |            2            |

## 2.2 大枠として、多対多であることを捉える

色々と書いてあるが、多対多であることは変わらないので、順を追って理解する。  
まず、フォローしているユーザーを取得するための実装について、短絡的に考えてみる。  
（フォロワーであるユーザーを取得するための実装については一旦考えないこととする）  

```rb
# user.rb
# 該当部分のみ
  has_many :relationships
  has_many :following, through: :relationships, source: :user
```

```rb
# relationship.rb
# 該当部分のみ
  belongs_to :user # follower_idカラムに紐付け
  belongs_to :user # followed_idカラムに紐付け
```

ここでの狙いは、中間テーブルであるrelationshipsテーブルを使って、  
フォローしているユーザーを獲得することである。  

前回の実装において、いいねをした投稿を取得しようとした際に以下のような  
コードを書いたが、やろうとしていることは同様である。  

```rb
# user.rb
# 該当部分のみ
  has_many :likes
  has_many :like_posts, through: :likes, source: :post
```

```rb
# post.rb
# 該当部分のみ
  belongs_to :user
  has_many :likes
```

```rb
# like.rb
# 該当部分のみ
  belongs_to :user # usersテーブルのidカラムに紐付け
  belongs_to :post # postsテーブルのidカラムに紐付け
```

また、ここで改めてそれぞれのオプションの意味について、Railsドキュメントで確認しておくとよい。  
[Railsドキュメント (has_many)](https://railsdoc.com/page/has_many)  

## 2.3 フォローしているユーザーを獲得するためのassociaionを考える

ただ、冷静に考えてみると、今回の試みには問題があることに気づく。  

まず、明らかにおかしいRelationshipモデルについて考える。  
`belongs_to :user`と２回書いてあるが、relationshipsのどちらのカラムに紐づくか不明であるため、  
区別できるように適切な名称を与えた後、どのモデル紐づくか分かるよう`class_name: 'User'`を加える。  

```rb
# relationship.rb
# 該当部分のみ
# belongs_to の名称を与えて、参照するモデルクラスを指定した
  belongs_to :follower, class_name: 'User' # follower_idカラムに紐付け
  belongs_to :followed, class_name: 'User' # followed_idカラムに紐付け
```

次に、Userモデルについて考えてみる。  

エンドポイントから逆算して考えてみる。  
今回、取得を目指しているのが、該当のユーザーがフォローしているユーザーの情報である。  

具体的には、以下のプロセスを経るはずである。  

1. usersテーブルのidと紐づく、relationshipsテーブルのfollower_idを使って、followed_idを取得。
2. followed_idを使って、usersテーブルのidと紐づくusersの情報を取得。

第１のプロセスは、relationshipsテーブルを経由して行われるが、  
そもそもusersテーブルのidをどちらと紐づければよいのか、  
`through: :relationships`と言われても、Rails側では判断のしようがない。  

そこで、どちらと紐づければよいのか判断できるように、参照元の外部キーを指定する。  
今回の場合、relationshipsテーブルのfollower_idを使って、followed_idを取得したいので、  
`foreign_key: 'follower_id'`と指定する。  

```rb
# user.rb
# foreign_key を追加
  has_many :relationships, foreign_key: 'follower_id'
  has_many :following, through: :relationships, source: :user
```

また、第２のプロセスでは、「followed_idを使って、usersテーブルのidと紐づくusersの情報を取得」  
しようとするが、`source: :user`ではrelationshipsテーブルにあるどちらのカラムを使って、  
usersテーブルの紐付けするか判別することができない。  

カラムが２つしかないのだから、'followed_id'のカラムを使って欲しいに決まっているのだが、  
さすがのRailsさんでもそこまで良しなにやってくれないもよう。。。

そこで、`source: :followed`ときちんと指定して、relationshipsテーブルにおいて紐付けをしたい  
カラムを明確にしてあげることが必要である。  

```rb
# user.rb
# source: :followed に変更
  has_many :relationships, foreign_key: 'follower_id'
  has_many :following, through: :relationships, source: :followed
```

## 2.4 実装結果（フォローしているユーザーを獲得するためのassociaion）

結論。  
以下のとおりコードを書くと、フォローしているユーザーを獲得するための実装が完了する。  

```rb
# user.rb
# 該当部分のみ
  has_many :relationships, foreign_key: 'follower_id'
  has_many :following, through: :relationships, source: :followed
```

```rb
# relationship.rb
# 該当部分のみ
  belongs_to :follower, class_name: 'User' # follower_idカラムに紐付け
  belongs_to :followed, class_name: 'User' # followed_idカラムに紐付け
```

`rails console`上で以下のように入力すると、フォローしているユーザーの情報が取得できる。  

```text
[1] pry(main)> User.first.following

# 発行されるSQL
=> "SELECT `users`.* FROM `users` INNER JOIN `relationships` ON `users`.`id` = `relationships`.`followed_id` WHERE `relationships`.`follower_id` = 1"

# フォローしているユーザー情報が以下のように取得できるはず
=> [#<User:0x00007f9116cd7b58
  id: 3,
  email: "alice@example.com",
  username: "alice",
  created_at: Sat, 04 Jul 2020 13:28:59 JST +09:00,
  updated_at: Sat, 04 Jul 2020 13:28:59 JST +09:00>,
 #<User:0x00007f9116cd7a18
  id: 7,
  email: "camila@example.com",
  username: "camila",
  created_at: Sat, 04 Jul 2020 13:28:59 JST +09:00,
  updated_at: Sat, 04 Jul 2020 13:28:59 JST +09:00>,

# フォローしているユーザーがいない場合
=> []
```

## 2.5 道を外れてみる 1 （`belongs_to :experiment`と`source: :experiment`に変更してみる）

以下のようなケースにおいて、上手く機能するのか気になった。  

「relationshipsテーブルにおいて紐付けをしたいカラムをしっかりと指定してあげることが必要」  
と書いたが、もしかしたら`belongs_to :`を正しく指定すれば、relationshipsテーブルのカラム名に  
係らず、associationは上手く機能するのではないかと考えたからだ。  
  
道を外れて、`belongs_to :experiment`と`source: :experiment`に変更した上で問題が起きないか  
検証してみることにした。  

```rb
# user.rb
# 該当部分のみ
# source: :experiment にあえて変更
  has_many :relationships, foreign_key: 'follower_id'
  has_many :following, through: :relationships, source: :experiment
```

```rb
# relationship.rb
# 該当部分のみ
# belongs_to :experiment にあえて変更 
  belongs_to :follower, class_name: 'User' # follower_idカラムに紐付け
  belongs_to :experiment, class_name: 'User' # followed_idカラムに紐付け
```

結果、こうなった。  
relationshipsテーブルにexperiment_idはないので、フォローしているusersデータは取得できない。  

```
[1] pry(main)> User.first.following.to_sql

=> "SELECT `users`.* FROM `users` INNER JOIN `relationships` ON `users`.`id` = `relationships`.`experiment_id` WHERE `relationships`.`follower_id` = 1"
```

## 2.6 道を外れてみる 2 （`source: :follower`に変更してみる）

以下のようなケースにおいても、上手く機能するのか気になった。  

既に述べたとおり、`foreign_key: :'followed_id'`と書いてあるのだから、  
「`source: :follower`になる訳ないだろう、Rails側で良しなになぜ対応してくれないのだろう」  
と不思議に思っていた。  

そこで、そもそも`source: :follower`と書いたらどうなるのか、エラーでも起きるのかと  
思って実験してみた。  

```rb
# user.rb
# 該当部分のみ
  has_many :relationships, foreign_key: 'follower_id'
  has_many :following, through: :relationships, source: :follower
```

```rb
# relationship.rb
# 該当部分のみ
  belongs_to :follower, class_name: 'User'
  belongs_to :followed, class_name: 'User'
```

結果、こうなった。  
フォローしているユーザーの数だけ、自身のユーザー情報を取得できた。（誰得。。。）  

```
[1] pry(main)> User.first.following.to_sql

=> "SELECT `users`.* FROM `users` INNER JOIN `relationships` ON `users`.`id` = `relationships`.`follower_id` WHERE `relationships`.`follower_id` = 1"

=> [#<User:0x00007fa9f885e478
  id: 1,
  email: "miketa@example.com",
  username: "miketa",
  created_at: Mon, 29 Jun 2020 17:41:24 JST +09:00,
  updated_at: Mon, 29 Jun 2020 17:41:24 JST +09:00>,
 #<User:0x00007fa9f885e298
  id: 1,
  email: "miketa@gmail.com",
  username: "miketa",
  created_at: Mon, 29 Jun 2020 17:41:24 JST +09:00,
  updated_at: Mon, 29 Jun 2020 17:41:24 JST +09:00>,
```

## 2.7 フォローしてくれているユーザーを獲得するためのassociaionを考える

フォローしているユーザーの獲得方法についてきちんと理解すれば、  
フォローしてくれているユーザーの獲得方法についてのassociationも簡単に書ける。  

改めてエンドポイントを考えてみる。  

1. usersテーブルのidと紐づく、relationshipsテーブルの**followed_id**を使って、follower_idを取得。
2. **follower_id**を使って、usersテーブルのidと紐づくusersの情報を取得。

そして、前回と変わった部分だけ、実装も変える。  
すると、コードは以下のとおりとなる。  

```rb
# user.rb
# foreign_key と source を変更
  has_many :relationships, foreign_key: 'followed_id'
  has_many :followers, through: :relationships, source: :follower
```

```rb
# relationship.rb
# 変更なし
  belongs_to :follower, class_name: 'User' # follower_idカラムに紐付け
  belongs_to :followed, class_name: 'User' # followed_idカラムに紐付け
```

## 2.8 これまで書いたassociationのコードを合体させてみる

表題のとおり、これまで書いたassociationのコードを合体させてみる。  
すると、単純に合体させる訳にはいかないことが分かる。  

```rb
# user.rb
# 該当部分のみ
  has_many :relationships, foreign_key: 'follower_id'
  has_many :following, through: :relationships, source: :followed
  has_many :relationships, foreign_key: 'followed_id'
  has_many :followers, through: :relationships, source: :follower
```

```rb
# relationship.rb
# 該当部分のみ
  belongs_to :follower, class_name: 'User' # follower_idカラムに紐付け
  belongs_to :followed, class_name: 'User' # followed_idカラムに紐付け
```

どこが問題なのかというと、以下の部分である。  

- `has_many :relationships, foreign_key: 'follower_id'`

以上のコードは、以下のコードにより上書きされてしまっているのである。  

- `has_many :relationships, foreign_key: 'followed_id'`

これでは困るので、relationshipsにそれぞれ名称を与えて、使い分けができるように設定する必要がある。  

以下のようにactive_relationshipsとpassive_relationshipsの名称を与え、`has_many :following`  
や`has_many :followrs`をする場合のthroughにて正しいものを指定してあげれば、問題なく作動する。  

```rb
# user.rb
# 該当部分のみ
  has_many :active_relationships, class_name: 'Relationship',
                                  foreign_key: 'follower_id'
  has_many :following, through: :active_relationships, source: :followed

  has_many :passive_relationships, class_name: 'Relationship',
                                   foreign_key: 'followed_id'
  has_many :followers, through: :passive_relationships, source: :follower
```

## 2.9 dependent オプションを与えて、associationを完成させる

usersテーブルのデータが削除された場合、中間テーブルであるrelationshipsのデータ、  
つまりフォロー・アンフォローの関係を保存しておく必要はないので、  
`dependent: :destroy`のオプションをactiveとpassiveのrelationshipsに追加する。  

これで、完成である。  

```rb
# user.rb
# Associationに係る部分のみ

  has_many :active_relationships, class_name: 'Relationship',
                                  foreign_key: 'follower_id',
                                  dependent: :destroy
  has_many :passive_relationships, class_name: 'Relationship',
                                   foreign_key: 'followed_id',
                                   dependent: :destroy
  has_many :following, through: :active_relationships, source: :followed
  has_many :followers, through: :passive_relationships, source: :follower
```

```rb
# relationship.rb
# Associationに係る部分のみ
class Relationship < ApplicationRecord
  belongs_to :follower, class_name: 'User'
  belongs_to :followed, class_name: 'User'
end
```
