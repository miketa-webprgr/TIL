# Issue11 メール通知の実装

## どんな感じ？

メール通知機能が実装されているので、以下の場合において、メールが送信される。  

- フォローされた時
- いいねされた時
- コメントされた時

つまり、通知のインスタンスが作成された時、メールが送信されている。  

開発環境においてはメールの確認が手間になるため、`letter_opener_web`というgemを使用する。  
確認方法は至ってシンプルであり、`http://localhost:3000/letter_opener`を開けばよい。  

<a href="https://gyazo.com/09006c3070ba5ace05cf563386dd6047"><img src="https://i.gyazo.com/09006c3070ba5ace05cf563386dd6047.png" alt="Image from Gyazo" width="500" border=1/></a><br>  

## 求められている機能実装・実装条件について

- 条件に合致した際にメールを自動送信する
- default_url_optionsの設定値はconfigというgemを使い定数として設定する

## 分からない単語・概念等の一覧

- ActionMailer
- gem `letter_opener_web`
  - [fgrehm/letter\_opener\_web: A web interface for browsing Ruby on Rails sent emails](https://github.com/fgrehm/letter_opener_web)
  - [ryanb/letter\_opener: Preview mail in the browser instead of sending\.](https://github.com/ryanb/letter_opener#rails-setup)
- gem `config`
  - [rubyconfig/config: Easiest way to add multi\-environment yaml settings to Rails, Sinatra, Pandrino and other Ruby projects\.](https://github.com/rubyconfig/config)

## ActionMailerとは

Railsガイドにて説明されている。  
実装方法に学ぶ上で、まず一読するとよい。  

- [Action Mailer の基礎 \- Railsガイド](https://railsguides.jp/action_mailer_basics.html)

ActionMailerは、Railsに備わっている標準の機能である。  
`rails g mailer クラス名`で簡単に必要なファイルを作成できる。  

使い方の概要は把握する上では、以下のQiita記事が分かりやすいように思う。  

- [Action Mailer でメール送信機能をつくる \- Qiita](https://qiita.com/annaaida/items/81d8a3f1b7ae3b52dc2b)

詳細はRailsガイドやQiita記事に任せるとして、ここでは概要を記す。  
ActionMailerを使う際にやることは、以下のとおりである。  

1. `config/environments/development.rb`にてメール送信サーバーの設定（開発環境用）を行う
2. `XXX_mailer.rb`でロジックを書く（Modelに相当）
3. `app/views/contact_mailer`ディレクトリにてメーラービューを作成（Viewに相当）
4. コントローラにてメール送信のロジックを書く

## gem `letter_operner_web` とは

公式GitHubを見ると、丁寧にherokuでサンプルアプリを公開している。  

- [letter_web_openerのサンプルアプリ（heroku）](http://letter-opener-web.herokuapp.com/)

公式GitHubを見ると、設定方法が端的に書いてある。  
まず、これを`routes.rb`に書け！  

```rb
Your::Application.routes.draw do
  mount LetterOpenerWeb::Engine, at: "/letter_opener" if Rails.env.development?
end
```

次に、letter_opener delivery methodの設定をするように書いてある。  
以下を`config/environments/development.rb`に書くとよい。  

```rb
config.action_mailer.delivery_method = :letter_opener_web
config.action_mailer.perform_deliveries = true #=> デフォルトでtrueとなっている
```

なお、`letter_opener`というgemもある。  

`letter_opener`にはインターフェースがなく、メールが送信された場合に端的にメールの送信内容を  
表示させる機能しかない。  

また、ActionMailerにはプレビュー機能がある。  
Railsガイドでも紹介されているが、以下のQiita記事で概要が説明されている。  

- [ActionMailer Preview のススメ \- Qiita](https://qiita.com/port-development/items/ce30e580da064de07b4a)

## gem `config` とは

定数管理をするgem。  

といっても全然ピンとこなかったが、パスワードとかGitHub上に直接晒すとマズいものを違うものに  
置き換えるためのgemだと考えるとよい。  

使い方は非常に簡単であり、以下で概要が紹介されている。  

- [【Rails】「config」gemを使って定数管理をおこなう方法 \| vdeep](http://vdeep.net/rubyonrails-config-gem)

1. `bundle exec rails g config:install`で必要なファイルを生成
2. ymlに「mailerpassword：pass123（実際のパスワード）」のような形で記述
3. パスワードを書くべきところで、Settings.mailerpasswordのような形で記述

## 実装手順

では、コードリーディングを行っていく。  
まず、gem `letter_web_opener`と gem `config`を導入する。  

続いて、以下のコマンドを実行し、configとActionMailerで必要になるファイルを生成する。  

- `bundle exec rails g config:install`
- `bundle exec rails g mailer UserMailer`

ここまで準備が進んだら、具体的なコーディングに移行する。  

## メーラーの設定

ActionMailerの設定方法については、Railsガイドで解説されている。  

- [6 Action Mailerを設定する - Railsガイド](https://railsguides.jp/action_mailer_basics.html#action-mailer%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)
- [3.12 Action Mailerを設定する - Railsガイド](https://railsguides.jp/configuring.html#action-mailer%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)

また、RailsAPIには、さすがドキュメントということもあって詳細に記されている。  
英語だが、Railsガイド等で見当たらないものはこちらを確認するとよさそうだ。  

- [ActionMailer::Base](https://edgeapi.rubyonrails.org/classes/ActionMailer/Base.html)

だいそんさんのコードでは、まず`config`の設定から行われている。  
`development.yml`ファイルを以下のとおりとしている。  

```yml
# config/settings/development.yml
default_url_options:
  host: 'localhost:3000'
```

以上の設定をしたことにより、`Settings.default_url_options.host`と書くと、  
`localhost:3000`が取得できるようになった。  

この定義した定数を使って、メーラーの設定を行うファイルである  
`config/environments/development.rb`を下記のとおり設定する。  

```rb
# development.rb
# 該当部分のみ記載
  config.action_mailer.default_url_options = Settings.default_url_options.to_h
```

アクションメーラーの設定オプションは色々とあるが、奥が深そうなので、  
ここではあまり立ち入らない。書かれているコードだけに集中する。  

`config.action_mailer.default_url_options`だが、  
その意味を調べるため、このQiita記事やRailsAPIの該当箇所を読んだ。  

- [Deviseを使うときに書いていたaction\_mailer\.default\_url\_optionsの意味 \- Qiita](https://qiita.com/minoriinoue/items/393d61b854a34358d102)
- [ActionMailer::Base(Generating URLの箇所を読むこと)](https://edgeapi.rubyonrails.org/classes/ActionMailer/Base.html)

正直Qiita記事はそこまで親切に書いていないので、これを一読しただけだとよく分からなかったが、  
英文の方を頑張って読んだ後にQiita記事を読んでみると、言わんとしていることが分かってきた。  

ざっとまとめると、こんな感じ。  

- メーラーのビューでURLを書く上では、通常のビューとは異なって以下のように書く必要がある
  - `<%= url_for(host: "example.com", controller: "welcome", action: "greeting") %>`
- ルーティングをきちんと設定している場合は、省略できるけど、以下のようにホストを書く必要がある
  - `<%= users_url(host: "example.com") %>`
- こうやってURLをわざわざ書くのはだるいので、`development.rb`でホストを指定するとよい
- hostはアプリを公開するサーバーのURLのこと（予想）

## `letter_opener_webの設定方法`

公式GitHubに書いてあるとおり、まずルーティングの設定を行う。  
`http://localhost:3000/letter_opener`にて送信メールを確認できるようになる。  
（もちろん、ロジックやビューの設定などが終わった後のことだが）  

```rb
# routes.rb

Your::Application.routes.draw do
  mount LetterOpenerWeb::Engine, at: "/letter_opener" if Rails.env.development?
end
```

次に、`config/environments/development.rb`を設定する。  

```rb
# development.rb
# 該当部分のみ記載
  config.action_mailer.delivery_method = :letter_opener_web
```

`config.action_mailer.delivery_method`は、メールの送信手段を設定するオプションである。  
これは、Railsガイドに書いてあるとおりである。  

- [6 Action Mailerを設定する - Railsガイド](https://railsguides.jp/action_mailer_basics.html#action-mailer%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)

stmp以外に、sendmail・file・testなどのオプションがあるらしい。  
sendmailというのはlinuxで使うメール配信ソフトウェアらしく、そちらを活用することもできるらしい。  
fileはメール配信をせずに保存するオプションであり、testは文字どおりテスト用のオプションだと思われる。  

今回は、`gem 'letter_opener_web'`を使うので、`letter_opener_web`とする。  

Gmail等でメールを送信する設定をしたい場合、stmpに設定した後、  
stmpのsettingについてさらにオプションで指定してあげる必要がある。  

Qiita記事などが上がっているので参考にするとよい。  

- [Rails の ActionMailer でメール送信処理 \- Qiita](https://qiita.com/hirotakasasaki/items/ec2ca5c611ed69b5e85e)

## メール送信機能の実装

だいそんさんのコードだと、controllerの方でメール送信処理が実行されるような仕組みになっていた。  
例えば、コメントがDB上で保存された際にメール送信処理が実行されるよう、以下のとおりコーディングされていた。  

```rb
# comments_controller.rb 
def create
  @comment = current_user.comments.build(comment_params)
  UserMailer.with(user_from: current_user, user_to: @comment.post.user, comment: @comment).comment_post.deliver_later if @comment.save
end
```

こうした方式を採用することも検討したが、違う形での実装をするの方が勉強になるかと思ったので、  
あえてコールバックで実行されるような形で実装した。  

なお、複雑なシステムにおいては、コールバックにメール送信機能を実装するのはNGであるらしい。  

- [Rails: メールをActive Recordのコールバックで送信しないこと（翻訳）｜TechRacho](https://techracho.bpsinc.jp/hachi8833/2019_09_12/76762)

Comment、Like、Relaitonshipのモデルにおいて、Notifableというモジュールを導入しているので、  
`send_notification_mail`というメソッドを設けて、その内部に各モデルをオーバーライドするメソッドを設けることとした。  

`create_notifications`と`send_notification_mail`が分かりやすくまとまったというメリットはあるものの、  
こんなダックタイピングするのであれば、両メソッドとコールバックを各モデルに書いた方がよいのではないかも。  

```rb
module Notifiable
  # コールバックなどを使うために必要
  # https://stackoverflow.com/questions/7444522/is-it-possible-to-define-a-before-save-callback-in-a-module
  extend ActiveSupport::Concern

  # URLヘルパーを使うために導入
  include Rails.application.routes.url_helpers

  included do
    has_one :notification, as: :notifiable, dependent: :destroy
    # after_create_commitは、after_commitのエイリアスメソッド
    # after_saveというメソッドもあるが、こちらはDBにsaveする直前に発火するメソッド
    # DBの制約に抵触して保存できない場合も考慮して、after_create_commitとする
    after_create_commit :create_notifications, :send_notification_mail
  end

  def partial_name
    raise NotImplementedError
  end

  def resource_path
    raise NotImplementedError
  end

  private

  def create_notifications
    raise NotImplementedError
  end

  def send_notification_mail
    raise NotImplementedError
  end
end
```

```rb
class Comment < ApplicationRecord
  # 関係するメソッドのみ記載
  # ダックタイピングのため、overrideする
  def send_notification_mail
    UserMailer.with(user_from: user, user_to: post.user, comment: self).comment_post.deliver_later
  end
end
```

```rb
class Like < ApplicationRecord
  # 関係するメソッドのみ記載
  # ダックタイピングのため、overrideする
  def send_notification_mail
    UserMailer.with(user_from: user, user_to: post.user, post: post).like_post.deliver_later
  end
end
```

```rb
class Relationship < ApplicationRecord
  # 関係するメソッドのみ記載
  # ダックタイピングのため、overrideする
  def 
    UserMailer.with(user_from: follower, user_to: followed).follow.deliver_later
  end
end
```

## メイラーの実装

`bundle exec rails g mailer UserMailer`にてメイラーを生成する。  
これにて、通常であれば以下が作成される。  

```text
create  app/mailers/user_mailer.rb
create  app/mailers/application_mailer.rb
invoke  erb
create    app/views/user_mailer
create    app/views/layouts/mailer.text.erb
create    app/views/layouts/mailer.html.erb
invoke  test_unit
create    test/mailers/user_mailer_test.rb
create    test/mailers/previews/user_mailer_preview.rb
```

メイラーは、メールのコントローラ・モデルのようなものである。  
ここでロジックを実装し、メールービューでメール文を出力する。  

まず、`application_mailer.rb`であるが、どのメイラーにも適用させたいような内容を記述する。  
今回は１つのメイラーしか実装しないが、通例にならって、`layout`と`default from`を設定する。  

```rb
class ApplicationMailer < ActionMailer::Base
  default from: 'instaclone@example.com'
  layout 'mailer'
end
```

続いて、`user_mailer.rb`を記述する。  
ここでは、通常のコントローラから受け取った`params`を適宜加工し、ビューファイルに渡す。  

```rb
class UserMailer < ApplicationMailer
  def comment_post
    @user_from = params[:user_from]
    @user_to = params[:user_to]
    @comment = params[:comment]
    mail(to: @user_to.email, subject: "#{@user_from.username}があなたの投稿にいいねしました")
  end

  def like_post
    @user_from = params[:user_from]
    @user_to = params[:user_to]
    @post = params[:post]
    mail(to: @user_to.email, subject: "#{@user_from.username}があなたの投稿にいいねしました")
  end

  def follow
    @user_from = params[:user_from]
    @user_to = params[:user_to]
    mail(to: @user_to.email, subject: "#{@user_from.username}があなたをフォローしました")
  end
end
```

例えば、コントローラで以下が実行された場合について考えてみる。  

- `UserMailer.with(user_from: user, user_to: post.user, comment: self).comment_post.deliver_later`

この場合、UserMailerクラスを探し、ハッシュ形式でパラメータを引き渡す。  
引き渡し先のメソッドは、`comment_post`と書くことで指定できる。  

## メールビューを実装する

メールビューのレイアウトについては、以下のようになっている。  

```rb
# mailer.html.slim
class ApplicationMailer < ActionMailer::Base
  default from: 'instaclone@example.com'
  layout 'mailer'
end
```

```rb
# mailer.text.slim
= yield
```

このレイアウトに差し込むビューファイルを各モデルごとに作成する。  
以下のとおりとした。  

```slim
/ comment_post.html.slim
h2 = "#{@user_to.username}さん"
p = "#{@user_from.username}さんがあなたの投稿にコメントしました。"
= link_to "確認する", post_url(@comment.post, { anchor: "comment-#{@comment.id}" })
```

```slim
/ like_post.html.slim
h2 = "#{@user_to.username}さん"
p = "#{@user_from.username}さんがあなたの投稿にいいねしました。"
= link_to "確認する", post_url(@post)

```

```slim
/ follow.html.slim
h2 = "#{@user_to.username}さん"
p = "#{@user_from.username}さんがあなたをフォローしました。"
= link_to "確認する", user_url(@user_from)
```

## 補足

デバッグ過程においてコールバックが作動し、メールが登録者に送信されるとトラブルにつながる。  
なので、そもそもコールバックを実装すべきでないのかもしれないが、コメントを書き足すことにより注意を促した。  

```rb
# コールバックにより、@commentがDBに保存されるとコメントがあった投稿のユーザーにメールが送信されるので注意すること
```

## 確認方法

フォローなどをしてから、`http://localhost:3000/letter_opener`にアクセスする。  
すると、以下のとおり動作することが確認できる。  

<a href="https://gyazo.com/8ad015c0d12035e54af2a98a729f0cc7"><img src="https://i.gyazo.com/8ad015c0d12035e54af2a98a729f0cc7.gif" alt="Image from Gyazo" width="500" border=1/></a><br>
<a href="https://gyazo.com/3081b74d2374753be156a11064a9b5f7"><img src="https://i.gyazo.com/3081b74d2374753be156a11064a9b5f7.gif" alt="Image from Gyazo" width="500" border=1/></a><br>
