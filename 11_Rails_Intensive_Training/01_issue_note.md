# Issue01 ログイン機能の実装

## 求められている機能実装について

1. ログイン機能を実装してください。
2. その他初期設定を行ってください
    - generateコマンド時に生成されるファイルを制限する
    - ルーティング、JS、CSS、テストが自動生成されないようにする
    - タイムゾーンの設定
    - gemの導入

## 分からない単語・概念等について

調べ出したら分量が凄まじいことになった。  
別ページにまとめた。以下には目次だけ記す。  

> [環境構築に関することノート](01_issue_note_settings.md)

- Railsでの環境構築全般
- git flowとは
- generateコマンド時に生成されるファイルを制限
- turbolinks
- coffee-script
- Slim
- redisとは
- sorceryとは
- rubocop
- annotate
- MySQL
  - 設定方法
  - bundle installの方法
  - rails new するときのコマンド
- i18n
- database.yml
- migrationファイル
- schema.rbとは
- config/application.rbとは
- yarnとは(Bootstrap Material Designについて)
  - application.html.slimにて、`application.css`と`.js`を読み込む
  - application.scssファイルにて、`@import`形式にて必要なcssを指定する
  - application.jsファイルにて、`//= require`形式読み込むJSファイルを指定する
- デバッグツールについて

## ログイン・ログアウト・ユーザー登録機能の実装

今回実装するのは、ログイン・ログアウト・ユーザー登録機能の実装である。  
`routes.rb`は以下のとおりとなる。  

```rb: routes.rb
     root GET    /                      users#new
    users POST   /users(.:format)       users#create
 new_user GET    /users/new(.:format)   users#new
    login GET    /login(.:format)       user_sessions#new
          POST   /login(.:format)       user_sessions#create
   logout DELETE /logout(.:format)      user_sessions#destroy
```

画面として必要になるのは、`/users/new`と`/login`の２つになる。  
また、コントローラとしては、ユーザー登録に係る`users_controller.rb`と、  
ログインに係る`user_sessions_controller.rb`の２つを用意する。  

### users_controller関係

ロジックはシンプルな形となり、`users#new`アクションの場合、  
Userクラスのインスタンスを生成し、`users#create`アクションでDBへの保存を試みる。  

成功すれば、`users_sessions#new`に飛ばし（つまりログインさせ）、成功のフラッシュメッセージを格納する。  
失敗すれば`render`し、失敗のフラッシュメッセージを表示する。  

ログインにあたっては、sorceryの`auto_login`メソッドを使う。  
`auto_login`だと、引数にメールアドレスやパスワードを使わずログインできる。  
冗長になるが、emailやpasswordを引数とする`login`メソッドでも問題はないかと思う。  

### user_sessions_controller関係

`user_sessions#new`にて、ログイン画面に飛ばす。  
`user_sessions#create`にてログインさせ、`user_sessions#destroy`にてログアウトさせる。  

sorceryの`login`や`logout`を活用し、`users_controller`と同様に実装する。  

### Viewの実装

BMDを活用して書く。  

### Userモデル関係

バリデーションもそうだが、以下のような実装をする必要がある。  
「you can use changes[:crypted_password] instead of crypted_password_changed?の違いが気になる

```rb: User.rb
class User < ApplicationRecord
  authenticates_with_sorcery!

  validates :username, uniqueness: true, presence: true
  validates :email, uniqueness: true, presence: true

  validates :password, length: { minimum: 3 }, if: -> { new_record? || changes[:crypted_password] }
  validates :password, confirmation: true, if: -> { new_record? || changes[:crypted_password] }
  validates :password_confirmation, presence: true, if: -> { new_record? || changes[:crypted_password] }
end
```

### application.html.slimやshared関係

基本的にパクってしまったが、とりあえず構造を整理してみた。  

`header`部分については、ログインしている場合とそうでない場合で読み込む`header`を分けている。  
その下にflashメッセージを読み込み、下に`yield`が来る構成となっている。  

<img src="01_application_html.drawio.svg" width=300 border="1">

## 動作確認方法

1. git clone https://github.com/miketa-webprgr/instagram_clone.git
2. git checkout git checkout -b feature/01_login_logout origin/feature/01_login_logout
3. bundle install
4. yarn install
5. MySQL と Redis を立ち上げる
