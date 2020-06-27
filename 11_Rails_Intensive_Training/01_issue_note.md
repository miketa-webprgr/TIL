# Issue01 ログイン機能の実装

## 求められている機能実装について

1. ログイン機能を実装してください。
2. その他初期設定を行ってください
3. generateコマンド時に生成されるファイルを制限する
4. ルーティング、JS、CSS、テストが自動生成されないようにする
5. タイムゾーンの設定
6. etc

## 分からない単語・概念等について

### git flowとは

git flowで出てくるブランチについて、ざっと確認。  
だいそんさんのノートを読むまで、developブランチの意義が理解出来ずにいた。  
特に、図が分かりやすくてありがたかった。  

#### 開発者視点でまとめていく  

#### 平常時

1. developブランチからfeatureブランチを作成する
2. featureブランチで機能実装などを行う
3. プルリクを送り、レビューを受ける
4. レビューでOKが出たら、developブランチにマージする
5. featureブランチは捨てる
6. developブランチがデプロイできるぐらいの作業量になったら、releaseブランチを作成する
7. releaseブランチにて、デプロイして問題がないか確認する
8. バグが見つかれば、developにマージしておく
9. 問題がなければ、masterにマージ！！！

#### 緊急時

1. masterブランチからhotfixブランチを作成
2. 急いでバグを修正！！！
3. masterにマージ！！！！！

#### 参考資料

- [git flowについて · DaichiSaito/insta\_clone Wiki](https://github.com/DaichiSaito/insta_clone/wiki/git-flow%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)
- [Git\-flowって何？ \- Qiita](https://qiita.com/KosukeSone/items/514dd24828b485c69a05)
- [いまさらだけどGitを基本から分かりやすくまとめてみた \- Qiita](https://qiita.com/gold-kou/items/7f6a3b46e2781b0dd4a0)

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

### i18n

### database.yml

### migrationファイル

### schema.rbとは

### config/application.rbとは

### yarnとは

- bootstrap material designを導入（gemだとうまく動かないのでyarnで導入）

### デバッグツールについて

- better_errorsを導入してエラー画面を使いやすくする  
- binding_of_callerを導入してエラー画面を使いやすくする
- pry-byebugを導入してデバッグ可能な状態にする
- pry-railsを導入してデバッグ可能な状態にする
git flow feature start
