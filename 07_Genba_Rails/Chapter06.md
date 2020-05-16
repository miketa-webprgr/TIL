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


