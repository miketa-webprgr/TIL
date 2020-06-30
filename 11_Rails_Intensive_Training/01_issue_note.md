# Issue01 ログイン機能の実装

## 求められている機能実装について

1. ログイン機能を実装してください。
2. その他初期設定を行ってください
    - generateコマンド時に生成されるファイルを制限する
    - ルーティング、JS、CSS、テストが自動生成されないようにする
    - タイムゾーンの設定
    - gemの導入

## 分からない単語・概念等について

### git flowとは

以下にまとめた。  

> [Gitflowとは](01_issue_note_gitflow.md)

最初は難しく感じたが、開発者視点で順を追っていくと理解できた。  
ざっと書いておくと、こんな感じ。  

- 基本的には、featureとdevelopを行き来する  
- 本番環境用のデプロイをする場合、developからreleaseブランチを切る  
- バグチェックを行い、releaseからmasterにマージ！!!

### generateコマンド時に生成されるファイルを制限

`application.rb`にて行う。  
以下にまとめました。  

> [application.rbとは](01_issue_note_applicaton_rb.md)

### turbolinks

以下が参考になった。  
[Turbolinks \| TECHSCORE\(テックスコア\)](https://www.techscore.com/tech/Ruby/rails-4.0/turbolinks/)  

> Turbolinks は Asset Pipeline を活用しているアプリケーションにおいて画面遷移を高速化するライブラリです。  
>
> 一般的にブラウザがページを表示するときには、  
>
> 1. ページ自体の HTML をダウンロードする。
> 2. ページの中で参照されているスタイルシートや Javascript をダウンロードする。
> 3. ページをレンダリングする。
>
> という順番で処理を行います。  
> ページ内でスタイルシートなどの他のリソースを参照している場合は、
> ブラウザと Web サーバの間で複数の通信が行われます。  
>
> スタイルシートや Javascript への参照は`<head>`内で行われることが多く、  
> Rails3 で登場した Asset Pipeline はその部分を最適化する手段を提供しました。  
>
> Asset Pipeline は複数のスタイルシートや Javascript を一つのファイルにまとめます。  
> ページから参照するスタイルシートや Javascript ファイルは一つだけになるため、  
> ブラウザとサーバ間の通信が抑えられます。  
>
> そして Rails4.0 で登場した Turbolinks は、リンクのクリックイベントをフックし、
> **Ajax リクエストに変換します**。  
> そしてレスポンス（新しいページの HTML）を受け取ると、現在のページの`<title>`と`<body>`を  
> 新しい HTML の`<title>`と`<body>`に交換します。  
>
> こうすることで、**<head> 内で参照されるスタイルシートや Javascript をブラウザが取得しなおす
> 処理をスキップすることができます**。  
> また、History API を用いてコンテンツのキャッシュやブラウザ履歴の操作が行われるため、  
> ブラウザの戻るボタンが押されたときにも違和感無く前のページの内容が復元されます。  

### coffee-script

JavaScriptの高級言語。
JSが簡潔に書けるということが魅力だったが、  
JSの問題点がES2015（ES6）へのバージョンアップにより改善されたので、  
採用されないケースが多くなっているらしい。  

ちなみに、Javaってジャパって発音しているが、ジャワカレーのジャワのこと。  
ジャワコーヒーのジャワから、Javaと命名したとの説があるらしく、  
その流れを汲んで coffee-script としたらしい。  

この辺りについては、以下のyoutube動画で知識を得ていたので、スムーズに理解できた。  
[【モダンJavaScript \#1】新章開幕！最も独学が難しい分野を徹底解説していきます！【フロントエンドエンジニア講座】 \- YouTube](https://www.youtube.com/watch?v=De9PH3EAz7c)  

### Slim

テンプレートエンジンの１種。  
テンプレートエンジンとは、ビューにおいてコントローラーで定義した  
変数を使用することができるようにする仕組み。  

RailsだとERB（Embedded Ruby）が標準搭載されている。  
Slimを使うと簡潔に書くことができる。  

haml, hamlit, faml というものもあるらしい。  
hamlの方がむしろ使用している人は多いらしい。  

違いをイメージする上で、下記の記事がコードも記載してあったので参考になった。  
[hamlとslimの両方を使ってみた感想 \- 営業職の俺がエンジニアになる〜群馬Web化計画編〜](https://toyokappa.hatenablog.com/entry/2017/09/10/231346)

### redisとは

現場RailsでSidekiqを使う時に導入したということ以外覚えていない。  

さらっとググってみたが、リレーショナルデータベース（SQL）でなくて、  
メモリ内に保存するから読み込みが早いということが以外全く理解できない。。。  

> Redisはインメモリで動作するKey-Valueストア(KVS)ソフトウェアです。  
> Redisのデータはすべてメモリ内に保存されるため、高速なデータの読み書きが可能です。  
> また、単純なキーと値のペアだけでなく様々なデータ構造が利用でき、データの永続化、  
> 冗長化、クラスタといった機能を備えており、様々な用途に対応できます。  
>
> [Redisの紹介](https://www.sraoss.co.jp/tech-blog/redis/redis-introduction/)から引用  

今回の事例に即していば、以下の記事が参考になった。  

「セッションの管理方法をクッキーストアではなくredisにする」ということなので、  
ブラウザ側にセッションを保存しないということらしい。

そもそもセッションの理解が浅いので、基本的なところから理解を深めていきたい。

[【Rails入門】Redisでセッションを高速化しよう！キャッシュも解説 \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/58218)  


[Redisとは？RailsにRedisを導入 \- Qiita](https://qiita.com/hirotakasasaki/items/9819a4e6e1f33f99213c)  

### sourceryとは

以下のQiita記事でイメージを把握した。  
使ってみないことには分からないと思うので、試してみたい。  

> deviseが「generate したらログインスパーン！！よっしゃ認証できんで！！！！」なのに対して  
> sorceryは「generateしたら、サブモジュールやら追加したるし、処理は自分で書くんやで」  
> と言ったイメージ。  
>
> 高機能の認証を手っ取り早く作りたいなら「devise」  
> 少し面倒臭さは感じても、ある程度はロジックを自分で書いていきたい方は「sorcery」  
>
> [シンプル認証gem sorceryを完全入門するで！！ \- Qiita](https://qiita.com/babashunsu/items/9937b0a2e08d318edece)  

### rubocop

Rubyのコードを綺麗に整えてくれる、警告してくれるgem。  
コミットする前に使うとよいかもしれない。  

> Rubyの静的コード解析を実行するgemです。  
> 難しく聞こえますが、要は RuboCop が `.rb` ファイルに記述してあるコードを検査し、  
> ここのコードは長すぎるね。とか、インデント入れたほうがいいよ。とかメソッド名変えようか。  
> とかをコマンド１つでターミナルに吐き出しててくれます。  

[RuboCop is 何？ \- Qiita](https://qiita.com/tomohiii/items/1a17018b5a48b8284a8b)  

### annotate

手動でこんなこと確かにやっていた。便利。  

> 各モデルのスキーマ情報をファイルの先頭もしくは末尾にコメントとして書き出してくれるGemです。  
> どんなカラムがあったっけ？ってなった時にいちいちdb/schema.rbを見に行く手間を省けます。  
> さらに、config/routes.rbにルーティング情報を書き出してくれる機能もあります。  
> いちいちrails routesを実行して確認する手間が省けますね。  

[【Rails】annotateの使い方 \- Qiita](https://qiita.com/kou_pg_0131/items/ae6b5f41c18b2872d527)

### MySQL

PostgreSQLと同じRDBMS。  
PostgreSQLよりメジャー。

設定方法について、需要がありそうな感じがあったので気合を入れて書いた。
若干時間を浪費してしまった感じがある。。。

[【MacOS \- Homebrew版】MySQLの設定方法（英語を翻訳してみた） \- Qiita](https://qiita.com/miketa_webprgr/items/ba7210ac57e2086fc5b6)  

なお、Railsへの導入については、以下の記事が分かりやすかった。  

[railsのDBをmysqlに変更する。 \- Qiita](https://qiita.com/pchatsu/items/a7f53da2e57ae4aca065)  

bundle install出来なかったが、以下が参考になった。  

[【Rails】MySQL2がbundle installできない時の対応方法 \- Qiita](https://qiita.com/fukuda_fu/items/463a39406ce713396403)  

For compilers to find openssl@1.1 you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"

順番も重要らしい
bundle config --local build.mysql2 "--with-cppflags=-I/usr/local/opt/openssl@1.1/include"
bundle config --local build.mysql2 "--with-ldflags=-L/usr/local/opt/openssl@1.1/lib"

[RailsプロジェクトでMySQLがbundle installできなかった \- Qiita](https://qiita.com/akito19/items/e1dc54f907987e688cc0)  

> Appleは、OpenSSLの使用を非推奨にしており、独自のTLSおよび暗号化ライブラリを支持しています
> Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries
>
> 通常、Appleのこの仕様による影響はありません。ただし、独自のソフトウェアが以下の式を必要とする場合、以下を追加する必要があります。
> Generally there are no consequences of this for you. If you build your own software and it requires this formula, you'll need to add to your build variables:
>
> LDFLAGS: -L/usr/local/opt/openssl/lib
> CPPFLAGS: -I/usr/local/opt/openssl/include
> PKG_CONFIG_PATH: /usr/local/opt/openssl/lib/pkgconfig

とりあえず、Appleが悪いってことでおk？笑

`rails new`する時は、以下のコマンドを入力。  

```text
bundle exec rails new instagram_clone -d mysql --skip-turbolinks --skip-test
```

```
bundle exec rake db:create
```

### i18n

### database.yml

- でーたべ

### migrationファイル

### schema.rbとは

### config/application.rbとは

RUNTEQのryotaさんに助けられました。

- [【Rails】rails generate controllerで生成されるファイルに制限をかける【config\.generatorsの設定】 \- Qiita](https://qiita.com/ryota21/items/643737b54f331b0aaa72)

### yarnとは

- bootstrap material designを導入（gemだとうまく動かないのでyarnで導入）

実際の作業においては、こちらが参考になった。
まさかの臺さんにお世話になりました 笑

[Cloud9上でYarnを使ってbootstrap material designを導入する \- Qiita](https://qiita.com/kenkentarou/items/e2ee6062fbff5d69fffd)  
[yarnの使いかた \- Qiita](https://qiita.com/senou/items/d939601e32c0005ebfe3)  


### デバッグツールについて

- better_errorsを導入してエラー画面を使いやすくする  
- binding_of_callerを導入してエラー画面を使いやすくする
- pry-byebugを導入してデバッグ可能な状態にする
- pry-railsを導入してデバッグ可能な状態にする


### その他初期設定を行う

- generateコマンド時に生成されるファイルを制限する
  - ルーティング、JS、CSS、テストが自動生成されないようにする

調べたら、RUNTEQの人に助けられた 笑
よくまとまっていた。

[【Rails】rails generate controllerで生成されるファイルに制限をかける【config\.generatorsの設定】 \- Qiita](https://qiita.com/ryota21/items/643737b54f331b0aaa72)

- タイムゾーンの設定

