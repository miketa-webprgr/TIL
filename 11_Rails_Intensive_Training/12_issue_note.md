# Issue12 メールのジョブの永続化を実装

## どんな感じ？

Raisガイドには以下のとおり書かれています。  

> デフォルトのRailsは非同期キューを実装します。これは、インプロセスのスレッドプールでジョブを実行します。  
> ジョブは非同期に実行されますが、再起動するとすべてのジョブは失われます。
>
> [Active Job の基礎 \- Railsガイド](https://railsguides.jp/active_job_basics.html#active-job%E3%81%AE%E7%9B%AE%E7%9A%84)

つまり、`Sidekiq` + キューを保存するための`Redis`が導入されていないと、  
Railsサーバーが再起動した途端、メール送信のジョブは全て失われてしまう。  

`Sidekiq`が導入されることにより、キューの保存や管理が可能になる。  
こんな感じで。  

<a href="https://gyazo.com/98fdc71c336dd5e59a50954ff6bf8db7"><img src="https://i.gyazo.com/98fdc71c336dd5e59a50954ff6bf8db7.png" alt="Image from Gyazo" width="700" border=1/></a><br>  

## 求められている機能実装・実装条件について

- メールのジョブを永続化させる
  - ActiveJobのアダプターにはsidekiqを利用する
  - ダッシュボード用にsinatraもインストールする

## はじめに

今回の場合、実装内容はかなり限られている。  

ただ、理解すべき内容は多いので、実装内容についてはノートを作るのは今回はやめて、  
むしろ概念の理解や挙動の仕組みについて重点的にノートを作成する。  

## Sidekiqが動く仕組みの概略・確認方法

今回の場合、`bundle install`して、`bundle exec rails server`していればよい訳ではない。  
よって、バックグラウンドでの`Sidekiq`の動かし方についてまず記す。  

仕組みの概略、同期・非同期の概念について触れた後、`Sidekiq`の動かし方や挙動について確認していく。  

### 1. Sidekiqが動く仕組み

英語になるが、以下のyoutubeが非常に分かりやすかった。  

- [Background Processing with Rails, Redis and Sidekiq](https://www.youtube.com/watch?v=GBEDvF1_8B8&t=693s)

その中でイメージしやすい図が使われていたので、引用させてもらう。  

<a href="https://gyazo.com/9e221ed0c44a731cb0d1e0959494ab89"><img src="https://i.gyazo.com/9e221ed0c44a731cb0d1e0959494ab89.png" alt="Image from Gyazo" width="700" border=1/></a><br>  

通常のHTTP通信では、以下のように実行される。  

```text
クライアントからのリクエスト
→ サーバからのレスポンス
→ クライアントからのリクエスト...
```

これを順番どおり実行していくような仕組みとなっている。  
これを「同期通信」という。  

例えば、動画をアップロードするような通信を「同期通信」で行うと、サーバーに動画をアップロードする処理が終わるまで、  
その後にちょっとトップページを見たいだけなのにもかかわらず、長々と足止めを食らうことになる。  

これでは非常に不便である。  

例えていうならば、レストランでお会計を済ませて帰りたいのに、店員さんが他のお客さんからステーキの注文を受けてしまったので、  
ステーキを焼き上げてその客に届けるまではお家計の対応ができない、というような状況である。  

これでは非常に困るので、店員さんはコックにステーキを焼くようにだけ伝えてもらって、すぐ店員さんに会計に来てもらいたい。  
つまり、ステーキを焼く行為と会計行為を同時に処理してもらえれば、問題が解決するというわけだ。  

サーバーにおいても、アップロードの処理とトップページのビューを返す処理を同時に行うと問題が解決する。
そこで、SidekiqとRedisを導入するとよい。  

SidekiqとRedisが導入されると、設定しておいた処理（今回の場合においてはメール送信）がキューとしてRedisサーバーに保存される。  
保存されたキューはSidekiqによって取り出され、「非同期」にて処理される。

### 2. 何を同期しているのか

先ほどの説明において、「同期」や「非同期通信」という言葉がしっくりこないかもしれない。  
そこで、HTTP通信において何を同期しているのか、ここで簡単に触れる。  

HTTP通信においては、以下の一連の通信において、互いに通信が上手くいったのか確認する。  
互いに状況を「同期」しながら通信を積み上げていくため、「同期通信」という。  

```text
クライアントからのリクエスト（投稿にいいねする）
→ サーバからのレスポンス（処理①：いいねされたユーザーに対する通知を作成）
→ サーバからのレスポンス（処理②：メールを送信する）
→ 処理が終了し、クライアントにHTTPレスポンスを返す

クライアントからのリクエスト（マイページにアクセス）
→ サーバーからのレスポンス（マイページのビューを返す）
```

非同期通信においては、状況を同期せず、同時にクライアントの処理を受ける。  
そして、終わった処理から適宜クライアント側に返していくため、「非同期通信」という。  

```text
クライアントからのリクエスト（投稿にいいねする）
→ サーバからのレスポンス（処理①：通知のリソースを作成）
→ 非同期で処理することとして、レスポンスをすぐに返さない（処理②：メールを送信する）

クライアントからのリクエスト（マイページにアクセス）
→ サーバーからのレスポンス（マイページのビューを返す）

非同期処理が終了
→ サーバからのレスポンス（処理②：メールを送信する）
```

非同期通信の方が処理が早く済むので良いことだらけと感じるかもしれない。  

ただ、特定の処理Aが終わったことを前提にしておこなれる処理Bがあったとすると、  
同期通信の場合は考えることが少なくて済んだが、非同期通信の場合は処理Bが先に終わってしまった  
場合についての対応についても考える必要があるため、考えることが多くなる。  

以下の動画が分かりやすく解説しているため、参考までにリンクを貼っておく。  

- [非同期処理とは何か？【超入門編/JavaScript/プログラミング】](https://www.youtube.com/watch?v=OBqj4I5NAEg)

## 3. Sidekiqの動かし方

`bundle install`と`bundle exec rails server`をするだけでは、今回のアプリは動かすことができない。  
そこで、これまでの概要を踏まえた上で、`Redis`と`Sidekiq`を動かす必要がある。  

まず、ターミナルで`redis-server`を入力し、redisサーバーを立ち上げる。  
サーバーを立ち上げると、以下のような画面が立ち上がる。  

このredisがキューの受け皿となる。  
Redisにキューが保存されるので、メールのジョブが永続される。  

```text
23005:C 12 Sep 2020 18:55:53.418 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
23005:C 12 Sep 2020 18:55:53.418 # Redis version=6.0.3, bits=64, commit=00000000, modified=0, pid=23005, just started
23005:C 12 Sep 2020 18:55:53.418 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 6.0.3 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 23005
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'            
```

続いて、`Sidekiq`立ち上げる。  

今回のアプリの場合、`Rails`と`Sidekiq`をつなぐものとして、`Active Job`が活用されている。  

非同期にてキューの処理を可能とするサーバーには、`Sidekiq`だけでなく`Rescue`や`Delayed Job`などがあるが、  
Rails標準のフレームワークである`Active Job`を活用することで、各サーバーの仕様の違いを気にすることなく実装できる。  
（逆に言えば、Sidekiqを使うと決め打ちすれば、`Active Job`を挟まずに`Sidekiq`を使うことができる）  

また、`Active Job`は`ActionMailer`と連携が取れているので、`deliver_later`メソッドなどでメール送信処理を実行  
すると、設定先であるサーバー（このアプリの場合は`Sidekiq`)に対して、`mailers`という名前でキューが自動で送られる。  

そこで、単に`Sidekiq`を実行するのではなく、以下のオプションで実行するとよい。  

```text
# defaultというキューとmailersという２種類のキューを受け付ける
bundle exec sidekiq -q default -q mailers
```

以上のコマンド及びActionMailerに関する使い方については公式のWikiできちんと説明されている。  

- [Active Job · mperham/sidekiq Wiki](https://github.com/mperham/sidekiq/wiki/Active-Job)

もしくは、今回のアプリの場合においては、`config/sidekiq.yml`に設定がされているので、  
この設定を活用する形で以下のように実行してもよい。  

```text
bundle exec sidekiq -C config/sidekiq.yml
```

なお、`config/sidekiq.yml`では以下のとおり設定されている。  

```yml
# 処理できるprocessの数を設定
concurrency: 25

# キューの設定
queues:
  - default
  - mailers
```

設定方法については、こちらで参照できる。  

- [Advanced Options · mperham/sidekiq Wiki](https://github.com/mperham/sidekiq/wiki/Advanced-Options)

実行すると、以下のような画面が立ち上がる。  
たしかに、サイドにキックしている。  

```text
2020-09-12T09:57:07.010Z pid=23122 tid=owwaw44ym INFO: Booting Sidekiq 6.0.1 with redis options {:url=>"redis://localhost:6379", :id=>"Sidekiq-server-PID-23122"}


               m,
               `$b
          .ss,  $$:         .,d$
          `$$P,d$P'    .,md$P"'
           ,$$$$$b/md$$$P^'
         .d$$$$$$/$$$P'
         $$^' `"/$$$'       ____  _     _      _    _
         $:     ,$$:       / ___|(_) __| | ___| | _(_) __ _
         `b     :$$        \___ \| |/ _` |/ _ \ |/ / |/ _` |
                $$:         ___) | | (_| |  __/   <| | (_| |
                $$         |____/|_|\__,_|\___|_|\_\_|\__, |
              .d$$                                       |_|
```

## 4. 実際の挙動を確認する

これで`Sidekiq`が動く準備が整った。  
厳密には違う部分もあるだろうが、この流れを頭に入れて追っていく。  

```text
RailsからSidekiqがキューをpush => Redis（キューを保存） => SidekiqがRailsに非同期でキューを実行させる
```

なお、GitHubの公式にてSidekiqのBasicsが説明されている。  

- [The Basics · mperham/sidekiq Wiki](https://github.com/mperham/sidekiq/wiki/The-Basics)

例えば、この投稿にいいねをして、「くらげ」というコメントをした場合の挙動を確認してみる。  

<a href="https://gyazo.com/39dd412de04122f885073a0020e51f3a"><img src="https://i.gyazo.com/39dd412de04122f885073a0020e51f3a.png" alt="Image from Gyazo" width="700" border=1/></a><br>  

すると、Railsサーバーは以下のような挙動になる。  
長くなるので、いいねしたところだけを抜粋する。  

```text
# 投稿をいいねした

Started POST "/likes?post_id=6" for ::1 at 2020-09-12 18:58:03 +0900
Processing by LikesController#create as JS
  Parameters: {"post_id"=>"6"}
  User Load (0.6ms)  SELECT  `users`.* FROM `users` WHERE `users`.`id` = 1 LIMIT 1
  ↳ /Users/kentasuedomi/.rbenv/versions/2.6.4/lib/ruby/gems/2.6.0/gems/activerecord-5.2.3/lib/active_record/log_subscriber.rb:98
  Post Load (0.4ms)  SELECT  `posts`.* FROM `posts` WHERE `posts`.`id` = 6 LIMIT 1
  ↳ app/controllers/likes_controller.rb:5
   (1.6ms)  BEGIN
  ↳ app/models/user.rb:51
  Like Exists (0.4ms)  SELECT  1 AS one FROM `likes` WHERE `likes`.`user_id` = 1 AND `likes`.`post_id` = 6 LIMIT 1
  ↳ app/models/user.rb:51
  Like Create (6.1ms)  INSERT INTO `likes` (`user_id`, `post_id`, `created_at`, `updated_at`) VALUES (1, 6, '2020-09-12 18:58:03', '2020-09-12 18:58:03')
  ↳ app/models/user.rb:51
   (2.0ms)  COMMIT
  ↳ app/models/user.rb:51
  User Load (0.3ms)  SELECT  `users`.* FROM `users` WHERE `users`.`id` = 3 LIMIT 1
  ↳ app/models/like.rb:35
   (0.3ms)  BEGIN
  ↳ app/models/like.rb:35
  Activity Create (6.1ms)  INSERT INTO `activities` (`subject_type`, `subject_id`, `user_id`, `action_type`, `created_at`, `updated_at`) VALUES ('Like', 56, 3, 1, '2020-09-12 18:58:03', '2020-09-12 18:58:03')
  ↳ app/models/like.rb:35
   (0.7ms)  COMMIT
  ↳ app/models/like.rb:35


# ActiveJobにより、いいねされたことを投稿者にメールで通知するキューがSidekiqに渡される

[ActiveJob] Enqueued ActionMailer::Parameterized::DeliveryJob (Job ID: e10cbb1a-3114-4787-b4bf-b89697b245fd) to Sidekiq(mailers) with arguments: "UserMailer", "like_post", "deliver_now", {:user_from=>#<GlobalID:0x00007fb354c4d178 @uri=#<URI::GID gid://insta-clone-app/User/1>>, :user_to=>#<GlobalID:0x00007fb354c4cb38 @uri=#<URI::GID gid://insta-clone-app/User/3>>, :post=>#<GlobalID:0x00007fb354c4c4f8 @uri=#<URI::GID gid://insta-clone-app/Post/6>>}


# いいねしたので、ビューをJSでrenderする

Rendering likes/create.js.slim
  Like Load (0.5ms)  SELECT  `likes`.* FROM `likes` WHERE `likes`.`user_id` = 1 AND `likes`.`post_id` = 6 LIMIT 1
  ↳ app/views/posts/_unlike.html.slim:1
  Rendered posts/_unlike.html.slim (5.0ms)
  Rendered likes/create.js.slim (17.0ms)
Completed 200 OK in 296ms (Views: 26.1ms | ActiveRecord: 37.3ms)
```

すると、Redisにキューが保存される。  

```text
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 6.0.3 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 23005
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'

23005:M 12 Sep 2020 18:55:53.424 # Server initialized
23005:M 12 Sep 2020 18:55:53.425 * Loading RDB produced by version 6.0.3
23005:M 12 Sep 2020 18:55:53.425 * RDB age 6613 seconds
23005:M 12 Sep 2020 18:55:53.425 * RDB memory usage when created 1.55 Mb
23005:M 12 Sep 2020 18:55:53.427 * DB loaded from disk: 0.003 seconds
23005:M 12 Sep 2020 18:55:53.427 * Ready to accept connections
```

キューの中身を確認しようと思ったが、相当大変そうだった。  
苦労して確認した記録が記されたQiita記事があったので、興味のある人は読んでみてもいいかもしれない。  

- [SidekiqはRedisに何を書き込んでいるのか \- Qiita](https://qiita.com/hosopy/items/d2c87b6489991091ddab)

さて、キューはSidekiqに引き渡され、以下のような挙動となっている。  

```text
               m,
               `$b
          .ss,  $$:         .,d$
          `$$P,d$P'    .,md$P"'
           ,$$$$$b/md$$$P^'
         .d$$$$$$/$$$P'
         $$^' `"/$$$'       ____  _     _      _    _
         $:     ,$$:       / ___|(_) __| | ___| | _(_) __ _
         `b     :$$        \___ \| |/ _` |/ _ \ |/ / |/ _` |
                $$:         ___) | | (_| |  __/   <| | (_| |
                $$         |____/|_|\__,_|\___|_|\_\_|\__, |
              .d$$                                       |_|

# いいねされたことを投稿者にメールで通知するキューがSidekiqに渡り、処理をする

2020-09-12T09:58:03.332Z pid=23122 tid=owwbsa376 class=ActionMailer::Parameterized::DeliveryJob jid=bf832c444294b2c70ba8296c INFO: start
/usr/bin/open file:////Users/~~~~~~/Desktop/website/insta_clone/tmp/letter_opener/1599904684_186125_a699d21/rich.html
2020-09-12T09:58:04.200Z pid=23122 tid=owwbsa376 class=ActionMailer::Parameterized::DeliveryJob jid=bf832c444294b2c70ba8296c elapsed=0.868 INFO: done

# コメントされたことを投稿者にメールで通知するキューがSidekiqに渡り、処理をする

2020-09-12T09:58:14.971Z pid=23122 tid=owwcbeb1q class=ActionMailer::Parameterized::DeliveryJob jid=21c61adb864b8912427859cd INFO: start
/usr/bin/open file:////Users/~~~~~~/Desktop/website/insta_clone/tmp/letter_opener/1599904695_041902_de3e4cc/rich.html
2020-09-12T09:58:15.045Z pid=23122 tid=owwcbeb1q class=ActionMailer::Parameterized::DeliveryJob jid=21c61adb864b8912427859cd elapsed=0.074 INFO: done
```

さて、実際にジョブが実行されたのか確認するため、`letter_opener_web`で確認する。  
無事、メール送信が実行されたことが確認できた。  

<a href="https://gyazo.com/e94cbe0c15ae139c20dd22c493ae9b31"><img src="https://i.gyazo.com/e94cbe0c15ae139c20dd22c493ae9b31.png" alt="Image from Gyazo" width="700" border=1/></a><br>  

## 5. ダッシュボードでSidekiqを確認する

`Sidekiq`には、ジョブの処理状況をGUIで確認できるインターフェースが備わっている。  
そのインターフェースは`sinatra`で動いているため、`sinatra`のインストールが必要となる。  

インターフェースを実装するため、`letter_opener_web`と同様に`routes.rb`にマウントする。  
`at`以下については自由に設定できるが、`/sidekiq`としている例が多いように思う。  

```rb
# routes.rb

Rails.application.routes.draw do
  if Rails.env.development?
    mount Sidekiq::Web, at: '/sidekiq'
  end

  # 以下省略
```

設定方法については、公式で参照できる。  

今回は開発環境に限定して導入しているので問題がないが、本番環境でも導入したい場合、  
セキュリティについても考える必要がある。（このままだとURLに誰でもアクセスできる）  

sorceryを使う場合についても公式で説明がされている。  
以下を参照するとよい。  

- [Monitoring · mperham/sidekiq Wiki](https://github.com/mperham/sidekiq/wiki/Monitoring)
- [Sidekiqをモニタリングする \[翻訳\] \- Qiita](https://qiita.com/ts-3156/items/2959c79c79bd9979ef97)  

設定が終わると、`http://localhost:3000/sidekiq/`にてダッシュボードが確認できる。  
なお、他のジョブも実行してしまったため、件数が３件となっている。  

<a href="https://gyazo.com/98fdc71c336dd5e59a50954ff6bf8db7"><img src="https://i.gyazo.com/98fdc71c336dd5e59a50954ff6bf8db7.png" alt="Image from Gyazo" width="700" border=1/></a><br>  

ちなみに、SidekiqのダッシュボードはSinatraで動いているため、ダッシュボードにアクセスしてからRailsに戻ると、  
セッションが失効し、ログオフした状態になってしまう。  

この問題を解決するには、Redisサーバーを２つ立てるのがよいらしい。  
公式には、以下のように書いてある。とりあえず、Google翻訳を貼っておきます。  

> 多くの人がキャッシュとしてRedisを使用していますが（Railsキャッシュストアとしてうまく機能します）、  
> Sidekiqをキャッシュとしてではなく永続ストアとして構成されているRedisインスタンスに対して実行することが重要です。  
> キャッシュとSidekiqにRedisを使用する場合は、適切に構成された2つの個別のRedisインスタンスを使用することをお勧めします。  
> Redis名前空間ではこの構成が許可されておらず、他にも多くの問題が発生するため、個別のRedisインスタンスを使用することを  
> 常にお勧めします。
>
> [Multiple Redis instances / sidekiq Wiki](https://github.com/mperham/sidekiq/wiki/Using-Redis#multiple-redis-instances)

また、こんな議論がある。全然読めていないけど。  

- [Not sharing session with sidekiq web\-ui · Issue \#34 · redis\-store/redis\-rails](https://github.com/redis-store/redis-rails/issues/34)

## 実装に関する補足

`config/application.rb`にて以下のとおり設定する。  

```rb
# config/application.rb

  config.active_job.queue_adapter = :sidekiq
```

これはバックグラウンドで使うキューの管理サーバーを指定するオプションである。  

- [4.2 バックエンドを設定する - Railsガイド](https://railsguides.jp/active_job_basics.html#%E3%83%90%E3%83%83%E3%82%AF%E3%82%A8%E3%83%B3%E3%83%89%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)

また、`config/initializers/sidekiq.rb`を以下のとおり設定する。  
ファイルは自分で作成する必要がある。  

```rb
# config/initializers/sidekiq.rb

Sidekiq.configure_server do |config|
  config.redis = {
      url: 'redis://localhost:6379'
  }
end

Sidekiq.configure_client do |config|
  config.redis = {
      url: 'redis://localhost:6379'
  }
end
```

こちらについては、`Sidekiq`の公式にて説明がされている。  

- [Using Redis · mperham/sidekiq Wiki](https://github.com/mperham/sidekiq/wiki/Using-Redis#using-an-initializer)  

`Sidekiq`の`configure_server`や`configure_clien`について意味が分からなかったので、  
改めて調べてみた。すると、英語であるが、以下のような質問が見つかった。  

- [What is a server and a client? · Issue \#638 · mperham/sidekiq](https://github.com/mperham/sidekiq/issues/638)
- [ruby on rails \- What is server and client terminology mean in Sidekiq? \- Stack Overflow](https://stackoverflow.com/questions/21670040/what-is-server-and-client-terminology-mean-in-sidekiq)

基本的な内容としては、以下を理解すると分かってくる。  

- [The Basics · mperham/sidekiq Wiki](https://github.com/mperham/sidekiq/wiki/The-Basics)

勘違いしている部分があるかもしれないが、Sidekiqには2つの機能がある。  
1つがクライアントとしてキューをRedisに送る機能であり、もう1つはサーバーとしてRedisからキューを取り出す機能である。  

クライアントとしてキューを送り出す先と、サーバーとしてキューを取り出す元をなぜ２回も設定する必要があるのか  
疑問を持つかもしれないが、それはどうやら、キューを送り出す機能やキューを取り出す機能を他に代替させることを  
視野に入れているからである。  

・・・まあ、正直に言って、このあたりはレベルが高くてちょっと手に負えないけど 笑  

## どうでもいい話

GitHubの公式を見ていたら見つけた。  
Qiita記事もあがっていた。  

[Sidekiq\.❨╯°□°❩╯︵┻━┻ \- Qiita](https://qiita.com/shouta-dev/items/42ae926effccd32edc8f)
