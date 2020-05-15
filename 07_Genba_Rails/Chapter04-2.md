## 現場Rails Chapter04-5 ~ Chapter04-10

## 「現実の複雑さに対応する」

---

<BR><BR>

### Chapter04-5 「ログイン機能を実装する」

---

認証方法にもいくつもの方法がある。  
また、自力で実装するだけでなく、deviseなどのgemを使う方法もある。  

* メールアドレスとパスワードの組み合わせで認証を行う
* FacebookやTwitterなどの外部サービスにログインして、認証を行う

<br>

#### セッションとは

---

HTTPはステートレス（誰からのリクエストなのか記憶しない）ので、サーバー側にセッション  
という仕組みを用意して、一連のリクエストの間で「状態」を共有できるようにしている。  

Railsでは、コントローラからsessionメソッドを呼び出し、セッションにアクセスできるようにしている。  

``` 
# sessionへの格納
session[:user_id] = @user.id
```

``` 
# sessionの呼び出し
@user_id = session[:user_id]
```

<br>

#### Cookieとは

---

Sessionに似た仕組みとして、Cookieがある。  
SessionはAPサーバー側で用意した仕組みなのに対し、CookieはブラウザとWebサーバーの間で使われる汎用的な仕組み。  

[ブログ: Cookieとセッションについてわかりやすく解説します！](https://umaroidblog.com/webtechnology1)  

要は、cookieは見られてしまう可能性があるので危険。  
cookieを通してやりとりをするのは、単純な受付番号札だけにする。  
それはセッション（一連のやり取り）が終わったら破棄。  
パスワードなどの重要な情報はcookieでやり取りしない。ブラウザ側で保存しない。  
これでセッション中にcookieが盗まれない限り、安全性は担保される。  

という仕組みらしい。  

細かく紐解いた説明は以下のとおり。  
[Cookieとセッションをちゃんと理解する](https://qiita.com/hththt/items/07136ad74127999df271)  

<br>

#### Userモデルを作る

---

ユーザーのログイン情報を管理するUserモデルを作成する。  
Userのモデルは、下記のとおりとする。  

|意　味|属性名|データ型|
|-----|-----|-------|
|名　前|name |string|
|メアド|email|string|
|パ　ス|password_digest|string|

なお、password_digestとは、ハッシュ化を行ったパスワードのことを指す。  
ハッシュ化されたものであれば、仮に漏洩したとしてもパスワード自体に復号することはできない。  

さて、generateコマンドを使って、Userモデルを作成する。  
これでマイグレーションファイルが生成される。

``` rb
bin/rails g model user name:string email:string password_digest::string
```

ここで復習だが、マイグレーションは２ステップ  

1. スキーマを変更するマイグレーションファイルの作成
2. マイグレーションファイルを「rails db:migrate」で走らせ、データベースに適応

マイグレートする前に、マイグレーションファイルにnullを受け入れない属性を設定する。  
他、同ファイルに必要な事前の設定を行っていく。  

``` rb
#db/migrate/~~~~_create_users.rb
# マイグレーションは２ステップ  

1. スキーマを変更するマイグレーションファイルの作成
2. マイグレーションファイルを「rails db:migrate」で走らせ、データベースに適応

```

設定後、マイグレートする。  

<br>

#### パスワードを受け付けてdigest（ハッシュ値）を保存する

---

bcryptというgemを導入する。  
導入後、has_secure_passwordをuserモデルに記載する。  

導入方法については、Qiita等でも紹介されている。  
[Qiita: Rails gem bcryptの導入](https://qiita.com/bomber0522/items/becb7bd4452561d76087)  

これにより、一時的にパスワードとパスワードをハッシュ化したものを格納する属性が生成される。  

Railsコンソールで試すと、ダイジェスト化されたパスワードが確認できる。  
さて、ここで試そうとするがトラブル発生。  

``` 
You don't have bcrypt installed in your application. Please add it to your Gemfile and run bundle install.
```

Gemfileに書いたのになんで。。。  
ググると以下の記事を発見。  

[Qiita: rails consoleでコマンド実行時のbcryptに関するエラーとその対処法](https://qiita.com/ricoirico/items/f3600abea6eb221a62df)  
とりあえず、試す。  

うまくいかない 笑  
うまくいかないのにも慣れてきた。。。  

仕方がないが、 Webrickを再起動したら〜といった記述があったので、  
関係するか分からないが、そもそもターミナル自体を一旦再起動してみる。  
（railsサーバー自体は再起動してみたがうまくいかず）  

不思議と問題が解決。  

``` 
# userインスタンスにname, email, password, password_confimationを代入 
user = User.new(name: 'ユーザー', email: 'sample@example.com', password: 'password', password_confirmation: 'password')

user.save

# パスワードがハッシュ化される！
user.password_digest
=> $2a$12$lHl.dAurBkK48uX/7jVByO5KstZlzaOzhzOzCCljScE80wAEMVn1u"
```

<br>

#### ユーザー管理機能一式を追加する

---

Userモデルを作成したので、どのように利用者登録をさせるのか検討する。  
一般的には、以下の２通りの方法がある。  

1. 未登録者が自らサインアップする
2. 管理者がユーザーを登録する

今回は、２の方法で行っていく。  
具体的には、ユーザー管理機能を追加し、その画面からユーザーの登録・編集等が行えるようにする。  

ユーザー管理機能は、/admin/usersで始まるURLで提供する。  
adminというフラグがtrueのユーザーだけ利用できるようにする。  

<br>

#### Userモデルにadminフラグを追加

---

まず、ユーザーが管理者かどうかを表すフラグを追加する。
・・・と言われてもピンとこない。

とりあえず、指示のとおりマイグレーションファイルを作成する。  
また、マイグレーションファイルのカラムを設定する。

``` 
bin/rails g migration add_admin_to_users
```

``` rb
class AddAdminToUsers < ActiveRecord::Migration[5.2]
  def change
    add_column :users, :admin, :boolean, default: false, null: false
  end
end
```

ここでbooleanについて調べる。
ブール型ともいうらしい。

> boolean型とは、true(トゥルー) またfalse(フォールス)のどちらかのデータが<br>
> 必ず入ることが決まっているデータ型です。
>
> [Samurai Blog: 【Java入門】booleanとBooleanの使い方(初期値も解説)](https://www.sejuku.net/blog/41241)

なお、こちらに設定についての解説があった。  
[Qiita: Boolean型のカラムを追加するときは必ずデフォルト値を設定しよう](https://qiita.com/jnchito/items/a342b64cd998e5c4ef3d)

細かいところはスルーしてしまったが、

* boolean型の属性だと、trueかfalseのどちらかの値しか受け付けない
* デフォルト値を設定でき、今回はfalseとした
* NULLとなることを禁止した

ということが理解できたのでよしとする。  

そして、仰せのとおりマイグレートする。

<br>

#### ユーザー管理のためのコントローラの実装

---

「管理系」の機能として「ユーザー管理」を行うので、「Admin:: UsersController」と名前をつける。  
・・・

Rubyのことがほぼ全く分からないので、そもそも「::」に戸惑う。  
ActiveRecordとかで見るやつというイメージしかない。  

また、ググって調べてみる。

[Qiita: rubyのクラスやらモジュールがモヤっとしている人へ](https://qiita.com/gogotanaka/items/c931360b3f6248959f89)  
[Qiita: メソッドとクラスメソッドとインスタンスメソッドが曖昧だった](https://qiita.com/right1121/items/c74d350bab32113d4f3d)  

なるほど。  
Adminというクラス（モジュール？）があって、その中のメソッドとしてUserControllerがある。  
そして、クラスメソッドだから、Admin. UsersControllerですぐ使用できる。  
そんな感じだろうか。  

そして、おそらくRailsの規約により、Adminというクラス（モジュール？）を作れば、  
admin/users_controller.rbのコードがそのメソッドとして対応する。  

・・・
自分で書いておいて訳が分からなくなってきたが、先に進めることとする。  
Rubyの勉強も必要かもしれない。  

Admin:: UsersControllerには一般的なCRUD機能を持たせることにする。  
以下のようなアクションを持たせる。

こちらについてはChapter03−3で学習したとおり、同じような形で進めていく。

| URL                       | アクション | HTTPメソッド | 動作                |
|:--------------------------|:--------|:----------|:------------------ |
| /admin/users          | index   | GET       | ユーザ一覧画面を表示   |
| /admin/users          | create  | POST      | ユーザの登録処理      |
| /admin/users/new      | new     | GET       | ユーザの登録画面を表示 |
| /admin/users/:id/edit | edit    | GET       | ユーザの編集画面を表示 |
| /admin/users/:id      | show    | GET       | ユーザの詳細画面を表示 |
| /admin/users/:id      | update  | PATCH     | ユーザの更新処理      |
| /admin/users/:id      | update  | PUT       | ユーザの更新処理      |
| /admin/users/:id      | destroy | DELETE    | ユーザの削除処理      |

まず、Admin:: UsersControllerの作成である。

``` 
bin/rails g controller Admin::Users new edit show index
```

<br>

#### routes.rbの設定

---

そして、routes.rbの設定を行う。  
namespce以降からrootまでの記述が追加されたことを確認する。  

``` rb
Rails.application.routes.draw do

  # rails generate をしたことに追加されたコード
  namespace :admin do
    get 'users/new'
    get 'users/edit'
    get 'users/show'
    get 'users/index'
  end

  # これまでのコード
  root to: 'Users#index'
  resources :Users
end
```

namespaceについて調べてみる。  
[Qiita: Railsのroutingにおけるscope / namespace / module の違い](https://qiita.com/ryosuketter/items/9240d8c2561b5989f049)  

とりあえず、URLもファイル構成も連動して指定のパスにしたいしたい場合、  
namespaceを使うとよいことが分かった。  

さて、routes.rbを以下のとおり書き換える。

``` rb
# 該当部分のみ抜粋して掲載

  # rails generate をしたことに追加されたコード
  namespace :admin do
    resources :users
  end
```

ここでも、Chapter03でCRUD機能を実装したときと同様にresourcesを使う。  
resourcesについては以下を改めて参照。  

[Qiita：Rails resourcesメソッドとresourceメソッド](https://qiita.com/Tamitchao/items/6f45aa6daf1412b78d10)  

<br>

#### コントローラの設定

---

まず、アクションの流れを確認する。

1. admin/users/newにて、ユーザーの登録画面（users/new.html.slim）を表示
2. このとき、newアクションにてuserインスタンスにUserオブジェクトが代入
3. 登録ボタンを押すと、createアクションに飛ぶ
4. createアクションにて、validationに抵触しなければ登録し、showアクションへ
5. validationに抵触すれば、エラーメッセージを表示し、登録画面を再表示

さて、コントローラの設定を行う。

ここでは、newアクション、createアクションを作成する。  
なお、showアクションへの「_url」は、rails routes コマンドにて確認できる。

``` rb
# admin/users_controller.rb

class Admin::UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to admin_user_url(@user), notice: "タスク「#{@user.name}」を登録しました。"
    else
      render :new
    end
  end

  private

  def user_params
    params.require(:user).permit(:name, :email, :admin, :password, :password_confirmation)
  end
end
```

<br>

#### ユーザー登録画面の作成

---

次に、ユーザーの登録画面（users/new.html.slim）を作成する。  
Usersディレクトリ内のnew.html.slimを活用する。  

``` 

h1 ユーザーの新規登録
# users/new.html.slim

= form_with model: [:admin, @user], local: true do |f|
  .form-group
    = f.label :name, '名前'
    = f.text_field :name, class: 'form-control'
  .form-group
    = f.label :email, 'メールアドレス'
    = f.text_field :email, class: 'form-control'
  .form-check
    = f.label :admin, class: 'form-check-label' do
      = f.check_box :admin, class: 'form-check-input'
      # "|"は、テキストを書くときに使用。<p>と同じ効果。
      | 管理者権限
  .form-group
    = f.label :password, 'パスワード'
    = f.password_field :password, class: 'form-control'
  .form-group
    = f.label :password_confirmation, 'パスワード（確認）'
    = f.password_field :password_confirmation, class: 'form-control'
  = f.submit '登録する', class: 'btn btn-primary'
```

まず、form_with modelの書き方であるが、改めて以下の記事を確認。  
[Qiita: 【Rails】form_with/form_forについて【入門】](https://qiita.com/snskOgata/items/44d32a06045e6a52d11c)  

今回は、form_withに複数モデルを渡す方法を知る必要があるので、該当部分を確認。

ここでは、１つ目がインスタンス変数でないため「/admin」が必ずurlに付与され、  
２つ目がインスタント変数userなので、中身の有無で振り分け先のアクションを判別する。  

userがない場合はcreateアクションへと飛び、ある場合はupdateアクションへと飛ぶ。  

なお、作成画面は下記のとおりとなった。  

<br>

<a href="https://gyazo.com/d6e65151b30bf7469fbc56d86b8c73d0"><img src="https://i.gyazo.com/d6e65151b30bf7469fbc56d86b8c73d0.png" alt="Image from Gyazo" width="550" border=1/></a>

<br>

#### その他のアクションや画面作成

---

さて、その他のアクションやビュー画面を作成する。  
ついては、アクションの流れを確認する。  

<a href="https://gyazo.com/8a2f4803ccbe6cdfd183251e66d9dbff"><img src="https://i.gyazo.com/8a2f4803ccbe6cdfd183251e66d9dbff.png" alt="Image from Gyazo" width="550" border=1/></a>

図にしてみたが、分かりやすくなったのだろうか。  

<br>

#### indexアクションとビューの作成

---

まず、indexアクションとビューの作成から行う。

``` rb
# admin/users_controller.rb
# indexアクション部分のみ
# 登録ユーザー一覧を掲載するため、DBからデータを全て取得

def index
  @users= User.all
end
```

``` 
# users/index.html.slim

h1 ユーザー一覧

= link_to '新規登録', new_admin_user_path, class: 'btn btn-primary'

# 一覧では以下の属性を表示させる
# name, email, admin, created_at, updated_at
# adminは、管理者権限の有無を示す

.mb-3
table.table.table-hover
  thead.thead-default
    tr
      th= User.human_attribute_name(:name)
      th= User.human_attribute_name(:email)
      th= User.human_attribute_name(:admin)
      th= User.human_attribute_name(:created_at)
      th= User.human_attribute_name(:updated_at)

    tbody

      - @users.each do |user|

        tr
          # 該当のadminについてのデータをparamsに渡す
          td= link_to user.name,[:admin, user]
          td= user.email
          td= user.admin? ? 'あり' : 'なし'
          td= user.created_at
          td= user.updated_at
          td
            = link_to '編集', edit_admin_user_path(user), class: 'btn btn-primary mr-3'
            = link_to '削除', [:admin, user], method: :delete, data: { confirm: "ユーザー「#{user.name}」を削除します。よろしいですか？"}, class: 'btn btn-danger'

```

なお、rubyの勉強不足で分かっていなかったが、  
「user.admin? ? 'あり' : 'なし'」の意味は、以下のとおり。  

1. user.admin が true か確認
2. true であれば「あり」と表示
3. falseであれば「なし」と表示

無事、実装完了。

<br>

<a href="https://gyazo.com/8faf56c342da4d29b3bacc5719c8c785"><img src="https://i.gyazo.com/8faf56c342da4d29b3bacc5719c8c785.png" alt="Image from Gyazo" width="600" border=1/></a>

<br>

#### showアクションとビューの作成

---

次に、showアクションとビューの作成を行う。  

``` rb
# admin/users_controller.rb
# showアクション部分のみ
# 登録ユーザー一覧を掲載するため、DBからデータを全て取得

def show
  @user= User.find(params[:id])
end
```

``` 
# users/show.html.slim

h1 ユーザーの詳細

.nav.justify-content-end
  = link_to '一覧', admin_users_path, class: 'nav-link'

table.table.table-hover
  tbody
  tr
    th= User.human_attribute_name(:id)
    td= @user.id
  tr
    th= User.human_attribute_name(:name)
    td= @user.name
  tr
    th= User.human_attribute_name(:email)
    td= @user.email
  tr
    th= User.human_attribute_name(:admin)
    td= @user.admin? ? 'あり' : 'なし'
  tr
    th= User.human_attribute_name(:created_at)
    td= @user.created_at
  tr
    th= User.human_attribute_name(:updated_at)
    td= @user.updated_at

= link_to '編集', edit_admin_user(@user.id), class: 'btn btn-primary mr-3'
= link_to '削除', [:admin, @user], method: :delete, data: { confirm: "タスク「#{@user.name}」を削除します。よろしいですか？"}, class: 'btn btn-danger'
```

慣れてきたのか、それなりにすんなりと実装できた。  
ただ、白紙の状態からやれと言われると相当つらそうだ。。。  

<br>

<a href="https://gyazo.com/1c78ddbd53fd817f1403b1957586f013"><img src="https://i.gyazo.com/1c78ddbd53fd817f1403b1957586f013.png" alt="Image from Gyazo" width="550" border=1/></a>

<br>

#### editアクションとビューの作成

---

続いて、editアクションとビューの作成を行う。

ビュー関係がこれで終わるはずなので、先がかなり見えてきた。

``` rb
# admin/users_controller.rb
# editアクション部分のみ
# 登録ユーザー一覧を掲載するため、DBからデータを全て取得

def edit
  @user= User.find(params[:id])
end
```

``` 
# users/edit.html.slim
# new.html.slimと同じ内容になるはずなので、特に記載しない。
# 共通部分が多いので、パーシャル化する。
# 以下、パーシャル化したコード

h1 ユーザーの編集
= render partial: 'form', locals: { user: @user}
```

<br>

#### updateアクションとeditビューの修正

---

updateアクションを実装する。  

updateメソッドが成功したらindexアクションへとredirectさせる。  
失敗したら、editアクションへとrenderし、エラーを表示する。  
分岐はif文で書く。  

``` rb
# admin/users_controller.rb
# updateアクション部分のみ
# 登録ユーザーを更新するため、DBから該当データを取得

def update
  @user= User.find(params[:id])

  if @user.update(user_params)
    redirect_to admin_users_url(@user), notice: "ユーザー「#{@user.name}」を更新しました。"
  else
    render: edit
  end
end
```

続いて、editビューを修正する。  
なお、newアクションにおいてもエラーを共通して表示したいので、  
パーシャルに以下を追記する。  

``` 
# _form.html.slimに追記
# エラー関係部分のみ記載

* if user.errors.present?

  ul 

  + user.errors.full_messages.each do |message|

      li= message
```

<br>

#### destroyアクションの実装

---

destroyアクションを実装する。  

``` rb
# admin/users_controller.rb
# destroyアクション部分のみ
# 登録ユーザーを削除するため、DBから該当データを取得

def destroy
  @user = User.find(params[:id])
  @user.destroy
  redirect_to admin_users_url, notice: "ユーザー「#{@user.name}」を削除しました。"
end
```

<br>

#### yamlファイルへの追記

---

ユーザー管理画面を日本語化するため、以下を追記。  
インデントに誤りがあり、適用に時間がかかったが、日本語化に無事成功。  

```yml
# ja.yml

user:
  name: 名前
  email: メールアドレス
  admin: 管理者権限
  password: パスワード
  password_confirmation: パスワード（確認）
  created_at: 登録日時
  updated_at: 更新日時

```

<br>

#### ログイン機能の実装（Ch4-5-5）〜最初の管理者ユーザーを作るまで(Ch4-5-13)

---

Gitで作業をしていたら、git reset --softとすべきところを勢いで滑って、  
git reset --hardのまま実行してしまい、ワーキングツリー内のファイルが吹き飛ぶ。  

git reset --hard は慎重にやろう 笑  

あと、git resetする前には、とりあえずcommitしておくのが重要。  

<br>

#### データを絞り込む

---

RailsではDBからデータ検索するための充実した機能が備わっている。  
検索するだけでなく、更新や削除時に対象を絞り込む際にも利用できる。  

コード組み立ての際には以下を意識すること。  

```rb
# User部分 → 起点
# where部分 → 絞り込みの条件
# first部分 → 実行部分

User.where(admin:true).first
```

誤解を恐れずに自分の言葉に置き換えると、  
1. まずどこのデータを対象とするか決定する（user.tasksなどを起点とすることもできる）
2. 次に条件を指定する（sqlコマンドのようなメソッドを使用）
3. 処理メソッド（該当のものを取得、更新する、有無の確認など）

<br>

#### タスク一覧を作成日時の新しい順に並び替え（scopeを活用）

---

orderメソッドを活用する。  

なお、クエリー用のメソッドについては、scopeを活用するとよい。  
scopeを活用すると、カスタムのクエリー用メソッドとして活用できる。  

```rb
# models/task.rb
# 以下を追記

# recentメソッドというカスタムメソッドを登録
scope :recent, -> { order(created_at: :desc) }
```

続いて、このカスタムメソッドを活用する。  
これにより、作成日時が新しい順に並ぶようになった。  

```rb
# task_controller.rb
# 修正をするindexアクション部分のみ記載

  def index
    # .recentを追記し、並び順を指定
    @tasks = current_user.tasks.recent
  end
```

<br>

#### フィルタを使い、重複を避けるリファクタリング

---


taskコントローラ内にあるアクションのうち、４つのアクションに共通するコードがある。  

```rb
@task = current_user.tasks.find(params[:id])
```

フィルタを使って、該当のアクションの場合については以上のコードをセットするように設定する。  

```rb
#tasks_controller.rb

# only内に記載のアクションの前に、set_taskメソッドを起動
before_action :set_task, only: [:show, :edit, :updte, :destroy]

〜これまでに作ってきたコード〜
（共通するコードは@taskから始まるコードの行は削除）

# set_taskメソッドを追記し、4つのアクションに共通するコードをここへ
def set_task
  @task = current_user.tasks.find(params[:id])
end
```

<br>

#### 詳しい説明に含まれるURLをリンクとして表示

---

rails_autolinkというgemを導入し、URLが自動でリンクされるように設定する。  
なお、auto_link()で囲むだけでよい。  

無事実装すると、以下のとおりリンクが投稿時にHTMLタグを付けなくとも自動的にリンクが生成される。

<a href="https://gyazo.com/e3bc65cf3186ffd93c47313f87436163"><img src="https://i.gyazo.com/e3bc65cf3186ffd93c47313f87436163.png" alt="Image from Gyazo" width="600" border=1/></a>  