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
config.action_mailer.delivery_method = :letter_opener
config.action_mailer.perform_deliveries = true
```

なお、delivery methodについては、`:letter_opener_web`としてもよい。  

`letter_opener`というgemもあるので、develivery_methodを`:letter_opener`とする場合、  
`letter_opener`のインストールが必要になるかもしれない。  

`letter_opener`にはインターフェースがなく、メールが送信された場合に端的にメールの送信内容を  
表示させる機能しかない。  

## gem `config` とは

定数管理をするgem。  

といっても全然ピンとこなかったが、パスワードとかGitHub上に直接晒すと不味いものを違うものに  
置き換えるためのgemだと考えるとよい。  

使い方は非常に簡単であり、以下で概要が紹介されている。  

- [【Rails】「config」gemを使って定数管理をおこなう方法 \| vdeep](http://vdeep.net/rubyonrails-config-gem)

1. `bundle exec rails g config:install`で必要なファイルを生成
2. ymlに「mailerpassword：pass123（実際のパスワード）」のような形で記述
3. パスワードを書くべきところで、Settings.mailerpasswordのような形で記述

## コードリーディング

では、コードリーディングを行っていく。  



