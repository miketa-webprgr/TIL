# 20200628 だいそんさん質問会

## 参加者

- だいそんさん  
- Dai Kentaroさん（臺さん）  
- りゅういちさん  
- みけた

## 質問内容

1. 公式ドキュメントの読み方・活用方法
2. ルーティング・RESTfulについて
3. データベース関連
4. 就職・今後の勉強方針

詳細は以下を参照すること  
> [事前にみけたが送りつけた質問一覧](https://github.com/miketa-webprgr/TIL/blob/master/99_etc/20200627_questions.md)

## はじめに

これは、動画を元にざっと文章にまとめたものです。  
りゅういちさんが取ってくれたノートも活用しつつ、みけたが書き上げました。

みけたの解釈が多分に入っているので、きちんとだいそんさんが公開している動画も参照してください。  
動画自体が長いので、気になる部分を逆引きするような形で使ってもらえると幸いです。  

## 1. 公式ドキュメントの読み方・活用方法

だいそんさんは、gemの使い方が知りたい時、公式ドキュメント等をどう活用しているのか。  
よく公式を参照するといいと聞くが、未だ活用できた試しがないので質問しました！  

### 仕組みを理解することが重要 （1分20秒~)

- Qiitaを先に読んでしまうことある
- Qiitaで済ませるのが悪い訳ではない
- 公式も参照するように心がけているが、重要なのはコピペで済ませないこと
- Qiita・公式ドキュメントというところよりも、どのような仕組みで動いているのか理解するのが大切

## 2. ルーティング・RESTfulについて

どのようにルーティングテーブルを設計すればいいか教えてください。  
（技術的に`routes.rb`をどう書くかではなく、どのような考えで設計しているかについて）  

### resources, resource について（6分10秒~）

まず意識していることは、以下の内容。  
初学者は、意識していない人が多い。

- resources（複数形）を使うべきなのか
- resource（単数形）を使うべきなのか

具体的には、以下のような事例はありえないが、初学者にありがちなミス。  
各ユーザーがいくつものプロフィールを持つことはありえないので、  
`resource`を使うのが正解となる。

```rb: broutes.rb
resources :users
  resources :profiles
end
```

`users/:id/profiles/:id`としても動きはするが、`:id`が２や３になる可能性はない。  

### 隠れたリソースを探す努力をする（13分20秒~）

例えば、taskにdoneというアクションを追加する場合、  
以下のとおり、8つ目のアクションを追加しようと考えるのが自然かと思う。  

```rb: routes.rb
resources task do
  post :done, on: :member
end
```

ただ、以上のように安易にそうするべきではなく、  
以下のとおり、taskにtask_completionというresourceがあると考えるとよい。  

```rb: routes.rb
resources task do
  resouce :task_completion
end
```

ここでは、task_completionというモデルがある訳ではない。  
あるのはTaskというモデルだけである。

ただ、新しく、TaskCompletionsControllerを作り、  
task_completionという、taskが完了しているか未完了か示すような  
隠れたresourceを見つけることができれば、RESTに書くことができる。  

詳細はこちらを参照するとよい。  
> [DHH流のルーティングで得られるメリットと、取り入れる上でのポイント \- KitchHike Tech Blog](https://tech.kitchhike.com/entry/2017/03/07/190739)
  
アクションの数が７から９に増える程度であれば問題ないが、処理が複雑になり、  
アクションの数が20にも30にも増えてきて、独自のアクション名が増え出してくると、  
コードを書いていない他の開発者が読む際に、理解をすることが困難になってくる。  
（このあたりの話は36分あたりに出てくる）

それであれば、規約にガッツリと従って、サブアクションを増やす形がよい。  
（というのが規約を大事にする、DHH流の考え方らしいです）  

### ルーティングのネストについて（17分10秒~）

RESTな考え方に則って書いていくと、以下のようなルーティングになることもあるかと思う。  

```rb: routes.rb
resources users do
  resources tasks do
    resource :task_completion
  end
end
```

場合によっては、これ以上にネストが深くなることもあるかと思う。

ネストが深くなることは好ましくない。  
Railsガイドにも、ネストは１段階までに留めましょうと書いてある。  

> [Rails のルーティング（ネストについて） \- Railsガイド](https://railsguides.jp/routing.html#%E3%83%8D%E3%82%B9%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%E5%9B%9E%E6%95%B0%E3%81%AE%E4%B8%8A%E9%99%90)

ネストを避ける方法として、shallowオプションがある。  
shallowオプションについても、Railsガイドに説明がある。  

> [Rails のルーティング（shallowについて） \- Railsガイド](https://railsguides.jp/routing.html#%E3%80%8C%E6%B5%85%E3%81%84%E3%80%8D%E3%83%8D%E3%82%B9%E3%83%88)  

shallowオプションを使うと、以下のとおりURLを簡潔なものにしてくれる。  

> [resources を nest するときは shallow を使うと幸せになれる \- Qiita](https://qiita.com/kuboon/items/96bbd227f9497ed81f38)

ただ、shallowオプションによってURLが短くなるのは、  
メンバーのみ（show, edit, update, destroyの４アクション）であり、  
コレクション（index, create, newの３アクション）は短くならない。  

また、`tasks?user_id=1`というような形で、パラメータをURLに含めて送ること  
により対応する方法もある。（インスタクローンのいいね機能で勉強するとのこと）  

ただ、現実問題として、２段階以上になることもある。  
この辺りは、考え方に左右されるところではある。  

### namespaceについて（24分20秒~）

また、ルーティングを書く際に欠かせないのが、namespaceの活用である。  
以下のとおりnamespaceを使うと、URLが `mypage/articles/~`になる。  

```rb: routes.rb
namespace :mypage do
  resources :articles
end
```

namespaceを使うと、ロジックの実装が楽になる。  
自分が書いた記事のデータを削除したい場合、ccurrent_userを使って以下のように書ける。  

```rb
current_user.articles.find(params[:id]).destroy
```

ただ、namespaceを使わない以下のようなシンプルなルーティングを想定した場合、  
ロジックの実装が複雑になってくる。  

```rb
resources :articles
```
  
```rb
article = Article.find(params[:id])
if article.id == current_user.id
  article.destroy
```

要は、namespaceを使えば、`mypage/articles`に属するcontrollerを作り、  
mypageにログインしている前提で共通の処理を書くことができるのだが、  
名前空間を分けずに、何でもかんでも`articles`に書き込もうとすると、mypageにログインしているか  
いないか常にロジックを書いて分岐させなければいけない。（ということかと思います。。。）  

なお、namespaceで分けることで活用できるテクニックは以下に記載されている。  

> [Rails\(5\)namespace でファイルを分ける方法 \- Qiita](https://qiita.com/maggam/items/5c361558559f1c3488bf)  

### namespaceだけでは対応できない場合（29分40秒~）

以上のようなmypageにログインしている場合とそうでない場合ぐらいの棲み分けであれば、  
namespaceの活用で対応できる。  

ただ、場合によっては様々な権限を持つサービスが考えられる。  
例えば、以下のような事例。  

- 全権限を付与されている管理者
- 担当エリアに限り権限が付与されているサブ管理者
- 自分が投稿した記事に限り全権限が付与されているが他の人の記事は閲覧しかできない一般ユーザー
- 閲覧数が限られている未登録ユーザーなど  

このような場合、namespaceで区切るだけでは対応が困難なので、以下の「認可」管理のgemを使うとよい。  

- pundit
- banken

なお、認証と認可は頻出する重要なキーワードである。  
「認証 = 誰か確認すること（ログイン周りのこと）」、「認可 = 権限付与」というイメージで覚えるとよい。  

### APIにおいてnamespaceを使う意味（33分00秒~）

単純に通常ページとAPIページが違うので分けているだけ。  
WebAPIに関する部分だけ作るのであれば、わざわざnamespaceで区切る必要はない。  

### Skinny Controllerを目指すわけ（38分10秒~）

- コントローラを見やすくするため
- 現場railsのP429を参照（ロジックはモデルに寄せるとよい事例が掲載されている）

### moduleの使いどころ(42分20秒〜)

まず、`namespace`、`module`、`scope`については以下を参照。  

> [routeのmoduleとnamespaceとscopeの違い \- Qiita](https://qiita.com/blueplanet/items/522cc8364f6cf189ecad)  

このQiita記事にまとめれているとおり、`module`は`controller`の格納フォルダだけ、指定パスになる。  
例えば、就活サイトを作ると仮定した場合、`namespace`で以下のとおり分けることが考えられる。  

```rb
namespace :admin
  # 管理者用の画面
end

namespace :employers
  # 採用企業用の画面
end

namespace :employees
  # 就活する人が使う画面
end

namespace :users
  # 一般の人が見れる画面
end
```

ただ、一般の人が見れる画面のURLは、考え方によるかもしれないが、  
`users/~`から始まることは避け、`/users`を付けずに始めたいはずだ。  

そういった場合、moduleを活用するとよい。  

また、recipesコントローラの下にpublicationというコントローラを作るという場合、  
以下のとおり書くことができるが、その場合、recipesコントローラとpublicationコントローラが  
同じディレクトリに保存されてしまう。  

```rb
resources :recipes do
  resource :publication do
end
```

ただ、publicationというresourceは、recipesに属するものであるので、  
本当はrecipesコントローラのサブディレクトリに保存したい。  

その場合も、以下のようにmoduleを活用して書くとよい。  

```rb
resources :recipes do
  resource :publication do, module: :recipes
end
```

なお、このあたりの話は、既に取り上げた以下のブログに解説されている。  

> [DHH流のルーティングで得られるメリットと、取り入れる上でのポイント \- KitchHike Tech Blog](https://tech.kitchhike.com/entry/2017/03/07/190739)  

また、moduleと書かずに、以下のとおり書くこともできる。  

```rb
resources :recipes do
  resource :publication do, controller: "recipes/publication"
end
```

## 3. データベース関係

### Redisとは何か(52分50秒〜)

Redisについて勉強する場合は、Key Value Storeで調べるとよい。  
例えば、JSONファイルも、KeyとValueで構成されている。  

```json
{ name: "dyson" }
```

Redisでセッションを扱う場合、以下のように、  
`session_id`というKeyと`user_id`と`cart`のValueを保存している。  

```json
session_id_1234: { user_id: 1, cart: {macbookpro: 1} }
```

Redisは、複雑な処理を行うのには向かないが、処理がRDBMSと比べて圧倒的に早い。  
なので、sessionの保存や取り出しには、RedisのようなNoSQLが適している。  

なお、Redisを使う方法が「Redis Store」と言われるのに対して、  
Cookieを使う方法が「Cookie Store」と呼ばれている。  

#### Redis Store はなぜ Cookie Store よりセキュアなのか？(59分00秒〜)

この記事を見て質問した。  
> [【Rails入門】Redisでセッションを高速化しよう！キャッシュも解説 \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/58218)  

以下のようなセッション情報を取り扱うと仮定する。  

```json
session_id_1234: { user_id: 1 }
```

Redis Store を使う場合、クライアント側に渡すのは`session_id_1234`だけになる。  
他方、Cookie Store を使う場合、クライアント側に渡すのは`user_id: 1`になる（ただし暗号化した形で）。  

どちらも、クライアント側に渡すsession情報が盗まれたら、成り済ましは行われてしまう。  
そういう意味においては、セキュリティ上の優劣はつけられない。  

ただ、Redis Store の場合においては、`user_id: 1`に紐づけるsession_idを変えてしまえば、  
簡単に成り済ましを防ぐことができる。  

Cookie Storeの場合、クライアント側に`user_id: 1`という情報を丸ごと渡しているので、  
暗号化した`user_id: 1`を復号化する仕組みそのものを変えないと、成り済ましには対処できない。  
（具体的には、`secret_key_base`というものを変える必要がある。）  

復号化する仕組みそのものを変えると、`user_id: 1`のユーザーだけでなく、他のユーザーにも影響が出る。  

ユーザーが少ないアプリであればよいかもしれないが、常に多くの使用ユーザーを抱える大規模サービスにおいては、  
全てのセッション情報が無効化されてしまうので、あまり望ましい解決方法ではない。  
（例えばamazonの場合、カート情報が全て失われてしまう？ ）。  

セキュリティ上簡単に成り済まし対策ができるという点において、Redis Storeの方が好ましいと言える。  
なお、セッションについては以下を参考にするとよい。  

> [Webセッションとは何か？ついでにクッキーとは何か？ \| ただ屋ぁのブログ](https://tadtadya.com/web-what-is-a-session/)  
> [RUNTEQ勉強会レポート　Railsで学ぶCookieとセッション〜入門編〜 \| RUNTEQ \- 公式ブログ](https://blog.runteq.jp/programming-school/event/3111/)  

#### Sidekiqとは？(1時間04分50秒〜)

Sidekiqとはアダプターである。  

Sidekiqがあることで、非同期処理をしたジョブ（例えば、8時になったらDB上に登録してあるユーザーに  
一斉にメールを送る）を管理できるようになる。  

Railsguideに書かれているとおり、Rails単体だと、サーバーが落ちた場合、そのジョブが全て失われしまう。  

> Rails自身が提供するのは、ジョブをメモリに保持するインプロセスのキューイングシステムだけです。  
> プロセスがクラッシュしたりコンピュータをリセットしたりすると、デフォルトの非同期バックエンドの  
> 振る舞いによって主要なジョブが失われてしまいます。アプリケーションが小規模な場合やミッション  
> クリティカルでないジョブであればこれでも構いませんが、多くのproductionでは永続的なバックエンド  
> を選ぶ必要があります。  
>
> [Active Job の基礎 \- Railsガイド](https://railsguides.jp/active_job_basics.html) 

#### NoSQLとSQLの違いは？(1時間11分00秒〜)

- SQLは、複数のテーブルをSELECTやJOINなどで上手く取得したり、結合したりするイメージ
- NoSQLは、RedisとかFirebaseとか
  - schemaを定義しなくてよいので楽

詳しくは動画を見るなり、各自理解を深めるなりしてください 笑

## 4. 就職・今後の勉強方針

### 初学者が就職した場合、どのような業務を行うのか？(1時間15分00秒〜)

- 比較的任せられそうな機能実装やバグ修正を行って、Gitでレビューを受けて・・・  
  というイメージで概ね間違っていないと思う。  

- 臺さん（今年度からエンジニア転職を果たした）の場合、ペアを組んで仕事を行っているとのこと。  

### SQLってどのくらい知っておけばよいのか？ ActiveRecordに慣れていく心配...(1時間15分00秒〜)

- もちろん、知っておくにこしたことはないし、Active Recordを使う上でも理解が必要  
- だいそんさんは、中途採用の際、テーブル設計とデータ取得のSQL文について質問されたらしい
- 臺さんは、テーブル設計とAWSの構成図を考えさせるような質問をされたらしい
- SQLは、JOINとSELECTについては重点的に理解しておいた方がよい  
  - INSERTやUPDATE文については不要かも。。。
- 【メモ】コンソールで、`.to_sql`すればSQL文が見れる。

### JSの勉強はどの程度必要なのか、インフラの勉強もした方がよいのか...(1時間21分20秒〜)

- 当然、勉強の軸はRubyとRailsになる。  
  - APIも書けるとよい。
- インフラについては、現場に行かないと深い部分まで理解できないと思う。  
  - AWSで言えば、EC2などAWSの触りぐらいが分かればよいのではないか
- だいそんさんの意見として、インフラよりはJSをやった方がよいと思うとのこと。
- 企業側からすると何かに特化している方がタスクを振りやすいので、そこは意識するとよい

### 話の流れ上、APIについての話になった(1時間28分10秒〜)

- HTMLを返すか、JSONを返すかの違い
- 郵便番号投げたら住所返ってくるようなイメージでよい
  - [WebAPIについての説明 \- Qiita](https://qiita.com/busyoumono99/items/9b5ffd35dd521bafce47)
- ちなみに、Vueを使う場合、RailsではAPIを返す仕様になってくる
- 臺さんは、JbuilderというAPIに関係するgemを使って実装などをしたらしい

### りゅういちさんの質問（link_toメソッドでremote:trueが使われている場合）

インスタクローンの課題についての質問。  
`remote: true`が使われている場合、どこにlinkされているか分からなかったので質問した。  

- やりたいことについてエンドポイントを考えることが重要
  - 今回の場合、投稿に「いいね」をしたい
  - エンドポイントは、`/likes?post_id=1`

- エンドポイントから、`likes_controller.rb`に飛ばすようルーティングされていることが分かる
− `method: :post`でなので、`likes_controller.rb`の`create`アクションを参照する
- 今回は`remote: true`なので、`create.js.slim`を使ってレスポンスを返そうとする
