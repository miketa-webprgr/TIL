## 現場Rails Chapter04-5 ~ Chapter04-10
## 「現実の複雑さに対応する」
---

<BR><BR>

### Chapter04-5 「ログイン機能を実装する」
---

認証方法にもいくつもの方法がある。  
また、自力で実装するだけでなく、deviseなどのgemを使う方法もある。  
- メールアドレスとパスワードの組み合わせで認証を行う
- FacebookやTwitterなどの外部サービスにログインして、認証を行う

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
cookieを通してやりとりをするのは、単純な受付番号札だけにして、それはセッション（一連のやり取り）が終わったら破棄。  
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

```rb
bin/rails g model user name:string email:string password_digest::string
```

ここで復習だが、マイグレーションは２ステップ  

1. スキーマを変更するマイグレーションファイルの作成
2. マイグレーションファイルを「rails db:migrate」で走らせ、データベースに適応

マイグレートする前に、マイグレーションファイルにnullを受け入れない属性を設定する。  
他、同ファイルに必要な事前の設定を行っていく。  

```rb
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