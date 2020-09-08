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

また、マイページの方では、下記のとおり表示されます。  

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

## 求められていないけど、勝手にやった実装について

- 以前に勝手に実装したコメントのNGワードフィルター機能のリファクタ
  - コントローラにロジックがあるのが気持ち悪くて耐えられなかったのでモデルに寄せた
- だいそんさんのコードでは、ActivityモデルとしていたがNotificationモデルとした
  - 紐付け先のテーブルを特定するカラム名はsubjectではなく、notifiable_typeとした
  - 紐付け先のレコードのidを特定するカラム名はsubject_idではなく、notifiable_idとした
- だいそんさんのアプリでは、activitiesテーブルにaction_typeというカラムが設けられていた
  - このカラムは不要であり、紐付け先のテーブルを特定するnotifiable_typeを使ったロジックで対応可能
  - Notificationモデルに独自メソッドを設けることで対応した
- だいそんさんのコードだとダックタイピングまでは行われていなかったので、ダックタイピングにチャレンジした

## 分からない単語・概念等の一覧

- ポリモーフィック関連付け
- ダックタイピング
- localizeメソッド（lメソッド）

## ポリモーフィック関連とは

ポリモーフィック関連については以下のとおりノートでまとめてみた。  

- [ポリモーフィック関連とは](10_issue_note_polymorphic.md)

なお、ポリモーフィック関連は便利な手法ではあるが、ポリモーフィック関連を使わずとも、  
同じような機能を実装することは可能であるので、押さえておくとよい。

- [複数のテーブルに対して多対一で紐づくテーブルの設計アプローチ｜スパイスファクトリー株式会社](https://spice-factory.co.jp/development/has-and-belongs-to-many-table/)

## テーブル設計について

今回のケースにおいて、どのような実装を行えばよいのだろうか。  

ポリモーフィック関連は、複数のモデルに対して共通のモデル紐づけるような場合に使用する。  

今回であれば、フォローされた・いいねされた・コメントがあった場合にユーザーに通知を届けるため、  
`Relationship`・`Like`・`Comment`のモデルに対して、共通のモデルを紐づける。  

だいそんさんのコードにおいては、ポリモーフィック関連とするモデル名が`Activity`となっているが、  
今回追加するのは通知機能であるため、`Notification`モデルを追加することとする。  

テーブル設計は下記のとおりとなる。  

<img src="10_issue_note_polymorphic_tables.png" width=800 border="1"><br>

misakiさんが各テーブルの関係性をまとめた図（力作！！！）を作っていたので、参考にした。  

- [10 通知機能の実装 by misaki\-kawaguchi · Pull Request \#12 · misaki\-kawaguchi/insta\_clone](https://github.com/misaki-kawaguchi/insta_clone/pull/12)

## notificationsテーブルの`action_type`カラムは作らない

だいそんさんのコードにおいては、`action_type`というカラムを設け、こちらを`enumerable`としている。  

```rb
enum action_type: { commented_to_own_post: 0, liked_to_own_post: 1, followed_me: 2 }
```

ただし、この`action_type`の情報については、紐付け先を識別するための`subject_type`カラム  
（こちらのコードにおいては`notifiable_type`カラムに当たるもの）と重複するところがある。  
（例えば、`subject_type`がlikesであれば、必然的に`action_type`は`liked_to_own_post: 1`になる）  

そこで、今回の実装においては、`action_type`に当たるようなカラムは設けない形で、自分なりの実装  
を行ってみたいと思う。だいそんさんのコードにおいては、`action_type`の値をrenderするパーシャルのファイル名  
とうまく連携させることで、コードを短く書く工夫をしているが、おそらく違う形でも出来るのではないかと思われる。  
（結果として、Notificationモデルに独自メソッドを実装する形で解決した）  

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

## コールバックの設定

コメントされた場合、いいねされた場合、そしてフォローされた場合、通知が作成される。  

つまり、Notificationオブジェクトは、ポリモーフィックに紐づくモデルのオブジェクトであるCommentオブジェクト、  
Likeオブジェクト・Relationshipオブジェクトが生成された際に連動して作成されなければならない。  

方法論としては２つの方法があると思われる。  

- 各コントローラのcreateアクションに記述する
- 各モデルのコールバックを活用する

コールバックをうまく活用するとコードをDRYに書くことができるが、コントローラ側からコードが追いづらいため、  
条件分岐を含むコールバックは多用すべきでないとの情報も目にした。  

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

def call_appropiate_partial
  puts "commented_to_own_post" if notifiable_type == "Comment"
  puts "liked_to_own_post" if notifiable_type == "Like"
  puts "followed_me" if notifiable_type == "Like"
end
```

## `commented_to_own_post.html.slim`の作成

コメントされた場合の通知のパーシャルを作成する。  

<a href="https://gyazo.com/c7e74b227959ba1d675e515bf608ac31"><img src="https://i.gyazo.com/c7e74b227959ba1d675e515bf608ac31.png" alt="Image from Gyazo" width="500"/></a><br>  

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

## reads_controller.rbを作成する

ヘッダーに表示されるハートのボタンを押すと、通知が表示されるよう一通りのパーシャルを作り終えた。  
ただ、この時点ではクリックをしても、link先として指定した`notification_read_path(notification)`がない。  

そこで、受け入れ先となる`reads_controller.rb`を作成し、そちらのcreateアクションにおいて、  
以下のロジックを実装していく。  

1. コールバック関数にて作成されたNotificationオブジェクトのreadというresouceを作成する
2. Notificationオブジェクトの紐付き先に合わせて、適切なパスへとredirectさせる

```rb
# reads_controller.rb

class ReadsController < ApplicationController
  # 念のため、ログインしていないユーザーが既読にしないよう制限しておく
  before_action :require_login, only: %i[create]

  def create
    @notification = Notification.find(params[:notification_id])
    # enum設定をしたので、read!メソッドでreadカラムをreadに書き換えることができる
    @notification.read! if @notification.unread?
    # appropiate_pathメソッドはモデルで実装する
    redirect_to @notification.appropiate_path
  end
end
```

appropiate_pathメソッドであるが、以下のとおりとした。  

なお、シンボルの方が処理速度が早いらしいので、そこはだいそんさんに倣って、  
ダックタイピングをする際に合わせてリファクタリングする。  

```rb
  # Notification.rbに追記した部分のみ記載

  # URLヘルパーを使うために導入
  include Rails.application.routes.url_helpers

  # 紐付き先のモデルのshowアクションにredirectさせる場合、polymorphic_pathが使える
  # 今回の場合、コメントした・いいねした投稿のshowアクション、フォローしてくれたユーザーのshowアクションに
  # redirectする必要があるので、モデルで独自メソッドを実装する
  # こちらも、ダックタイピングを使って後ほど綺麗に整える
  def appropiate_path
    case self.notifiable_type
    when "Comment"
      post_path(self.notifiable.post, anchor: "comment-#{notifiable.id}")
    when "Like"
      post_path(self.notifiable.post)
    when "Relationship"
      user_path(self.notifiable.follower)
    end
  end
```

## SCSSを適用する

既読となった場合、CSSを適用させて背景色を変更する。  
また、体裁を整えるため、その他の設定も行う。  

マイページでも使いまわせるよう、`application.scss`に書くのではなく、  
`header.scss`にコードを書き、そのscssをimportする形で対応する。  

```scss
// application.scss

@import 'header';
```

```scss
// _header.scss

#header-notifications {
  width: 400px;
  .dropdown-item {
    max-width: initial;
    font-size: 12px;
  }

  .read {
    background: #f1f1f1;
  }
}
```

## マイページ配下に通知一覧ページを作成する

これまで、ヘッダーに関する部分を作成してきたが、続いてマイページで表示する通知一覧を作成する。  
まず、マイページのメニュー画面にて、通知一覧へのリンクを作成する。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/be277b69646cf730527cfbfbc8a0c8e9.png" alt="Image from Gyazo" width="500"/></a></a><br>  

```slim
/ app/views/mypage/shared/_sidebar.html.slim

nav
  ul.list-unstyled
    li
      = link_to 'プロフィール編集', edit_mypage_account_path
      hr

    / 以下を追加
    li
      = link_to '通知一覧', mypage_notifications_path
      hr
```

続いて、通知一覧を表示させるため、モデルからビューにデータを渡すためのコントローラを作成する。　　
なお、マイページに関するものであるため、Mypage::BaseControllerを継承させる。  

```rb
class Mypage::NotificationsController < Mypage::BaseController
  before_action :require_login, only: %i[index]

  def index
    # kaminariのメソッドを使い、10件以上の場合はページネーションさせる
    @notifications = current_user.notifications.order(created_at: :desc).page(params[:page]).per(10)
  end
end
```

`@notifications`を受け取るビューファイルを作成する。  

```slim
/ mypage/notifications/index.html.slim

- if @notifications.present?
  - @notifications.each do |notification|
    = render "shared/#{notification.call_appropiate_partial}", notification: notification
  = paginate @notifications

- else
  .text-center
    | お知らせはありません
```

CSSも忘れてず適用させる。  

```scss
// mypage.scss

@import 'header';

.read {
  background: #f1f1f1;
}
```

## ダックタイピングでNotificationモデルのメソッドをリファクタする

ポリモーフィック関連付けを使った場合、caseを使うのはアンチパターンらしい。  
ということで、以下のようなダックタイピングを使ってみた。  

モジュールを使って型を明示するのがベストプラクティスとなりつつあるらしいので、  
ダックタイピングを終えた後に、その作業も行う。  

ダックタイピングの事例をいくつか載せていたので、この作業では以下の記事が一番  
参考になった。ゆっくりとコードを追っていけば、チェリー本のクラスやメソッドなど  
の知識程度があれば、その仕組みを理解することができる。　 

- [【リファクタリング】こんなコードはイヤなので一刻も早く綺麗にしたい \- Qiita](https://qiita.com/yu-croco/items/36a891af2e7e7ab564b4)

さて、リファクタ前のNotificationモデルのメソッドであるが、下記のとおりである。  

```rb
# Notification.rbの独自メソッド

  # URLヘルパーを使うために導入
  include Rails.application.routes.url_helpers

  def call_appropiate_partial
    case self.notifiable_type
    when "Comment"
      "commented_to_own_post"
    when "Like"
      "liked_to_own_post"
    when "Relationship"
      "followed_me"
    end
  end

  def appropiate_path
    case self.notifiable_type
    when "Comment"
      post_path(self.notifiable.post, anchor: "comment-#{notifiable.id}")
    when "Like"
      post_path(self.notifiable.post)
    when "Relationship"
      user_path(self.notifiable.follower)
    end
  end
```

以上を下記のとおり、リファクタする。  
`Notification.rb`の独自メソッドについて、紐付き元である`Comment.rb`などの各モデルのメソッドで上書きする。  

```rb
# Notification.rbの独自メソッド（リファクタ後）

  # 適切なパーシャルを取得するメソッド（ダックタイピングを活用）
  def call_appropiate_partial
    notifiable.partial_name
  end

  # 適切なパスを取得するメソッド（ダックタイピングを活用）
  def appropiate_path
    notifiable.resource_path
  end
```

```rb
# Comment.rb

  # URLヘルパーを使うために導入
  include Rails.application.routes.url_helpers

  def partial_name
    'commented_to_own_post'
  end

  def resource_path
    post_path(post, anchor: "comment-#{id}")
  end
```

```rb
# Like.rb

  # URLヘルパーを使うために導入
  include Rails.application.routes.url_helpers

  def partial_name
    'liked_to_own_post'
  end

  def resource_path
    post_path(post)
  end
```

```rb
# Relationship.rb

  # URLヘルパーを使うために導入
  include Rails.application.routes.url_helpers

  def partial_name
    'followed_me'
  end

  def resource_path
    user_path(follower)
  end
```

## ダックタイピングの型を明示し、共通化できるものを`concerns/notifiable.rb`内のモジュールに記す

ダックタイピングを行う場合、上書きすべきメソッドが上書きされない場合にエラーとなるよう、  
モジュールを活用して、型を明示してあげるのが一般できらしい。  

正直、モジュール・ミックスイン・ダックタイピング・AcitiveSupport::Concern、  
いずれも理解がまだ不十分なところが多いが、とりあえず見よう見まねで実践してみた。  

具体的には、以下のようなコードとして、モジュールとして各クラスに取り込んでもらいたい  
ものを`notifiable.rb`として作成し、`models/concerns`ディレクトリに置いた。  

なお、この機会を利用して、以下を共通化してみた。  

- URLヘルパー
- アソシエーション
- コールバックと`create_notifications`メソッド

```rb
# concerns/notifiable.rbを作成

# インターフェースを明確化するために、moduleで固める
module Notifiable
  # コールバックなどを使うために必要
  # https://stackoverflow.com/questions/7444522/is-it-possible-to-define-a-before-save-callback-in-a-module
  extend ActiveSupport::Concern

  # URLヘルパーを使うために導入
  include Rails.application.routes.url_helpers

  included do
    has_many :notifications, as: :notifiable
    after_create_commit :create_notifications
  end

  def partial_name
    raise NotImplementedError
  end

  def resource_path
    raise NotImplementedError
  end

  def notification_user
    raise NotImplementedError
  end

  private

  # 通知を作成するメソッド（ダックタイピングを活用）
  def create_notifications
    Notification.create(notifiable: self, user: notification_user)
  end
end
```

## NGワード制約機能のリファクタ

Rails特訓コースで求められている実装ではないが、遊びで`gem 'swearjar'`を導入して、  
NGワードを含む投稿ができないような制約を実装していた。  

ただ、実装当時は勉強不足であり、無理やりコントローラやビューファイルにロジックを書くような形で実装していたので、  
カスタムバリデータを導入し、コメントだけでなく、投稿の本文にもNGワードを含めることができないような形で実装した。  

具体的には、validatorsディレクトリを切り、`ng_words_validator.rb`というファイルを作成し、  
そこで独自のロジックを書き込み、必要なモデルクラスで読み込むような形で実装した。  


```rb
class NgWordsValidator < ActiveModel::Validator
  def validate(record)
    # NGワードをここで読み込む
    ng_words = Swearjar.new('config/locales/my_swears.yml')
    # NGワードを含んでいる場合はerrorを返す
    record.errors.add(:body, 'にはNGワードが含まれています。綺麗な言葉を使いましょう。') if ng_words.profane?(record.body)
  end
end
```

```rb
# comment.rbとpost.rbに以下を追加
validates_with NgWordsValidator
```

```rb
# application.rbに追記

# カスタムバリデータを使用する
class Application < Rails::Application
  config.autoload_paths += Dir["#{config.root}/app/validators"]
end
```

なお、以下にcommitログが掲載する。  

- [\[update\] カスタムバリデータを使って、Postの本文にもNGワード制約を追加した](https://github.com/miketa-webprgr/instagram_clone/pull/10/commits/b337049538f43971a5ca1c885ce9ed2ed0647619)

また、カスタムメソッドという単純にクラス内に独自バリデーションを追加する方法もあるが、そちらも実践してみた。  
そちらの`commit log`は下記のとおり。  

- [\[update\] Commentのバリデーションにカスタムメソッドを導入](https://github.com/miketa-webprgr/instagram_clone/pull/10/commits/bb015471184402e8e29a98ce4fa8d268157a756f)

カスタムバリデーションについては、以下の記事が参考になるので、ざっと眺めてみるとよい。  
そこまで難しくないので、実践してみると導入できるかと思う。  

- [Active Record バリデーション \- Railsガイド](https://railsguides.jp/active_record_validations.html#%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E5%AE%9F%E8%A1%8C%E3%81%99%E3%82%8B)
- [カスタムバリデーションの作成方法まとめ \| Qrunch（クランチ）](https://qrunch.net/@hikey/entries/HZv7DnkeDThTjl5b)
- [【Rails】Railsのバリデーションの使い方をマスターしよう！ \| Pikawaka \- ピカ1わかりやすいプログラミング用語サイト](https://pikawaka.com/rails/validation#%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E4%BD%9C%E3%82%8D%E3%81%86)
- [Railsバリデーションまとめ \- Qiita](https://qiita.com/h1kita/items/772b81a1cc066e67930e#%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3)
- [【Rails】カスタムバリデータの使い方 \- TASK NOTES](https://www.task-notes.com/entry/20160918/1474210236)

ちょっと恥ずかしいけど、導入するとこんな感じでバリデーションがかかる。  

<img src="https://i.gyazo.com/5b34e00855e9448623141c012c834990.png" alt="Image from Gyazo" width="500"/><br>  

<img src="https://i.gyazo.com/f077645a8755eb4416b5005ddf7c9877.png" alt="Image from Gyazo" width="500"/><br>  
