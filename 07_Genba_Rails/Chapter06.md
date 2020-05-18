## 現場Rails Chapter06 「Railsの全体像を理解する」

---

<BR><BR>

### Chapter06-1 「Railsを取り巻く世界」  

---

Railsは、Rubyをはじめとする様々な技術を使って組み上げられている。  
関連技術や重要な概念、そしてそれが全体としてどのように関係しているか把握するのが重要。  

<BR><BR>

### Chapter06-2 「ルーティング」  

---

ルーティングは、config/routes.rbで定義される。  

ルーティングは、以下の二つを連携している。  
1. URL及びHTTPメソッド
2. コントローラ・アクション

ルーティングは、以上の２つの間の架け橋となっている。  

例えば、URLに対して、GETやPOSTなどのHTTPメソッドでウェブブラウザ側からリクエストが来る。  
それに対して、サーバー側がそのリクエストを受けて、適切な処理プログラムにつなぐ。  

そのつなぐ先が、それがRailsなどのMVCモデルでいうところのControllerに該当し、  
ControllerがModelやViewとうまくやりとりをして、レスポンスとしてウェブブラウザにHTMLとして返す。  

Railsの場合、RESTfulという概念が重要になる。  
RESTfulという概念が分かれば、ルーティングを行うにあたっての方向性が分かるようになる。  

そして、RESTfulの考え方に影響を受けている、Railsの「resources」について学んでいく。  

<br>

#### ルートを構成する５つの要素

---

1. HTTPメソッド（GETやPOST）
2. URLパターン（'/tasks'や'/tasks/:id'）
3. URLパターンの名前（URLに対して一意な名前が付けられる）
4. コントローラ(アクションを束ねているもの)
5. アクション（具体的なメソッド）

なお、ルーティングは、HTTPメソッドとアクションをつなぐだけではなく、  
URLパターンに名前を付けて、その名前を元にURLを簡単に生成するためのヘルパーメソッドを作る。  

<br>

#### １つのルートを定義する

---

```rb
get '/login', to 'sessions#new'
```

Sinatraを使った際に学習済みではあるが、以上のルーティングは、  
- 「GETというHTTPメソッドでアクセスしたら、sessionsコントローラのnewアクションを呼び出す」  
- 「'/login'というURLをlogin_pathというヘルパーメソッドで生成できるようにする」  
という２つのことを意味している。  

なお、原則としてURLを得るためにはURLヘルパーメソッドを使うのが望ましい。  
なお、ハッシュを使い、{ controller: :sessions, action: :new}と書くこともできる。  

<br>

#### RESTfulの概要

---

RESTful = REpresentational State Transfer  

いまいちピンとこなかったが、  
「Representational = 何かをRepresent（表す・代理する）ような 」という意味で捉えると把握できた。  

要は、「URLとHTTPメソッドで、何をしようとしているか分かるような設計にしろよ！」 ということだ。  

以下の記事を読んで、理解を深める。  
[REST入門 基礎知識 \- Qiita](https://qiita.com/TakahiRoyte/items/949f4e88caecb02119aa)  
[アーキテクチャスタイル「REST」とは何か \| Think IT（シンクイット）](https://thinkit.co.jp/free/article/0609/8/4)  

なお、RESTfulな設計においては、URLのパターンは名称（しかも複数形）にすることが望ましいとされている。  
細かい話が以下に書いてあったので、参考に読了。  
[RESTful APIのURI設計\(エンドポイント設計\) \- Qiita](https://qiita.com/NagaokaKenichi/items/6298eb8960570c7ad2e9)  

<br>

#### resourcesでCRUDのルートを定義する
---

書籍に書いてあるとおりだが、resources及びresourceを使うと、自動でルート設定を行う。  
どのように行うかは下記のとおり。  
[Rails resourcesメソッドとresourceメソッド \- Qiita](https://qiita.com/Tamitchao/items/6f45aa6daf1412b78d10)  

なお、resourcesは複数のリソースがある場合に使い、resouceは一つしかない場合に使う。  
例えば、Twitterのようなサービスにおいてのユーザー情報であれば、複数のユーザーがいるはずなので、  
resourcesを使ってルーティングするのが望ましい。  

他方、管理者画面を作成するのであれば、１つあれば結構なので、resouceを使うとよい。  
一覧画面と詳細画面に分ける必要がないので、indexは存在しない。  

なお、resourcesなどをベースとして、一部のアクションしか使わないこともできる。  
具体例は下記のとおり。  


```rb
# indexアクションだけ使う
resources :tasks, only [:index]
# delete, edit, updateアクションは使わない
resources :tasks, except [:delete, :edit, :update]
```

<br>

#### routes.rbの構造化
---

正直、scopeやnamespaceについての説明は、現場railsだけではよく分からず。  
以下を読んだら、すっきりと理解できた。  

[routeのmoduleとnamespaceとscopeの違い \- Qiita](https://qiita.com/blueplanet/items/522cc8364f6cf189ecad)  
[RailsのRoutingネストについて \- Qiita](https://qiita.com/keisukegdk/items/beb5a62c17278c25c00d)  
[Railsのルーティングの種類と要点まとめ \- Qiita](https://qiita.com/senou/items/f1491e53450cb347606b)  

なお、routes.rbの整理のコツとして、URLの構造とコントローラの構造を意識すべきであり、  
筆者のおすすめとしては、コントローラの構造に沿った形で整理すべきだという指摘があった。  

要は、①の方が②よりオススメということらしい。  

① コントローラ構造に沿った書き方
1. まず、コントローラに関するルーティングを書く
2. 次に、Bコントローラに関するルーティングを書く


② URL構造に属した書き方
1. まず、/tasks/~というURLに属するルーティングを書く
2. 次に、/admin/~というURLに属するルーティング

<BR><BR>

### Chapter06-3 「国際化」  

---

国際化の３ステップ
1. 翻訳データをconfig/localesの下に配置する
  - 保存するデータはymlファイル
2. 現在のロケール（国や地域の設定）を示す「I18n.locale」を正しく設定する
  - 設定すれば、言語の切り替えもできる
3. 目的の翻訳データを利用する
  - I18n.tメソッドを使うとよい（翻訳してくれたり、日時等をローカライズしてくれたり）
  - 個別に指定することも可能

<br>

#### ユーザーごとに言語を切り替える
---

I18n.locale に en や jp などの値を代入することで、言語の切り替えが可能。  

なので、ユーザーごとに切り替えたい場合は、以下のようなコードを書くとよい。  
（ユーザーのDBにlocaleという属性を用意し、そこに言語を登録してもらっている前提）

```rb
# コントローラファイル
# 該当部分のみ記載

# まず、set_localeというメソッドを呼ぶように設定
before_action :set_locale

...

private

def set_locale
  # I18n.localに対して、現在のユーザーのlocale値を代入する
  # ユーザーがログインしていない場合は、ja とする
  # ここではぼっち演算子と||（or）を使って表現されている
  I18n.locale = current_user&.locale || :ja
end

```

あくまで雰囲気でしか読んでいないが、以下のような記事を発見。
実装する場合、参考にしたい。  

[Rails 多言語化対応 View編 \- Qiita](https://qiita.com/ayies128/items/64bbf8ed878daafcd4f1)  
[Ruby on Railsで言語切り替え機能を作る \- Qiita](https://qiita.com/lhside/items/52623ca8d09858fc7d6e)  

<br>

#### 翻訳ファイルの扱い方
---

現場Railsによると、以下のような形で翻訳ファイルを取得できる。  

```rb
# config/locales/jp.yml を参照して「タスク」を取得
Task.model_name.human(locale: :ja)
# config/locales/jp.yml を参照して「名称」を取得
TAsk.human_attribute_name(:name, locale: :ja)
# 現在のロケールが:jaである場合は{:tasks=>"タスク一覧"}とハッシュ形式で取得
I18n.t("taskleaf.page.titles")
```

なお、以下を参照したと仮定。  

```yml
#ja.yml

ja:
  activerecord:
    models:
      task: タスク
    attributes:
      task:
        name: 名称
        description: 詳しい説明
  taskleaf:
    page:
      titles:
        tasks: タスク一覧
```

Railsガイドを確認すると、以下のメソッドがあるとのこと。  
１番を使うことでモデル名、２番を使うことでアトリビュート名を取得できる。  

1. Model.model_name.humanメソッド
2. Model.human_attribute_name(attribute)メソッド


[Rails 国際化 \(i18n\) API \- Railsガイド](https://railsguides.jp/i18n.html#active-record%E3%83%A2%E3%83%87%E3%83%AB%E3%81%A7%E7%BF%BB%E8%A8%B3%E3%82%92%E8%A1%8C%E3%81%AA%E3%81%86)

slimに適当にコードを貼り付けてみたが、無事に表示された。  
<a href="https://gyazo.com/b0558898def84be296aa4bb3861afaf4"><img src="https://i.gyazo.com/b0558898def84be296aa4bb3861afaf4.png" alt="Image from Gyazo" width="581"/></a>  

<BR><BR>

### Chapter06-4 「日時の扱い方」  

---

日時を扱う場合には、タイムゾーンの問題に注意する必要がある。  
（国や地域によって時差が違うけど、どうするの問題）  

Railsでは、日時のデータをタイムゾーンと共に取り扱うことできる。  
タイムゾーンを扱いたい場合、ActiveSupport::TimeWithZoneクラスを用いる。  
扱わない場合、DateやTimeというRubyに備わっているクラスを活用するとよい。  

ActiveSupport::TimeWithZoneクラスを活用すると、  
TimeZoneを指定してあげることで、その地域の現在時刻が取得できるようになる。  

```
# UTCタイムゾーン（協定世界時間）にて時刻を表示
Time.zone.now
=> Sun, 17 May 2020 06:25:24 UTC +00:00

# Timeオブジェクト に’Asia/Tokyo’を代入
Time.zone = 'Asia/Tokyo'

# 日本時間にて、現在時刻を表示
Time.zone.now
=> Sun, 17 May 2020 15:26:19 JST +09:00
```

<br>

#### 日時の扱い方に関する設定
---

以下を意識して設定すること。
1. DBに保存する時にどのタイムゾーンの表現とするか
2. DBから日時データを取り出す際、どのタイムゾーンのオブジェクトとして取り出すか。

さて、試しにデフォルトのタイムゾーンをUTCから日本時間に変更する。  
その場合、config/application.rbを修正して、以下のコードを追加する。  

```rb
# config/application.rb
# 該当部分のみ抜粋

module Taskleaf
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 5.2
    config.time_zone = 'Asia/Tokyo'

    〜

  end
end  

```

すると、タスク一覧が日本時間で表示されるようになる。  
なお、Time.zone.nowというメソッドを使ったが、Time.currentという書き方もある。  

[RubyとRailsにおけるTime, Date, DateTime, TimeWithZoneの違い \- Qiita](https://qiita.com/jnchito/items/cae89ee43c30f5d6fa2c)  
[あなたはいくつ知っている？Rails I18nの便利機能大全！ \- Qiita](https://qiita.com/Kta-M/items/bd4ba36a58ad602a9d8b)  

<BR><BR>

### Chapter06-5 「エラー処理のカスタマイズ」  

---

RailsのProduction画面では、通常の開発用の画面とは異なり、エラーの詳細を見せないようになっている。  
そのままでも問題がないが、いかにもRailで作った感を出さないよう、カスタマイズすることができる。  

<br>

#### Railsのエラー処理の詳細
---

エラー処理においては、以下のプロセスを経て、表示する画面が決定されている。
1. アプリ固有のエラー設定に当てはまれば、指定の画面を表示
2. 当てはまらず、「config.consider_all_requests_local」がtrueであれば、デバッグ用エラー画面
3. 当てはまらず、「config.consider_all_requests_local」がfalseであれば、本番用のエラー画面

1の場合、rescue_fromというメソッドを利用することになる。  

2の場合、いわゆるいつも見ている画面が出てくる。  

3の場合、デフォルトで設定している画面もあるが、独自にエラー画面を設定することもできる（静的なページに限る）。  
以下がコードである。  

なお、1の場合、下記のようなコードを書く。  

```rb
# コントローラファイルに記載

# このMyCustomErrorのところに、例えば「ActiveRecord::RecordNotFound」と指定し、独自のエラー  
# を設定することができる。そのエラー対応について、show_custom_error_pageメソッドで規定している。  
rescue_from MyCustomError, with: show_custom_error_page

def show_custom_error_page(error)
  # @errorにエラー内容を格納し、render先で表示できる
  @error = error
  render :custom_error
end

```

以下において、詳細な案内があった。
[Railsガイド 14\.2 rescue\_from](https://railsguides.jp/action_controller_overview.html#rescue-from)  
[【Rails5】rescue\_fromによる例外処理：アプリ固有のエラーハンドリングとエラーページ表示 \- Qiita](https://qiita.com/E-46/items/03c30c2d37aae3756ecc)


<BR><BR>

### Chapter06-6 「Railsのログ」  

---

<br>

#### ログの利用方法
---

ログの出力先：log/development.log（開発環境の場合）
環境ごとに該当のフォルダに出力される。  

・・・改めてファイルを見て、ログの長さに驚かされた！  

自分でログを出力させることもできる。  
loggerオブジェクトを使う。  

```rb
# tasks_controller.rb
# 既存のコードに、logger.debugで始まる１行を追加

  def create
    @task = Task.new(task_params.merge(user_id: current_user.id))

    if @task.save
      #ここにloggerオブジェクトを追加
      logger.debug "task: #{@task.attributes.inspect}"
      redirect_to @task, notice: "タスク「#{@task.name}」を登録しました。"
    else
      render :new
    end
  end
```

以下が該当のログ。
保存したログ自体は、独自の設定を行わずとも残っているが、好きな形で加工できる。  

```log
Started POST "/tasks" for ::1 at 2020-05-18 10:41:57 +0900
Processing by TasksController#create as HTML
  Parameters: {"utf8"=>"✓", "authenticity_token"=>"BuPhhrrdgBIK8A8AR+pqsdJInBE/rdFK8wby/IGizKjLOG6eAxTNy5ByY7TNpsma7Ap2YP1tJxXblnzGVQN+/A==", "task"=>{"name"=>"現場rails", "description"=>"ログの勉強でーす"}, "commit"=>"登録する"}
  [1m[36mUser Load (1.8ms)[0m  [1m[34mSELECT  "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2[0m  [["id", 2], ["LIMIT", 1]]
  ↳ app/controllers/application_controller.rb:13
  [1m[35m (0.2ms)[0m  [1m[35mBEGIN[0m
  ↳ app/controllers/tasks_controller.rb:21
  [1m[36mCACHE User Load (0.0ms)[0m  [1m[34mSELECT  "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2[0m  [["id", 2], ["LIMIT", 1]]
  ↳ app/controllers/tasks_controller.rb:21

### 保存したログ自体は、以下に自動で残っている。

  [1m[36mTask Create (0.6ms)[0m  [1m[32mINSERT INTO "tasks" ("name", "description", "created_at", "updated_at", "user_id") VALUES ($1, $2, $3, $4, $5) RETURNING "id"[0m  [["name", "現場rails"], ["description", "ログの勉強でーす"], ["created_at", "2020-05-18 01:41:57.556984"], ["updated_at", "2020-05-18 01:41:57.556984"], ["user_id", 2]]
  ↳ app/controllers/tasks_controller.rb:21
  [1m[35m (11.2ms)[0m  [1m[35mCOMMIT[0m
  ↳ app/controllers/tasks_controller.rb:21

### ここからがコントローラで設定した部分 ### 

task: {"id"=>7, "name"=>"現場rails", "description"=>"ログの勉強でーす", "created_at"=>Mon, 18 May 2020 10:41:57 JST +09:00, "updated_at"=>Mon, 18 May 2020 10:41:57 JST +09:00, "user_id"=>2}

### ここまでがコントローラで設定した部分 ###

Redirected to http://localhost:3000/tasks/7
Completed 302 Found in 22ms (ActiveRecord: 13.9ms)
```

なお、ログレベルというものがあり、重要度ごとのメソッドがある。  
メソッドは下記のとおり。  

0. debug（全てのデバッグ用詳細情報）
1. info（通知）
2. warn（警告）
3. error（ハンドリング可能なエラー）
4. fatal（ハンドリング不可能なエラー）
5. unknown（原因不明のエラー）

例えば、logger.errorとすれば、全てのログではなく、エラーに関するもののみ出力できる。  

ログレベルの設定は、config/environmnets/development.rbに以下を追記する。  

```rb
  # 出力する内容をerrorレベルに限りたい場合
  config.log_level = :error
```


<br>

#### ログの設定
---

先ほどの事例のとおり、アプリケーションログにはパラメータなども出力される。  
ただ、パラメータにセキュアな情報が含まれる場合、あえて内容を隠したい場合がある。  

その場合、以下のように設定するとよい。  
特定のパラメータ値を隠すことができる。  

なお、デフォルトでは、パスワードは出力しないよう設定しているが、あえて解除することで、  
以下のとおりパスワードを出力させることができる。  

```rb
# config/initializers/fileter_parameter_logging.rb

# 以下のコードを消せば、パスワードまで出力されてしまう。
# 逆に、出力したくない項目を増やすこともできる。

Rails.application.config.filter_parameters += [:password]
```

ログを確認し、制限をしているコードを消すと、パスワードまで出てしまうことを確認。  
制限されている場合、以下のように表示されていた。  

```log
  Parameters: {〜 "password"=>"[FILTERED]"}
```

なお、ログファイルを日ごとや週ごとに分けるように設定することもできる。  
設定する際は、以下のように設定する。  

この場合、過去のログは「log/development.log.yyyymmdd」というファイル名になる。  

```rb
# config/environmnets/development.rb

config.logger = Logger.new('log/development.log', 'daily')
config.custom_logger = Logger.new('log/custom.log', 'weekly')
```

例えば、以下のようなコードをコントローラやビューに書くと、
その部分のコードが処理される度にログが残る。

```rb
# コードが走る度に'loggerに出力'が記録される
logger.debug 'loggerに出力'
# コードが走る度に'cutstom_loggerに出力'がcutstom_loggerに記録される
Rails.application.config.custom_logger.debug 'custom_loggerに出力'
```

以下が分かりやすく記載しており、参考になった。  
[Railsでlogを出力しdebugする \- Qiita](https://qiita.com/Kashiwara/items/f8a4030da6b17e96fabf)  


なお、オリジナルのロガーを作成したり、ロガーのフォーマットの設定を行うこともできる。  

<BR><BR>

### Chapter06-7 「セキュティを強化する」  

---

以下、３つの紹介を取り上げていく。  

1. Strong Parameters
2. CSRF対策
3. 各種インジェクション対策

<br>

#### 意図しないパラメータを弾く 「Strong Parameters」
---

paramsを受け取ってDBを更新するようなフォームがあった場合、  
Strong parametersを指定しておかないと、意図しない項目まで変更されてしまう恐れがある。  

例：開発者側が意図しないコードを送り込み、specialという有料会員かどうかを判断するカラムを修正する

```
  # paramsからspecialの値は取り出さない
  params.require(:task).permit(:name, :description)
```

<br>

#### CSRF対策
---

CSRF = Cross Site Request Forgery  
例：Twitterでの殺害予告など  

脆弱性のあるサイトなどを利用して、攻撃性のあるJavascriptなどを送りつけ、  
既にセッションがログイン状態にあるTwitterなどのアカウントで、  
勝手に殺害予告ツイートなどを投稿させる行為。  

なお、セッションは、通常クッキーに保存されており、それはCookie発行元のホストなど  
にしか送ることができないので、それ自体を受け取って悪意を働くことはできない。  

CSRFを防ぐには、そのリクエストがユーザーの意図によるものか確認する必要がある。  
一般的にはセキュリティトークンを発行し、照合する方法がよく用いられる。  

Railsでは、フォームから送られてくる情報には自動的にトークンを含めるような設定になっており、  
これと照合されるかコントローラで自動的にチェックするようになっている。  

なお、これはPOSTリクエストのみしか適用されないので、GETリクエストはparamsを受け取るような  
場合に安易に使わないように注意する必要がある。  

<br>

#### Ajaxリクエストへのセキュリティトークンの埋め込み
---

Webアプリでは、JavascriptによってPOSTメソッドでサーバーにリクエストを投げる場合もある。  
（いわゆるAjaxリクエスト）  

Railsでは、form_withからのPOSTだけでなく、Ajaxリクエストに対してもセキュリティトークンの埋め込みをサポートしている。  
具体的には、以下のような２段階方式を採用している。  
1. Railsが予めセキュリティトークンを出力
2. Javascriptがそのトークンを送る

トークンの送信は、Railsの機能が内部的に行うAjaxリクエストで自動的に行われる。  

<br>

#### インジェクションに注意する
---

インジェクションは、悪意のあるスクリプトやパラメータを入力し、実行させる攻撃。  
攻撃の標的となるのは、ユーザーがデータを入力可能なフォームなど。  

<br>

#### XSS

Cross Site Scripting  
脆弱性のあるサイトを利用して、Javascriptなどを送り込む攻撃。  

例えば、コメント機能がある箇所に scriptのタグで囲ったコードを送れば、  
セキュリティ対策がされていない場合、そのスクリプトが実行されるようなページに書き換えてしまうことができる。  

XSS防止のためには、危険なHTMLタグとして解釈されないように加工してから表示するような仕組みを作る。  

なお、Railsの場合、HTMLをエスケープする設定となっており、「&」「<」などは自動的に文字として表示するよう置き換えられる。  

そのままタグをある程度出したい場合、sanitizeヘルパーを用いて、危険なタグは出力しないよう設定するなどして対応する。  

<br>

#### SQL インジェクション

SQLインジェクションを使われると、SQLコードが意図しない形で書き換えられてしまうため、  
意図しないデータの流出や改竄などに繋がる危険がある。  

こちらも同じく、sanitizeするような対応を取ることが必要である。  

Railsでは、クエリメソッドに対してハッシュで条件を指定すると、自動的に安全化の処理を行ってくれるよう設定されている。  
よって、ハッシュで条件を指定するようにするのが、セキュリティ上の観点から推奨されている。  

<br>

#### Rubyコードインジェクション

Rubyにはあるオブジェクトのメソッドを任意に呼び出せる「Object#send」メソッドがある。  
ただ、ユーザーに好き勝手に使われると危険であるため、sendして渡してよいメソッドをホワイトリスト方式で限定した方が良い。  

また、「Kernel.#eval」メソッドも危険なので、注意すること。  

<br>

#### コマンドラインインジェクション

Railsアプリから、OSのコマンドを実行したいことがあるかもしれない。  
その場合、ユーザーの入力した情報をそのままコマンドラインに使うのは避ける必要がある。  

<br>

#### CSPの設定

CSPは、XSSやパケット盗聴といった特定の攻撃をブラウザ側で軽減するための仕組み。  

例えば、実行するスクリプトのドメインをホワイトリスト方式で指定すると、リスト外のスクリプトは実行しなくなる。  
これにより、例え脆弱性のあるサイトであっても、CSPの設定により悪意のjavascriptが走るのを食い止めることができる。  

Railsの場合、config/initializers/content_Security_policy.rbに記載することで設定できるが、  
開発したアプリが動かなくなることもあるため、十分なテストを行う必要がある。その際「report-only」モードを活用するとよい。  

<BR><BR>

### Chapter06-8 「アセットパイプライン」  

---  

アセットパイプラインは以下のような処理を行っている。  
なお、アセットパイプラインとは、JS, CSS, 画像などのアセットを効率的に扱うための仕組み。  

1. CoffeeScript, SCSS, Erb, Slimなどの高級言語を JS, CSS, HTMLなどにコンパイル（実行可能な言語に変換）する。  
2. 複数の JS,CSSファイルを１つのファイルに連結し、クライアントからのリクエスト数を減らす（読込時間の短縮に繋がる）。  
3. アセットの最小化（スペース、改行、コメントなどを削除）し、通信量を節約する。  
4. ダイジェストの付与（ハッシュ値をファイル名の末尾に付与することで、ブラウザキャッシュの影響で修正が反映されないことを防ぐ）  

<br>

#### 環境による挙動の違い
---

Railsは、development環境（開発）とproduction開発（本番）では挙動の仕方が異なる。  

とはいえ、development環境ではアセットパイプラインが使われず、  
production環境で使われる、といった単純なものではない。丁寧に見ていく。  

##### development環境

1. 高級言語のコンパイル → 行われる
2. JSやCSSファイルの連結 → 行われない
3. ファイルの最小化 → 行われない
4. ダイジェストの付与 → 行われる

2番の影響により、開発環境でのソースを確認すると、多くのCSSファイルへのリンクタグ、JSファイルへのスクリプトタグが生成される。  
なお、production環境ではファイル連結が行われるため、多くのタグは生成されない。

<br>

#### アセットファイルの読み込み
---

application.html.slimにて行われている。  
具体的には、head内にある下記のコードにて行われている。  

```
# application.html.slim

  = stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload'
  = javascript_include_tag 'application', 'data-turbolinks-track': 'reload'
```

<br>

#### アセットファイルはどのように生成されるのか
---

application.cssやapplication.jsは、マニフェストファイルと呼ばれる。  
設定等により変更可能だが、デフォルトのままであれば、'app/assets/*', 'lib/asssets/*',  
'vendor/assets/*'が探索パスに設定されている。  

<br>

#### アセットファイルはどのように生成されるのか
---

appplication.js において以下のようなコードがある。

```js
//= require rails-ujs
//= require activestorage
//= require turbolinks
//= require_tree .
```

以上のコードにより、読み込むディレクトリを指定している。  
指定の方法は、Sprocketsというgemに従うので、直接フォルダのディレクトリが書かれているわけではない。  

探索パスについてh、cofig/initializers/assets.rbで設定されている。  

Railsのコンソールで「Rails.application.config.assets.paths」と打ち込むことで、  
探索ぱすの一覧を確認することができる。  

<br>

#### アセット関連の設定
---

以下が設定先となる。  
詳細については、細か過ぎるような気がするので記載しない。  

- config/initializers/assets.rb
- config/environments/*


<BR><BR>

### Chapter06-9 「production環境でアプリケーションを立ち上げる」  

---

本番環境でアプリケーションを動かすため、必要となる基本的な設定作業がある。  
作業内容は以下のとおり。  

- アセットのプリコンパイル
- 静的ファイルの配信サーバの設定
- production環境用のデータベース作成
- config/master.keyが存在することを確認
- productionモードでサーバーを起動
- production環境用の秘密情報の管理

なお、実際はCapistranoなどでデプロイ作業を自動化することが多い

<br>

#### アセットのプリコンパイル
---

プリコンパイル = アセットパイプラインを実行して静的ファイルを事前に生成すること  

production環境でソースコードを更新してサーバーを起動する際には、必ずプリコンパイルを実行する必要がある。  
プリコンパイルは、Railsのコマンドとして用意されている。

プリコンパイルしたファイルは、「public/assets」の配下に保存される。

```
$ bin/rails assets:precompile

# なお、JSをプリコンパイルする場合、YarnというJSのパッケージマネージャが必要になる
```

<br>

#### 静的ファイルの配信サーバを設定する
---

Railsにも静的ファイルを配信する機能はあるが、本番環境ではNginxやApacheなどのWebサーバーに担わせることが一般的。  
（アプリケーションとしてのパフォーマンスを重視し、Railsには動的なコンテンツの配信に専念させるため）  

Railsには、静的ファイル配信機能をon/offする設定があり、production環境では基本的にoffに設定される。  
ローカル環境では、手軽に静的ファイルを配信させるため、onに切り替える必要がある。  

そのためには、~/.bash_profileに以下を記載する。  

```
export RAILS_SERVE_STATIC_FILES=1 
```

<br>

#### production環境用のデータベースを作成する
---

production環境を指定して、「bin/rails db:create」コマンドを使う。  

そのためには、以下の２つの設定作業が必要。  
1. postgresqlにtaskleafというユーザー(ROLE)を追加
2. taskleafユーザーがデータベースに接続する際に使うパスワードを環境変数「TASKLEAF_DATABASE_PASSWORD」で取得できるようにする  

```
createuser -d -P taskleaf
# その後、passwordを求められるので設定したいものを入力
# 再度設定するパスワードの確認が求められるので、再入力
```

その後、~/.bash_profileに以下を記載する。  

```
# password = 先ほど設定したもの

export TASKLEAF_DATABASE_PASSWORD=password
```

この後、production環境を指定して、db:createとdb:migrateを行う。  
```
RAILS_ENV=production bin/rails db:create db:migrate
```

<br>

#### config/master.keyの存在を確認
---

productionモードでアプリケーションを利用する場合、秘密情報を復号するための鍵の情報が必要になる。  
環境内にconfig/master.keyがあるはずなので、確認する。  

<br>

#### production環境でサーバーを起動
---

以下のコマンドを入力  

```
$ bin/rails s --environment=production
```

以下のとおり、サーバーの起動を確認できた。  
ただし、画面にCSSが反映されていなかったため、アセットパイプラインの設定に不具合があった可能性が高い。。。  

<a href="https://gyazo.com/e36763e50e50bf2f21b6e54478c9a231"><img src="https://i.gyazo.com/e36763e50e50bf2f21b6e54478c9a231.png" alt="Image from Gyazo" width="397" border=1/></a>  

<br>

#### production環境でサーバーを起動
---

例えば、TwitterのAPIを使ってタイムラインとアプリケーションを連携させる場合、  
APIの利用者を証明するための秘密情報を取得し、アプリケーションの中で利用することになる。  

その場合、Credentialsという機能を使う。  
Credentialsでは、秘密情報を構造化して記述してレポジトリで管理できるようにするが、  
レポジトリに入る内容はあるmaster.keyによって暗号化される。（master.keyはレポジトリ外で管理）  

<br>

#### master.keyについて
---

試したら、試したとおりになったが、一体何をやっているのかあまり把握できず。  
一度、デプロイなどをして、実践しないと理解が進まないかもしれない。  

<br>

#### アセットパイプライン関係の記事
---

[ebブラウザがWebページを表示するために必要なHTMLファイルの元ファイル（\.erb）をapp/viewsディレクトリ内に配置します。](https://www.transnet.ne.jp/2016/02/28/rails%E5%88%9D%E5%AD%A6%E8%80%85%E3%81%8C%E3%81%A4%E3%81%BE%E3%81%9A%E3%81%8Dcolnr%E3%80%8C%E3%82%A2%E3%82%BB%E3%83%83%E3%83%88%E3%83%91%E3%82%A4%E3%83%97%E3%83%A9%E3%82%A4%E3%83%B3/)  

[アセットパイプライン \- Qiita](https://qiita.com/krppppp/items/666d864e703a270fc8b6)  

[Ruby on Rails のアセットパイプラインの挙動を環境ごとに学ぶ \- 30歳からのプログラミング](https://numb86-tech.hatenablog.com/entry/2018/11/10/002439)

[Rails Assetの管理についてまとめる \- Qiita](https://qiita.com/shizuma/items/1980bf885906c73238b6)  

