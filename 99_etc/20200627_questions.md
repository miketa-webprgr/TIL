# 質問内容

こんな感じでまとめました。  
長くてすみません。

趣旨に合わないもの、まだ頑張って理解していくには早いもの、  
今回取り上げるには微妙なものはスルーで結構です。  

回答をいただけたら、Tech Essentialにざっとまとめてあげたいと思います！

1. 公式ドキュメントの読み方・活用方法
2. ルーティング・RESTfulについて
3. データベース関連
4. 就職・今後の勉強方針

## 1. 公式ドキュメントの読み方・活用方法

gemの使い方を学ぶ場合について。

githubのREADME、そこからドキュメントへのリンクがあるのは分かるが、実際のところ、  
「gemにどういう特徴があるのか、gemをどうやって使うのか」、公式サイト等を活用できたことが未だにない。  

真面目に読んでいくと理解するのに  ４〜5時間くらいかかりそうな印象を受けて、  
安易にQiitaに走るということがよくあるんですが、例えば、だいそんさんはどうやって活用しているのでしょうか。  

具体的には、RSpec の System Specを勉強をしている際、リターンキーを押すシミュレートをしたかったのですが、  
伊藤さんのブログだと「JSドライバとしてPoltergeist」を使った事例だったので、`selenium-webdriver`  
を使った場合はどうすればいいんだと困ったことがありました。

「これか！」と特定できたところまではよかったんですが、  

> https://www.selenium.dev/documentation/en/webdriver/keyboard/

どう設定していいか分からず（基礎力・そもそもの設定方法が分からないところが原因？）、  
Stackoverflow等で発見した質問等を参考にトライアンドエラーをしたら、よく分からないけど上手くいった  
ということがあったので、具体的にこうやっているっていうのを実演風にやってもらえると嬉しいです！

## 2. ルーティング・RESTfulについて

どのようにルーティングテーブルを設計すればいいか教えてください。  
（設定方法は大体分かってきた、もしくは調べれば分かるので、何をどう使えばいいのかという考え方について）  

けど、これは１ヶ月ぐらい前に盛り上がった回で話をしていたかもしれないですね。。。

### 具体例：Everyday Rails のアプリ

- [アプリの画面] https://i.gyazo.com/de6b6131778cf840a5792dacd78640e2.png
- 基本的には納得しているが、各Userに所属するという意味でUserについてはURLの下になぜネストしないのか
  - UserAさんとUserBさんのURLが同一になる可能性があるので、RESTfulではないような気もする。。。
  - ベストプラクティスは何か。何を考えて設計しているのか。

- namespace
  - 名前空間とは、メソッド名や変数名などが衝突しないようにするための機能
  - 意味は分かったが、なぜ管理系のAdminなんかでよく使われるのか
  - 今回だと、APIでも使われているがなぜなのか

- member
  - DHH流だとmemberというのはよくない（アクションが増えるので）と思うが、なぜなのか。
  - skinny controllerに努めるべきだと聞くが、その意義が分からない。。。
    - [DHH流のルーティングで得られるメリットと、取り入れる上でのポイント \- KitchHike Tech Blog](https://tech.kitchhike.com/entry/2017/03/07/190739)

- その他のルーティング設定方法
  - 例えば、ルーティングにおける module はいつ使うのか
  - Rails guide に色々と設定方法がまとめられているが、どのような目的でどれを使えばいいかが分からない。。。
    - [Rails のルーティング \- Railsガイド](https://railsguides.jp/routing.html)

```rb: routes.rb
Rails.application.routes.draw do

  devise_for :users, controllers: { registrations: 'registrations' }

  authenticated :user do
    root 'projects#index', as: :authenticated_root
  end

  resources :projects do
    resources :notes
    resources :tasks do
      member do
        post :toggle
      end
    end
    member do
      patch :complete
    end
  end

  namespace :api do
    resources :projects#, only: [:index, :show, :create]
  end

  root "home#index"
end
```

## 3. データベース関連

私にはまだレベルが高過ぎる質問だとは思うんですが、  
気になったので一応質問します。

当日取り上げるかどうかはお任せします！

- redisって何ですか？
  - NOSQLとSQLの違い
- redis使うと早くなるらしいぐらいのことしか分からない
- sidekiqって何ですか？
  - 現場Railsで動かしたけど、何をやっているかよく分からない

### 参考

> [【Rails入門】Redisでセッションを高速化しよう！キャッシュも解説 \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/58218)

## 4. 就職・今後の勉強方針

初学者が就職した場合、どのような業務を行うのか？
達成すべきゴールをイメージし、逆算するような形で勉強したい。

- 比較的任せられそうな機能実装やバグ修正を行って、Gitでレビューを受けて・・・というイメージ？
- SQLってどのくらい知っておけばよいのか？
  - 自作アプリ程度だとActiveRecord使えればよさそうなので、SQLコマンドを忘れていきそう
- JSの勉強はどの程度必要なのか、インフラの勉強もした方がよいのか
  - Ruby・Railsが当然軸にはなると思うが、プラスアルファとしてどこに力を入れるとよいのか
    - JS/Vueなのか、インフラなのか、データベースなのか、APIなのか、その他なのか、、、
    - AWS・Dockerはポートフォリオに導入しましょうとよく聞くが、、、  
    - 多分、全部を頑張るのは難しいと思うので、その辺りの考え方を聞きたい
    - 単純に自作アプリで使いたい技術・作りたいものに使ってみたい技術という方向性だけだと不味いような気がしたので
