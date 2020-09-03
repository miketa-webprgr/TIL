# Issue10 通知機能の実装

## どんな感じ？

フォローされた・いいねされた・コメントがあった場合、そのユーザーに通知が届きます。  
具体的には、右上のハートマークのところに表示されます。  

ハートには、新しい通知の数が表示されます。  
ハートをクリックすると、最新の通知が10件まで表示されます。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/12a653110f8ad5ed12b844074b246bbc.gif" alt="Image from Gyazo" width="500"/></a></a><br>  

既読の通知については薄暗い背景、未読の通知については白い背景で表示されます。  
クリックすると、該当のページにアクセスすることができます。  
（なお、各通知をクリックした場合に既読と判定されます。）  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/b3c887680a238e754ef4f23a75ff6b8e.gif" alt="Image from Gyazo" width="500"/></a></a><br>  

また、プロフィール編集画面の方では、下記のとおり表示されます。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/be277b69646cf730527cfbfbc8a0c8e9.png" alt="Image from Gyazo" width="500"/></a></a><br>  

## 求められている機能実装・実装条件について

- 通知機能を実装してください。
  - ヘッダー部分の通知リストには最新の10件しか表示させないでください。
- ポリモーフィック関連を使ってください。
- タイミングと文言は以下の通りとします。（リンク）と書いてある箇所はリンクを付与してください。
  - フォローされたとき
    - xxx（リンク）があなたをフォローしました
    - 通知そのものに対してはxxxへのリンクを張る
  - 自分の投稿にいいねがあったとき
    - xxx（リンク）があなたの投稿（リンク）にいいねしました
    - 通知そのものに対しては投稿へのリンクを張る
  - 自分の投稿にコメントがあったとき
    - xxx（リンク）があなたの投稿（リンク）にコメント（リンク）しました
    - 通知そのものに対してはコメントへのリンクを張る（厳密には投稿ページに遷移し当該コメント部分にページ内ジャンプするイメージ）
- 既読判定も行ってください。通知一覧において、既読のものは薄暗い背景で、未読のものは白い背景で表示しましょう。
- 既読とするタイミングは各通知そのものをクリックした時とします。
- 不自然ではありますが通知の元となったリソースが削除された際には通知自体も削除する仕様とします。

## 分からない単語・概念等の一覧

- [ポリモーフィック関連とは](10_issue_note_polymorphic.md)

## コードリーディング

基本的にはだいそんさんの作成したアプリをコードリーディングし、  
従来どおり、それに倣う形で実装を行うが、以下の点について違う形で実装する。  

- ActivityモデルをNoitificationモデルとする
- enumを使わず、ダックタイピングを利用する

なお、以上については該当の箇所で詳細を説明していく。  

## ポリモーフィック関連を使った実装方針

ポリモーフィック関連については既に以下のとおりノートでまとめているが、  
ただ、今回のケースにおいて、どのような実装を行えばよいのだろうか。  

- [ポリモーフィック関連とは](10_issue_note_polymorphic.md)

記載のとおり、ポリモーフィック関連とは、複数のモデルに対して共通のモデル紐づけるような場合に使用する。  
今回であれば、フォローされた・いいねされた・コメントがあった場合にユーザーに通知を届けるため、  
`Relationship`・`Like`・`Comment`のモデルに対して、共通のモデルを紐づける。  

だいそんさんのコードにおいて、ポリモーフィック関連とするモデル名は`Activity`とされているが、  
今回追加するのは通知機能であるため、`Notification`モデルを追加することとしたい。  

よって、テーブル設計は下記のとおりとする。  

<img src="10_issue_note_polymorphic_tables.png" width=800 border="1"><br>

なお、ポリモーフィック関連のノート内で既に触れているが、このような関連付けだけでなく、  
中間テーブルを使ったテーブル設計などでも実装可能である。詳細についてはこちらを参照すること。  

- [複数のテーブルに対して多対一で紐づくテーブルの設計アプローチ｜スパイスファクトリー株式会社](https://spice-factory.co.jp/development/has-and-belongs-to-many-table/)

また、だいそんさんのコードにおいては、`action_type`というカラムを設け、こちらを`enumerable`としている。  

```rb
enum action_type: { commented_to_own_post: 0, liked_to_own_post: 1, followed_me: 2 }
```

ただし、この`action_type`の情報については、紐付け先を識別するための`subject_type`カラム  
（こちらのコードにおいては`notifiable_type`カラムに当たるもの）と重複するところがある。  
（例えば、`subject_type`がlikesであれば、必然的に`action_type`は`liked_to_own_post: 1`になる）  

そこで、今回の実装においては、`action_type`に当たるようなカラムは設けない形で、自分なりの実装  
を行ってみたいと思う。だいそんさんのコードにおいては、action_typeの値をrenderするパーシャルのファイル名  
とうまく連携させることで、コードを短く書く工夫をしているが、おそらく違う形でも出来るのではないかと思われる。  

また、モデル間の関連付けについては、Railsガイドを参考にしたい。  

- [Active Record の関連付け \- Railsガイド](https://railsguides.jp/association_basics.html#%E3%83%9D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)

あと、misakiさんが各テーブルの関係性をまとめた図（力作！！！）を作っていたので、参考にした。  

- [10 通知機能の実装 by misaki\-kawaguchi · Pull Request \#12 · misaki\-kawaguchi/insta\_clone](https://github.com/misaki-kawaguchi/insta_clone/pull/12)

## マイグレーションファイルの作成 + `db:migrate`

さて、マイグレーションファイルを作成する。  
既に作成した図があるので、そちらに従う形で実装する。  

また、Railsガイドにおいてマイグレーションファイルの参考例が記載されているので、  
そちらをベースにして作成してみた。  

```rb
rails g model Notification notifiable:references{polymorphic} user:references read:boolean

# マイグレーションファイル作成後に以下を追加
readカラムに`null:false, default:false`を追加
```

すると、以下のようなマイグレーションになる。  

```rb
class CreateNotifications < ActiveRecord::Migration[5.2]
  def change
    create_table :notifications do |t|
      t.references :notifiable, polymorphic: true
      t.references :user、foreign_key: true
      t.boolean :read, null: false, default: false

      t.timestamps
    end
  end
end
```

なお、以下のマイグレーションでも同じようにテーブルを作成することができる。  

```rb
class CreateNotifications < ActiveRecord::Migration[5.2]
  def change
    create_table :notifications do |t|
      t.bigint  :notifiable_id
      t.string  :notifiable_type
      t.references :user
      t.boolean :read, null: false, default: false

      t.timestamps
    end
    add_index :notifications, [:notifiable_type, :notifiable_id]
  end
end
```

`rails db:migrate`を実行すると、以下のようなテーブル構造となる。  

```rb
  create_table "notifications", options: "ENGINE=InnoDB DEFAULT CHARSET=utf8", force: :cascade do |t|
    t.string "notifiable_type"
    t.bigint "notifiable_id"
    t.bigint "user_id"
    t.boolean "read", default: false, null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["notifiable_type", "notifiable_id"], name: "index_notifications_on_notifiable_type_and_notifiable_id"
    t.index ["user_id"], name: "index_notifications_on_user_id"
  end
```

## モデルの実装（ポリモーフィックな関連付けを行う）

繰り返しになるが、モデル間の関連付けについては、Railsガイドを参考にしたい。  

- [Active Record の関連付け \- Railsガイド](https://railsguides.jp/association_basics.html#%E3%83%9D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)

すると、以下のようになることが分かる。  

Railsガイドの事例においては`has_many`となっているが、今回の事例においては、  
一つのフォロー登録・いいねアクション・コメント投稿をした際に通知が飛ぶのは１件だけなので、  
`has_many`ではなく、`has_one`を使用した。  

また、「通知の元となったリソースが削除された際には通知自体も削除する仕様とする」とのことだったので、  
`dependent: :destory`を必要な箇所に追記した。  

```rb
class Notification < ApplicationRecord
  belongs_to :notifiable, polymorphic: true
end

class Relationship < ApplicationRecord
  has_one :notification, as: :notifiable, dependent: :destroy
end

class Like < ApplicationRecord
  has_one :notification, as: :notifiable, dependent: :destroy
end

class Comment < ApplicationRecord
  has_one :notification, as: :notifiable, dependent: :destroy
end
```

また、ポリモーフィックな関連付けとは別の話として、NotificationモデルはUserモデルと紐付けを  
する必要がある（通知一覧取得にあたって必要）ので、以下のとおり実装をおこなう。  

```rb
class Notification < ApplicationRecord
  belongs_to :user
end

class User < ApplicationRecord
  has_many :notifications, dependent: :destroy
end
```

## ルーティングの設定

通知一覧を表示させるため、`routes.rb`を以下のとおりとした。  
当初は`notifications_controller.rb`が2つに分かれている意味が分からなかったが、質問して解決した。  

- [Issue 10 ルーティングの設定とパーシャルの重複？について \| TechEssentials](https://tech-essentials.work/questions/131)

```rb
# routes.rb
# 関係部分のみ

  resources :notifications, only: [] do
    # だいそんさんのコードではpatchで新しいアクションを追加する形としていたが、DHH流にやってみることにした
    resource :read, only: %i[create]
  end

  namespace :mypage do
    resource :account, only: %i[edit update]
    # プロフィール編集に関するアクションなので、indexアクションはmypageディレクトリ下にネストさせた
    resources :notifications, only: %i[index]
  end
```

## コールバック関数の設定

コメントされた場合、いいねされた場合、そしてフォローされた場合、通知が作成される。  

つまり、Notificationオブジェクトは、ポリモーフィックに紐づくモデルのオブジェクトであるCommentオブジェクト、  
Likeオブジェクト・Relationshipオブジェクトが生成された際に連動して作成されなければならない。  

方法論としては２つの方法があると思われる。  

- 各コントローラのcreateアクションに記述する
- 各モデルのコールバック関数を活用する

コールバック関数をうまく活用するとコードをDRYに書くことができるが、コントローラ側からコードが追いづらいため、  
条件分岐を含むコールバック関数は多用すべきでないとの情報も目にした。  

- [苦しめられてやっと理解できたRailsコールバックの使い方 \- KitchHike Tech Blog](https://tech.kitchhike.com/entry/2018/06/30/232400)

今回はメリットが多く享受できるケースなので、コールバック関数を採用する。  
（ちなみにコントローラ側で実装しようとしたら、`likes_controller.rb`のロジック実装が面倒になりそうだった）  

各モデルにおいて、以下のとおり記載する。  

```rb
# Comment.rb

# after_create_commitは、after_commitのエイリアスメソッド
# after_saveというメソッドもあるが、こちらはDBにsaveする直前に発火するメソッド
# DBの制約に抵触して保存できない場合も考慮して、after_create_commitとする
after_create_commit :create_notifications

private

# コールバック関数を使いすぎると辛いという記事をいくつか目にしたので、
# コントローラにロジックを書く方法についても検討してみました！
# ただ、ファットコントローラを避けられる + コールバック関数のメソッドがシンプルで
# 許容できるので、コールバック関数を採用することにしました。
def create_notifications
  Notification.create(notifiable: self, user: post)
end
```

```rb
# Like.rb

after_create_commit :create_notifications

private

def create_notifications
  Notification.create(notifiable: self, user: post)
end
```

```rb
# Relationship.rb

after_create_commit :create_notifications

private

def create_notifications
  Notification.create(notifiable: self, user: followed)
end
```

## `shared/_header.html.slim`の作成

まず、トップページのヘッダーに出ているハートの部分について実装していく。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/12a653110f8ad5ed12b844074b246bbc.gif" alt="Image from Gyazo" width="500"/></a></a><br>  

コードは下記のとおりとなっていたが、ハートの部分に変更を加えていく。  

```slim
nav.navbar.navbar-expand-lg.navbar-light.bg-white
  = link_to 'InstaClone', root_path, class: 'navbar-brand'
  button.navbar-toggler aria-controls="navbarTogglerDemo02" aria-expanded="false" aria-label=("Toggle navigation") data-target="#navbarTogglerDemo02" data-toggle="collapse" type="button"
    span.navbar-toggler-icon
  #navbarTogglerDemo02.collapse.navbar-collapse
    = render 'posts/search_form', search_form: @search_form
    ul.navbar-nav.mt-2.mt-lg-0
      / 投稿ボタン
      li.nav-item
        = link_to new_post_path, class: 'nav-link' do
          = icon 'far', 'image', class: 'fa-lg'
      / ハート（通知）ボタン
      li.nav-item
        a.nav-link href="#"
          = icon 'far', 'heart', class: 'fa-lg'
      / ユーザボタン
      li.nav-item
        = link_to user_path(current_user), class: 'nav-link' do
          = icon 'far', 'user', class: 'fa-lg'
```

変更後のコードは下記のとおりです。  
該当箇所のみを記載します。  

```slim
/ ハート（通知）ボタンの部分のみ記載

      / ハート（通知）ボタン
      li.nav-item
        / Bootstapを使って、ドロップダウンのボタンにすることができる
        / https://getbootstrap.com/docs/4.0/components/dropdowns/#single-button-dropdowns
        .dropdown
          a#dropdownMenuButton.nav-link.position-relative href="#" data-toggle="dropdown" aria-expanded="false" aria-haspopup="true"
            = icon 'far', 'heart', class: 'fa-lg'
            / 未読の通知数をアイコンの右上に表示させる
            = render 'shared/unread_badge'
          #header-notifications.dropdown-menu.dropdown-menu-right.m-0.p-0 aria-labelledby="dropdownMenuButton"
            / クリックをすると、未読の通知一覧がドロップダウン形式で表示される
            = render 'shared/header_notifications'
```

## `shared/_unread_badge.html.slim`の作成

まず、未読の通知数をアイコンの右上に表示させるためのパーシャルを作成する。  
未読の通知がある場合にのみ数を表示させるので、以下のようなコードとなる。  

```slim
- if current_user.notifications.unread.count > 0
  span.badge.badge-warning.navbar-badge.position-absolute style='top: 0; right:0;'
    = current_user.notifications.unread.count
```

なお、unreadというメソッドを使っているが、これはNotificationモデルにてenumの設定をしたからである。  
enumの説明はパーフェクトRailsにて簡潔にまとめられていたが、以下も分かりやすそうな印象だった。  

- [【Rails】 enumとは? enumを用いてselectボックスを作ってみた \- Qiita](https://qiita.com/clbcl226/items/3e832603060282ddb4fd)

enumの設定は下記のとおりとなる。  
以下のとおり設定することで、scopeが自動的に定義され、readしたnotificationsやunreadであるnotificationsを取得できる。  

```rb
# Notification.rb

enum read: { unread: false, read: true }
```

また、最新のnotificationsを取得できるを使うので、併せてここで設定しておく。  

```rb
# Notification.rb

scope :recent, ->(count) { order(created_at: :desc).limit(count)}
```

## `shared/header_notifications`の作成

続いて、ハートのボタンをクリックしたときに表示させる通知一覧のパーシャルを作成する。  
コードは下記のとおりである。  

```slim
- if current_user.notifications.present?
  - current_user.notifications.recent(10).each do |notification|
    = render "shared/#{notification.call_appropiate_partial}", notification: notification
  - if current_user.notifications.count > 10
    = link_to 'すべてみる', mypage_notifications_path, class: 'dropdown-item justify-content-center'
- else
  .dropdown-item
    | お知らせはありません
```

ログインしているユーザーに通知がある場合に該当のパーシャルをrenderする。  

- コメントされた場合、`commented_to_own_post.html.slim`をrender
- いいねされた場合、`liked_to_own_post.html.slim`をrender
- フォローされた場合、`followed_me.html.slim`をrender

また、通知数があまりにも多いと困るので、10件以上の通知がある場合については「すべてをみる」をクリックしてもらい、  
マイページの通知閲覧画面で確認してもらうよう設計する。  

なお、`notification.call_appropiate_partial`としているが、`call_appropiate_partial`メソッドは自前で実装する必要がある。  
Notificationモデルにて、独自メソッドを下記のとおり実装する。だいそんさんの場合、テーブル上にaction_typeというカラムを設けていたが、  
ポリモーフィックなテーブルが持つカラムを利用してあげることで、余計なカラムを作る必要がなくなる。  

```rb
# Notification.rb
# ダックタイピングで後ほど綺麗に整える

def call_appropiate_paritial
  puts "commented_to_own_post" if notifiable_type == "Comment"
  puts "liked_to_own_post" if notifiable_type == "Like"
  puts "followed_me" if notifiable_type == "Like"
end
```

## `commented_to_own_post.html.slim`の作成

コメントされた場合の通知のパーシャルを作成する。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://gyazo.com/c7e74b227959ba1d675e515bf608ac31" alt="Image from Gyazo" width="500"/></a></a><br>  

ここでの通知とは、「---があなたの投稿にコメントしました」と表示される１行のことを指しており、  
その他の１行の通知が組み合わさって、通知一覧が作られている。  

アソシエーションについては、子供のオブジェクトから親のオブジェクトを取得する際、  
`@picture.imageable`というような形で書くので、そのことを意識しながら読むとよい。  

```slim
# readというresourceのcreateアクションにアクセスする
# `#{'read' if notification.read?}"`にて、既読の場合に暗めの背景にするCSSを適用する
= link_to notification_read_path(notification), class: "dropdown-item border-bottom #{'read' if notification.read?}", method: :post do
  = image_tag notification.notifiable.user.avatar.url, class: 'rounded-circle mr-1', size: '30x30'
  # これは<object>というタグ。画像、内部の閲覧コンテキスト、プラグインによって扱われるリソースなどのように扱われる外部リソースを表現。
  # おそらく、改行せずに並べていくには都合がよいタグなのだろうと推測。
  object
    = link_to notification.notifiable.user.username, user_path(notification.notifiable.user)
  | があなたの
  object
    = link_to '投稿', post_path(notification.notifiable.post)
  | に
  object
    # anchorを貼ることで、ページの途中にコメントがある場合であっても、該当のコメントがすぐ見れるように設定する
    = link_to 'コメント', post_path(notification.notifiable.post, anchor: "comment-#{notification.notifiable.id}")
  | しました
  .text-right
    # localizeメソッド。日時のフォーマットを修正するのに有効。
    = l notification.created_at, format: :short
```

## `liked_to_own_post.html.slim`の作成

いいねされた場合の通知のパーシャルを作成する。  
基本的な内容については、コメントされた場合のパーシャルと同じである。  

リンク先を適切に指定していく。  

```slim
= link_to notification_read_path(notification), class: "dropdown-item border-bottom #{'read' if notification.read?}", method: :post do
  = image_tag notification.notifiable.user.avatar.url, class: 'rounded-circle mr-1', size: '30x30'
  object
    = link_to notification.notifiable.user.username, user_path(notification.notifiable.user)
  | があなたの
  object
    = link_to '投稿', post_path(notification.notifiable.post)
  | にいいねしました
  .text-right
    = l notification.created_at, format: :short
```

## `followed_me.html.slim`の作成

```slim
= link_to notification_read_path(notification), class: "dropdown-item border-bottom #{'read' if notification.read?}", method: :post do
  = image_tag notification.notifiable.follower.avatar.url, class: 'rounded-circle mr-1', size: '30x30'
  object
    = link_to notification.notifiable.follower.username, user_path(notification.notifiable.follower)
  | があなたをフォローしました
  .text-right
    = l notification.created_at, format: :short
```

## localizeメソッドに関する対応

日時は、日本時間ではなく世界標準時間で表示されてしまう。  
また、必要以上に詳細な情報まで詳細されてしまう。  

そこで、localizeメソッドを使い、フォーマットを整える。  
詳細については、伊藤さんがQiita記事を投稿しているのでそちらを参照するとよい。  

- [【初心者向け・動画付き】Railsで日時をフォーマットするときはstrftimeよりも、lメソッドを使おう \- Qiita](https://qiita.com/jnchito/items/831654253fb8a958ec25)

フォーマットについては、`ja.yml`で指定する。  

```yml
# 該当箇所のみ表示する
ja:
  attributes:
    created_at: '作成日時'
    updated_at: '更新日時'
  time:
    formats:
      default: "%Y年%m月%d日(%a) %H時%M分%S秒 %z"
      long: "%Y/%m/%d %H:%M"
      short: "%m/%d %H:%M"
```


- マイグレート
- polymorphicな関連付けを行う
  - 状況を整理する
  - railsガイドを参考にする
  - [Active Record の関連付け \- Railsガイド](https://railsguides.jp/association_basics.html#%E3%83%9D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)
  - [If use enum for polymorphic\_type, polymorphic associations cannot work well · Issue \#17844 · rails/rails](https://github.com/rails/rails/issues/17844)
  - 
