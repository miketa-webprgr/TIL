# herokuで非同期処理・定期処理を行いたい場合の実装について

## 考えているアプリ

- ユーザーがAPPL(APPLE.inc)などの株を選択
- Yahoo Fincance API から株価を取得（選択時にAPIを叩く）
- ３ヶ月分くらいの株価の終値を取得し、特定の条件に当てはまっているものは赤く色を塗る
- 毎日、定期的なバッチ処理を行い、該当の条件に当てはまるデータがあれば、ユーザーのメールに通知する

## どのような可能性が考えられるか

- heroku scheduler や process schedularの導入
  - これは、heroku無料プランのプロセス数としてカウントされるのか不明
- sidekiqの導入
  - 定期処理をしたいだけでは意味がないかも（処理が重たい場合は有効）
  - 使うには、Railsが動いている必要がある（heroku schedulerで叩き起こす必要あり）
- sidekiq-scheduler
  - 使えなくもないが、Railsとsidekiqが動いている必要がある（heroku schedulerで叩き起こす必要あり）

## 要確認事項

- sleep状態のとき、また復帰した際にどういう挙動になるのか、いまいちまだ理解できていない
- heroku無料プランだとプロセス数が２とあるが、何がプロセス数としてカウントされるのかいまいち分からない
  - railsとsidekiqはプロセス数として数えられるのは確定なんだろうけど、他で必要なものがないかよく分からない
  - 昔はworker dynoが有料だったので、unicornサーバーと組み合わせて、Web dyno内で完結していたもよう
    - [Rails\+Sidetiqでお手軽定時処理 on Heroku（無料！） \- 株式会社CFlatの明後日スタイルのブログ](http://cflat-inc.hatenablog.com/entry/2014/12/22/075907)
- redis と sidekiq を使う上では、connection size の設定で注意する点があるらしい
  - [Sidekiq on Heroku with Redis To Go Nano \- Manuel van Rijn](https://manuelvanrijn.nl/blog/2012/11/13/sidekiq-on-heroku-with-redistogo-nano/)

## 結論

- 手軽なのは、heroku + heroku schedulerなので、そこから始めたい
  - まず、単純な設計から初めてみて、少し余裕があればsidekiqの導入についても頑張ってみよう

## その他

- 関係ないが、Einstein Vision and Language（機械学習系）や Cloudinary（画像等のアップロード）は無料で使えるらしく、試してみたい
- 色々と調べたが、古い記事がどれだけ信じていいのか分からないし、とにかくやってみようと思う

## 参考記事

- [sidekiq\(\+active job\)をheroku redisで使う \| うえさいそメモ](https://uesaiso.netlify.app/entry/2019/04/06/_20190406sidekiq-on-heroku-redis/#heroku-redis%E3%81%AE%E8%BF%BD%E5%8A%A0)
- [Herokuでスケジューラ（cron）を設定する方法【Heroku Scheduler】 \- Reasonable Code](https://reasonable-code.com/heroku-cron/)
- [FAQ：Herokuはジョブ管理などにつかえるの？](https://builder.japan.zdnet.com/virtualization/sp_heroku2012/35018838/)
- [heroku schedulerの使い方\(Rails,sinatra\) \- Qiita](https://qiita.com/kyohei8/items/5a7d7db3838728a04140)
- [herokuでsendgridから定期的にメールを送ってみた、pythonで \- Qiita](https://qiita.com/noexpect/items/896c583945c2eec16093)
- [heroku schedulerの使い方　herokuで簡単な処理を定期的に自動的に実行する \- Qiita](https://qiita.com/Hijiri-K/items/d436d10d0e1ba6c573c0)
- [Deployment · mperham/sidekiq Wiki](https://github.com/mperham/sidekiq/wiki/Deployment)
- [Heroku Scheduler \| Heroku Dev Center](https://devcenter.heroku.com/articles/scheduler)
- [Process Scheduler \| Heroku Dev Center](https://devcenter.heroku.com/articles/process-scheduler)
