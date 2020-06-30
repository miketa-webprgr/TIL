# Issue01 ログイン機能の実装

## 求められている機能実装について

1. ログイン機能を実装してください。
2. その他初期設定を行ってください
    - generateコマンド時に生成されるファイルを制限する
    - ルーティング、JS、CSS、テストが自動生成されないようにする
    - タイムゾーンの設定
    - gemの導入

## 分からない単語・概念等について

### Railsでの環境構築全般

以下を参考にした。  
すぐ忘れる。  

[最速！MacでRuby on Rails環境構築 \- Qiita](https://qiita.com/narikei/items/cd029911597cdc71c516)  
[rails newするときによく使うオプションと、rails newした後によく行う設定 \- Qiita](https://qiita.com/jun_jun_jun/items/dd260c43387a8e17803d)  

### git flowとは

以下にまとめた。  

> [Gitflowとは](01_issue_note_gitflow.md)

最初は難しく感じたが、開発者視点で順を追っていくと理解できた。  
ざっと書いておくと、こんな感じ。  

- 基本的には、featureとdevelopを行き来する  
- 本番環境用のデプロイをする場合、developからreleaseブランチを切る  
- バグチェックを行い、releaseからmasterにマージ！!!

### generateコマンド時に生成されるファイルを制限

`config/application.rb`にて行う。  
以下にまとめました。同ファイルにて、以下を設定できる。  

- generateコマンド時に生成されるファイルを制限するには
- タイムゾーンの設定

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
> [【モダンJavaScript \#1】新章開幕！最も独学が難しい分野を徹底解説していきます！【フロントエンドエンジニア講座】 \- YouTube](https://www.youtube.com/watch?v=De9PH3EAz7c)  

### Slim

テンプレートエンジンの１種。  
テンプレートエンジンとは、ビューにおいてコントローラーで定義した  
変数を使用することができるようにする仕組み。  

RailsだとERB（Embedded Ruby）が標準搭載されている。  
Slimを使うと簡潔に書くことができる。  

haml, hamlit, faml というものもあるらしい。  
hamlの方がむしろ使用している人は多いらしい。  

違いをイメージする上で、下記の記事がコードも記載してあったので参考になった。  

> [hamlとslimの両方を使ってみた感想 \- 営業職の俺がエンジニアになる〜群馬Web化計画編〜](https://toyokappa.hatenablog.com/entry/2017/09/10/231346)

### redisとは

Key Value Store という方式でデータを保存している。  
メモリ内に保存するから読み込みが早いらしい。  

> Redisはインメモリで動作するKey-Valueストア(KVS)ソフトウェアです。  
> Redisのデータはすべてメモリ内に保存されるため、高速なデータの読み書きが可能です。  
> また、単純なキーと値のペアだけでなく様々なデータ構造が利用でき、データの永続化、  
> 冗長化、クラスタといった機能を備えており、様々な用途に対応できます。  
>
> [Redisの紹介](https://www.sraoss.co.jp/tech-blog/redis/redis-introduction/)から引用  

今回の事例に即していば、以下の記事が参考になった。  

「セッションの管理方法をクッキーストアではなくredisにする」ということなので、  
セキュリティ上成り済ましへの対応がしやすくなる。  

> [【Rails入門】Redisでセッションを高速化しよう！キャッシュも解説 \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/58218)  
> [Redisとは？RailsにRedisを導入 \- Qiita](https://qiita.com/hirotakasasaki/items/9819a4e6e1f33f99213c)  

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

> [【MacOS \- Homebrew版】MySQLの設定方法（英語を翻訳してみた） \- Qiita](https://qiita.com/miketa_webprgr/items/ba7210ac57e2086fc5b6)  

なお、Railsへの導入については、以下の記事が分かりやすかった。  

> [railsのDBをmysqlに変更する。 \- Qiita](https://qiita.com/pchatsu/items/a7f53da2e57ae4aca065)  

bundle install出来なかったが、以下が参考になった。  

> [【Rails】MySQL2がbundle installできない時の対応方法 \- Qiita](https://qiita.com/fukuda_fu/items/463a39406ce713396403)  

以下のコマンドを実行するとよい。  
まず、`brew info openssl`で確認する。  

```text
For compilers to find openssl@1.1 you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"
```

次に、以下のコマンドを実行する。  
順番も重要らしい。そのあと、`bundle install`する！  

```text
bundle config --local build.mysql2 "--with-cppflags=-I/usr/local/opt/openssl@1.1/include"
bundle config --local build.mysql2 "--with-ldflags=-L/usr/local/opt/openssl@1.1/lib"
```

以下の記事によると、Appleの使用により、bundle configする必要があるらしい。  
とりあえず、Appleが悪いってことでおk？笑  

> Appleは、OpenSSLの使用を非推奨にしており、独自のTLSおよび暗号化ライブラリを支持しています
> Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries
>
> 通常、Appleのこの仕様による影響はありません。ただし、独自のソフトウェアが以下の式を必要とする場合、以下を追加する必要があります。
> Generally there are no consequences of this for you. If you build your own software and it requires this formula, you'll need to add to your build variables:
>
> LDFLAGS: -L/usr/local/opt/openssl/lib
> CPPFLAGS: -I/usr/local/opt/openssl/include
> PKG_CONFIG_PATH: /usr/local/opt/openssl/lib/pkgconfig
>
> [RailsプロジェクトでMySQLがbundle installできなかった \- Qiita](https://qiita.com/akito19/items/e1dc54f907987e688cc0)  

`rails new`する時は、以下のコマンドを入力。  

```text
bundle exec rails new -d mysql --skip-turbolinks --skip-test
```

```text
bundle exec rake db:create
```

### i18n

> アプリケーションの文言を英語以外の別の1つの言語に翻訳する機能や多言語サポート機能を  
> 簡単かつ拡張可能な方式で導入するためのフレームワークを提供する。  
>
> [Rails 国際化 \(i18n\) API \- Railsガイド](https://railsguides.jp/i18n.html)  

なお、設定方法については以下にまとめられている。

> [\[初学者\]Railsのi18nによる日本語化対応 \- Qiita](https://qiita.com/shimadama/items/7e5c3d75c9a9f51abdd5)  

### database.yml

- データベースに関する設定を行うファイル
- 今回のケースにおいては、MySQL関連の設定をここで行う
  -ユーザIDやパスワードなどもここで設定する

> [Ruby on Railsにおけるdatabase\.ymlの設定方法 \- Qiita](https://qiita.com/azusanakano/items/863e72819be05c4969b7)  

### migrationファイル

DBのテーブルに変更を加える場合には、マイグレーションファイルを実行する。  
マイグレーションファイルは、gitのようにテーブル変更の履歴になる。  

違うユーザーが git clone する場合も、マイグレーションファイルを実行することによって、  
簡単に同じ環境を用意することができる。  

### schema.rbとは

データベースの構造を示すもの。  
データベースの構造を変えたいからと言って、直接schema.rbをいじっても意味はない。  

### config/application.rbとは

以下のとおりまとめた。

> [application.rbとは](01_issue_note_applicaton_rb.md)

### yarnとは

- bootstrap material designを導入（gemだとうまく動かないのでyarnで導入）
- gem 'font-awesome-sass' が必要

実際の作業においては、こちらが参考になった。
まさかの臺さんにお世話になりました 笑

[Cloud9上でYarnを使ってbootstrap material designを導入する \- Qiita](https://qiita.com/kenkentarou/items/e2ee6062fbff5d69fffd)  
[yarnの使いかた \- Qiita](https://qiita.com/senou/items/d939601e32c0005ebfe3)  

### デバッグツールについて

- better_errorsを導入してエラー画面を使いやすくする  
  - 画面がカッコよくなる！！！
- binding_of_callerを導入してエラー画面を使いやすくする
  - better_errorsでirbを動かすのに必要
- pry-byebugを導入してデバッグ可能な状態にする
  - これにより、binding.pryしたところから次の操作にnextしたりできる
- pry-railsを導入してデバッグ可能な状態にする
  - binding.pryが使えるようになる

> [pry\-byebugでrubyをデバッグする \- Qiita](https://qiita.com/AknYk416/items/6f0bec58712edaf4940e)  
> [今更聞けないpryの使い方と便利プラグイン集 \- Qiita](https://qiita.com/k0kubun/items/b118e9ccaef8707c4d9f)  

なお、`better_errors`などについては、以下のとおり、使うのは開発環境だけにしろとGitHubに書いてある。  
もちろん、他についても同様。  

> It is critical you put better_errors only in the development section of your Gemfile.  
> Do NOT run better_errors in production, or on Internet-facing hosts.  
