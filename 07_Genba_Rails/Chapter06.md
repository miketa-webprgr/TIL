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

