# 20200621 ポートフォリオレビュー会

## レビュー対象

yu-kiさん  
CookingPost  

- https://cooking-post-0610.herokuapp.com/  
- https://github.com/yuki0509/cook_post  

## ポートフォリオ概要

登録ユーザーがレシピを投稿できるアプリ

## アプリの機能

- 管理者がユーザーを登録する。
- 登録されたユーザーはレシピを投稿できる。
- ユーザーはプロフィールを修正できる
- レシピ・プロフィールのいずれも写真投稿可
- レシピの検索機能やCSVの出力機能あり
- kaminariも実装している

## 管理者ログイン

- email messi@test.com  
- PASS king

## 受けた指摘など

### Heroku関係

- HerokuではなぜS3に画像を置かなければいけないのか
  - herokuのリポジトリがdynoという単位で管理されている
  - Freeプランだと一定時間経過後にdynoが再起動される仕様になっている
    - 再起動すると、画像が消える！！！
  - また、そもそもFreeプランだと保存できるのが５MBらしい
  - [【Rails】Herokuで画像を投稿できるようにする方法（ActiveStorage \+ Amazon S3） \- Qiita](https://qiita.com/hmmrjn/items/479c9e9ce82771f1b6d7)

- `credentials.yml.enc`について
  - `master.key`とセットになっている
  - `credentials`にはAWSのアクセスキーなど機密情報を書き込む
  - `credentials`は`master.key`がないと復号できない
  - [Rails5\.2から追加された credentials\.yml\.enc のキホン \- Qiita](https://qiita.com/NaokiIshimura/items/2a179f2ab910992c4d39)
  - Rails6から環境ごとに`credentials`を分けられる

### Baseコントローラ

- Baseコントローラを作って継承させるとよい（Admin関連）
- `before_action :login_required` やprivateメソッドなどを共通化させる
- 共通化できるからといって無闇にアクション自体は書かない方がよい（現場RailsのP415）
  - Baseコントローラは継承を受ける全てのコントローラに影響を及ぼすため

### curren_userに関係する話

- `helper_method :current_user`について
  - ヘルパーメソッドとは
    - 主に view を DRY に書くために使える
    - view にロジックを直書きする必要がなくなり、定義したヘルパーメソッドを呼び出すだけでよくなる
    - [【Rails】独自ヘルパー\(Helper\)メソッドの使い方 \- Qiita](https://qiita.com/shibata0406/items/d59336d9f84cf91b3eb2)
    - [【Rails】Helperを使ってよりDRYなviewを書こう \- Qiita](https://qiita.com/shunsuke227ono/items/21f5968ca7ca0391b583)

- `@curren_user ||= User.find_by(id: session[:user_id])`について
  - `||=`の意味
    - `@current_user`を代入する
    - `@current_user`がなければ、`User.find_by(id: session[:user_id])`を実行する
  - `@curren_user ||= User.find_by(id: session[:user_id])`ではダメなのか
    - SQLコマンドが不要な時は使用しないため
    - SQLコマンドの使用を避けることで、パフォーマンスの向上が期待できる
  - FindとFind_byの使い分け
    - `find`だと該当のデータが見つけられない時は例外処理になる（エラー画面になり、アプリが止まる）
    - `find_by`だと、データが見つけられない時は`nil`を代入する

### Adminと一般ユーザーで使う `application.html.slim`, `CSS`, `JS` を切り分ける

- レイアウトファイルの分け方
  - 一般ユーザーと管理者の画面のレイアウトを分けたい場合、`layout`メソッドを使うとよい
    - 設定がない場合、`application.html.slim`が適用される
    - `layout`メソッドで設定すると、`application.html.slim`の代わりに読み込んでくれる
    - [【Rails】layoutメソッドの使い方をマスターしよう！ \| Pikawaka \- ピカ1わかりやすいプログラミング用語サイト](https://pikawaka.com/rails/layout#application.html.erb%E3%81%8C%E8%AA%AD%E3%81%BF%E8%BE%BC%E3%81%BE%E3%82%8C%E3%82%8B%E7%90%86%E7%94%B1)

- CSSやJSも使い分けることができる
  - アセットパイプラインあたりの理解を深めることが必要な気がする
  - まだ理解できていないが、とりあえずだいそんさんから送られてきたコードを貼る
  - この記事が解読に役立ちそう
    - [Railsで、任意のJavaScriptやCSSだけを読み込む \- Qiita](https://qiita.com/Oakbow/items/2e712e05bb4bbf68faf5)
 
```
  Rails.application.config.assets.precompile += %w( admin.js admin.css )

  config/initializers/assets.rb

  <%= stylesheet_link_tag 'admin', media: 'all' %>
    <%= javascript_pack_tag 'admin' %>
```

### モデル・データベース関係

- 外部キー制約
  - 外部キーとは：テーブル同士の紐づけに用いるカラム
  - 外部キーがあると・・・
    - 存在しない値の外部キーは登録できない
    - 親テーブルに外部キーが登録されている子テーブルのリソースは削除できない  
  - [外部キーの概要と制約を使うことのメリット・デメリット \- Qiita](https://qiita.com/kamillle/items/5ca2db470f199c1bc3ef)

- references型
  - 外部キーを作るには、referencesを使うとよいらしい
    - [Railsの外部キー制約とreference型について \- Qiita](https://qiita.com/ryouzi/items/2682e7e8a86fd2b1ae47)
    - [外部キーをreferences型カラムで保存する \- Qiita](https://qiita.com/sinagaki58/items/7edea51ef00e393834ca)
    - 使わないパターンもあるらしい。。。

- `dependent: :destroy`やその他のオプション
  - yu-kiさんのアプリの場合、外部キーがなかったので紐付かないUserに紐付かないCookデータが作成できてしまう
  - ただ、単純に外部キー制約を付けると、Cookを投稿したUserを削除することができなくなる
  - `dependent: :destroy`を使うと、Userを削除した際に、Cookデータも削除してくれる
  - 今回のケースの場合、Userが削除された場合にCookデータが削除されてしまっては、レシピの数が減ってしまう
    - サービス運営者からすると、`dependent: :destroy`を使うのは避けたい
    - そこで`dependent: :nulify`を使うとよい
      - Userが削除された場合、Cookデータの`user_id`カラムを`nil`に変更して残しておいてくれる
      - [Active Record の関連付け \- Railsガイド](https://railsguides.jp/association_basics.html#has-one%E3%81%AE%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3)

### その他

- ルーティングの`admin`の`root`
  - indexだけでなく、admin階層でのルーティングも設定するとよい
  - 試して分かったが、`users#index`とnamespaceの下に書くと、`admin/users#index`アクションに飛ばしてくれる

```rb: routes.rb
Rails.application.routes.draw do
  get '/login', to: 'sessions#new'
  post '/login', to: 'sessions#create'
  delete '/logout', to: 'sessions#destroy'

  namespace :admin do
    root to: 'users#index'
    resources :users
  end
  
  root to:'cooks#index'
  resources :cooks
end
```

- RubyやRailsのコードの書き方のお作法（Lint)  
  - Rubyのインデントは2つ分のスペースなど
  - [satour/rails\-style\-guide\-jp: 有志による Ruby on Rails 4 のスタイル・ガイドです。](https://github.com/satour/rails-style-guide-jp)

### FactoryBot関係

- FactoryBot.create→createにするやり方
  - [\[RSpec\] FactoryBot 省略の仕方 \- Qiita](https://qiita.com/Yukina_28/items/b560ade3614dce2b55d1)

- Fakerというgemを使うと、自分でランダムにデータを生成してくれる
  - [Fakerを使ってみました！（使い方と実行例） \- Qiita](https://qiita.com/ginokinh/items/3f825326841dad6bd11e)

### レビューを受けて感じた雑感

- みなさんお手柔らかにと言いながらも、だいそんさんが理解度を試すような質問をしていた（笑）
- ただ、おかげで理解がかなり深まったし、前回よりも濃い内容になったように思う
  - ノートを作る作業を通じて、頭の中がかなり整理された
- 実例を通して説明を受けることで、Railsに備わっている技術の意義がよく分かった
  - 例えば、`dependent: :nulify`というものがありますと言われても、重要性が分からないと頭に入らない
