## 現場Rails 自作アプリ作りに挑戦（練習-20時間程度） 

---

<BR><BR>

### 自作アプリ作りにあたって  
---

今回は、アプリ作りはSinatraの時に続いて２回目になる。  
前回の反省などを踏まえて、以下のとおりアプリ作りを進めていくこととした。  

- 現場での開発を想定して、Githubをきちんと活用すること。
  - 実装機能をリストアップし、完了ごとにプッシュしていく。
  - 実装機能ごとに目安となる作業時間を考えていくこと。
- あくまで練習であることを意識すること（新しいことに挑戦しない、復習に努める）。
- きちんと手順を踏んでやること。
  - アプリ作成の見取り図を作成してから、コーディングに取り掛かること。
- 時間はかかりそうだけど、ちゃんとテストも書くこと。
  - ただ、RSpecについて勉強不足なので、練習程度に少し書くだけでよいこととする。
- Bootstrapを適用していくが、練習の主眼ではないので時間をかけ過ぎないこと。  
（つまづいた場合、テキトーに済ますこと）

<BR><BR>

### 自作アプリの概要について
---

部活で部費を管理するアプリを作成することとした。  

部員ごとのアカウントがあり、部員が部に関わる支出を立て替えた場合、  
部の会計係に対して返還を求めることが出来る機能を実装する。  

また、会計係には特別な管理者アカウントがあり、立て替えをした部員に対して精算を終えた場合、  
そのデータを未精算から精算済に変更する機能を実装する。  

その他、部員に対して部費などの集金を求めることが出来る機能を実装する。  
部員から支払いがあった場合、そのデータを未集金から集金済に変更する機能も実装する。  

ログイン機能やアカウント作成機能も実装する。  

実際の利用を想定すると不足の機能等もあるだろうが、練習であるので気にしないこととする。  

<BR><BR>

### アプリ作成の見取り図を作成する
---

以下の記事を参考にした。  
[要件定義～システム設計ができる人材になれる記事 \- Qiita](https://qiita.com/Saku731/items/741fcf0f40dd989ee4f8)  

前段部分はビジネスを想定した話となっているので、中段部分の画面遷移図の作成あたりから参考にした。  

また、勉強し出すとキリがないので、ツッコミどころは山ほどあるだろうが、  
とりあえずデータの項目と型を決めておくことが重要であると考えたので、「ER図もどき」を作ってみた。  

<BR><BR>

### 画面遷移図とER図もどき
---

CacooというWebアプリケーションを使うことにした。  
作った図は以下のとおり。  

<a href="https://gyazo.com/7bb2ec501682728e8b11255730a8ec13"><img src="https://i.gyazo.com/7bb2ec501682728e8b11255730a8ec13.png" alt="Image from Gyazo" width="700" border=1/></a> 

<br>

<a href="https://gyazo.com/a26cba937a8717a3f7eb24c715346cfd"><img src="https://i.gyazo.com/a26cba937a8717a3f7eb24c715346cfd.png" alt="Image from Gyazo" width="700" border=1/></a>  

<br>

想像以上に作成に時間がかかってしまった。  

紙とペンだと、図を動かしたり、作業しながら適宜調整ができない。  
Cacooだと、操作性の問題なのか時間がかかってしまう。  

また、そもそも要件定義を甘く見ていたように思う。  
作ってみて感じたが、既に要件定義自体がアプリ作成なのであって、  
この作業を通じて、ルーティングやMVCのイメージを固めることができた。  

数時間で終わると考えていたが、一度も作ったことがないのにそんなはずはなかった。。。  

<BR><BR>

### 作業の流れ
---

箇条書きにしてみた。  
漏れや誤りなどあるだろうが、Githubのissue管理する際に参考にしていきたい。  

[未経験がWeb系転職成功したいならgithubでissue管理して開発しよう \- Qiita](https://qiita.com/fukubaka0825/items/c7710b4e87d478c8ba3b)  

こうやって書くと、20時間で終えるのは厳しいような気もする。。。  
（とはいえ、管理画面の実装やログイン機能の実装から逃げたら、練習にならない気がする）  

1. Rails new + 環境構築(主にgem関係)
2. ルーティングの設定
3. Billsモデルの作成
4. Billsに共通のnavバーを作成
5. Billsのindexビューを作成する
6. Billsのshowビューを作成する
7. Billsのeditビューを作成する
8. Billsのnewビューを作成する
9. パーシャル化する（show・edit・new）
10. Billsのコントローラに処理を記述する + 各ビューをform_withでつなげる
11. RSpecにテストを書く（create, update, destroyが機能するか）
12. Usersモデルの追加 + Association
13. bcryptを使用する（パスワードのダイジェスト）
14. Adminのindexビューを作成する
15. Adminに共通のnavバーを作る
16. Adminのshowビューを作成する
17. Adminのeditビューを作成する
18. Adminのdairi_newビューを作成する
19. Adminのshukin_newビューを作成する
20. パーシャル化する（show・edit・new）
21. Adminのコントローラに処理を記述する + 各ビューをform_withでつなげる
22. RSpecにテストを書く（create, update, destroy, change が機能するか）
23. RSpecにテストを書く（Billsのビューにて、Admin が create, update, changeしたものが反映されるか）
24. ログインのビューを作る
25. ログイン・ログアウトの機能を作る
26. RSpecにテストを書く（ログイン・ログアウト機能）
27. Billsのビューにて、ログインユーザーのbillsのみ表示されるようにする
28. RSpecにテストを書く（ログイン後、ログインユーザーのbillsのみしか表示されないかなど）
29. 余裕がなさそうだけど、できればUser関係まで頑張る
30. その場合、Profileモデルを追加し、associationする

なお、ビュー関係に手厚く書いて、コントローラ関係やモデル関係の処理に関する部分はかなり大雑把になったが、  
とりあえずこれで進めていくこととしよう。  

<BR><BR>

### GithubでIssueを作ってみる
---

先ほどの記事に従って、愚直にIssueを作ってみた。
また、このノートをベースにして、README.mdを更新してみた。

<a href="https://gyazo.com/2284a6b8c056df0335df22f9a16e4482"><img src="https://i.gyazo.com/2284a6b8c056df0335df22f9a16e4482.png" alt="Image from Gyazo" width="700" border=1/></a>

また、Githubのプロジェクトも使ってみた。  
計画を立てることが目的となっている感がひどいので、さすがに作業を始めることにする。  

さて、git pull して作業に移行する。

<br>

### 環境構築
---

現場railsで学習をしていた際には Rails 5.2.1 を使っていたが、今回はRailsの6.0.3バージョンを利用することとする。  
また、Rubyのバージョンについては、2.6.6にする。（2.7系の方が新しいが、gemとのトラブルが多そうな気がした。根拠はないけど）  

<a href="https://gyazo.com/5daf78b1862d643348d06ec69989125d"><img src="https://i.gyazo.com/5daf78b1862d643348d06ec69989125d.png" alt="Image from Gyazo" width="800"/></a>  

できました！  

さて、使うであろうgemを取り込んでいく。  
あまり吟味せず、現場railsで使ったgemの中からとりあえず必要そうなものを放り込む。  

```
Gemfile

# 検索機能等を追加するransack
gem 'ransack'
# URLをオートリンク化するgemを追加
gem 'rails_autolink'
# パスワードをdigest（ハッシュ値）として保存する
gem 'bcrypt', '~> 3.1.7' 
# Slimジェネレータ
gem 'slim-rails'
# Erb形式をSlim形式に変換するerb2slimコマンドを提供
gem 'html2slim'
# Bootstrapを読み込む
gem 'bootstrap'
# エラーメッセージを日本語化する
gem 'rails-i18n', '~> 5.1' 
# パスワードをdigest（ハッシュ値）として保存する
gem 'bcrypt', '~> 3.1.7'
# RSpecに関係するgem
gem 'rspec-rails','~> 3.7'
gem 'factory_bot_rails','~> 4.11'
```

・・・あ、ブランチを切るのを忘れていた。  

本当は先にブランチを切らなければならなかったのだが、改めて'#1'というブランチを作成。  
git add からの commit を決め（メッセージに #1 を必ず含めること）、git push していく。  

・・・作業自体はほとんどしていないけど。  

<a href="https://gyazo.com/302c8b474c642f8e013e59d4b2685ac6"><img src="https://i.gyazo.com/302c8b474c642f8e013e59d4b2685ac6.png" alt="Image from Gyazo" width="600" border=1/></a>  

さて、ここから一人二役の時間。  
下っ端のプログラマーとしてプルリクを上司に出して、次に上司としてプルリクの内容を精査する。  
問題がなければ上司としてプルリクを受け入れてマージする。  

そしてマージされたあと、コミットメッセージに #1 を入れたことにより、うまく issue に結びついている。  
結果が以下のとおり。  

<a href="https://gyazo.com/282d1f26a1b5fe8f2bb70c613406586b"><img src="https://i.gyazo.com/282d1f26a1b5fe8f2bb70c613406586b.png" alt="Image from Gyazo" width="600" border=1/></a>  

issue を閉じて、ブランチを削除する。  

そして、誰かが作業しているかもしれないという想定で、一応 git pull する。  

また、branch '#2' を作成し、次の作業に移行する。  

<br>

### 要件定義関係の修正
----

画面遷移図等をメンターさんに確認を依頼したところ、誤り等についても指摘されたが、  
そもそも実装機能が多過ぎるので、時間内に終わるような作業量であるとは到底思えないとの話があった。  

・・・そんな気はしてました（笑）  
ということで、吟味した結果、以下のように修正。  

<a href="https://cacoo.com/diagrams/MIS8VpBACc4MNzLs/0D12F"><img src="https://cacoo.com/diagrams/MIS8VpBACc4MNzLs-0D12F-w800-h600.png" border=1/></a>  

<br>

<a href="https://gyazo.com/488d48d0f8a34b1047c3084160e9a03d"><img src="https://i.gyazo.com/488d48d0f8a34b1047c3084160e9a03d.png" alt="Image from Gyazo" width="800" border=1/></a> 

<br>

なお、issue管理の際に活用するタスクリストについても修正した。  

1. Rails new + 環境構築(主にgem関係)
2. ルーティングの設定
3. Billsモデルの作成
4. Billsに共通のnavバーを作成
5. Billsのindexビューを作成する
6. Billsのshowビューを作成する
7. Billsのeditビューを作成する
8. Billsのnewビューを作成する
9. パーシャル化する（show・edit・new）
10. Billsのコントローラに処理を記述する + 各ビューをform_withでつなげる
11. RSpecにテストを書く（create, update, destroyが機能するか）
12. Usersモデルの追加
13. bcryptを使用する（パスワードのダイジェスト）
14. Admin/billsのindexビューを作成する
15. Adminに共通のnavバーを作る
16. Admin/billsのshowビューを作成する
17. Admin/billsのeditビューを作成する
18. Admin/billsのnewビューを作成する
19. パーシャル化する（show・edit・new）
20. Adminのコントローラに処理を記述する + 各ビューをform_withでつなげる
21. RSpecにテストを書く（create, update, destroy, change が機能するか）
22. RSpecにテストを書く（Billsのビューにて、Admin が create, update, changeしたものが反映されるか）
23. ログインのビューを作る
24. ログイン・ログアウトの機能を作る
25. RSpecにテストを書く（ログイン・ログアウト機能）
26. 今後の課題として、Userの登録・編集・削除機能を追加する

<br>

### railtiesのトラブル
---

さて、コントローラの作成手続きに入る。

```
rails g controller bills 
```

```
/Users/XXXXX/.rbenv/gems/2.6.0/gems/bundler-2.1.4/lib/bundler/resolver.rb:290:in `block in verify_gemfile_dependencies_are_found!':  
Could not find gem 'ransack' in any of the gem sources listed in your Gemfile. (Bundler::GemNotFound)  
```

いきなり、環境構築系のトラブル。  
確認すると、なぜかGemfileにはransackが書いてあるのに、Gemfile.lockにはransackがない。  

ハイハイ、もう一度 bundle install すればいいんでしょ。  

```
Bundler could not find compatible versions for gem "railties":
  In snapshot (Gemfile.lock):
    railties (= 6.0.3.1)

  In Gemfile:
    rails (~> 6.0.3, >= 6.0.3.1) was resolved to 6.0.3.1, which depends on
      railties (= 6.0.3.1)

    rails-i18n (~> 5.1) was resolved to 5.1.3, which depends on
      railties (< 6, >= 5.0)

Running `bundle update` will rebuild your snapshot from scratch, using only
the gems in your Gemfile, which may resolve the conflict.
```

ご、ごめんなさい。訳分からないです。  

・・・ググったりして、格闘中・・・

ん、bundleと相性が良いrailtiesがないと言っているのだから、bundlerが古いのでは！？  
じゃあ、bundlerをアップデートしてみよう。


・・・bundler、最新バージョンだった。。。  

え、じゃあrails-i18n関係なのか。  
これは新しいバージョンが出ているようなので、Gemfileを書き換えて対応してみる。  

```
# 修正前
gem 'rails-i18n', '~> 5.1' # For 5.0.x, 5.1.x and 5.2.x
# 修正後
gem 'rails-i18n', '~> 6.0.0' # For 6.0.0 or higher
```

解決した！  
railtiesなんとか、とか惑わすようなことを言われたので時間がかかってしまった。  

<br>

### ルーティングの設定
---

改めて、コントローラの作成手続きに入る。

```
rails g controller bills 
```

続いて、resourcesを使って、ルーティングの設定を行う。  
目指す形は以下のとおり。  

<a href="https://gyazo.com/d6b72599156f2220cb7e109ef5085de1"><img src="https://i.gyazo.com/d6b72599156f2220cb7e109ef5085de1.png" alt="Image from Gyazo" width="600" border=1/></a> 

多少苦戦するも、無事画面のとおり設定完了。  

```rb
Rails.application.routes.draw do
  root to: 'bills#index'

  resources :bills

  namespace :admin do
    resources :bills do
      member do
        post :done
        post :undone
      end
    end
  end

  get '/login', to: 'sessions#new'
  post '/login', to: 'sessions#create'
  delete '/logout', to: 'sessions#destroy'

end
```

```
             Prefix      Verb        URI Pattern                            Controller#Action
  -----------------      ------      ---------------------------------      -------------------
               root      GET         /                                      bills#index
              bills      GET         /bills(.:format)                       bills#index
                         POST        /bills(.:format)                       bills#create
           new_bill      GET         /bills/new(.:format)                   bills#new
          edit_bill      GET         /bills/:id/edit(.:format)              bills#edit
               bill      GET         /bills/:id(.:format)                   bills#show
                         PATCH       /bills/:id(.:format)                   bills#update
                         PUT         /bills/:id(.:format)                   bills#update
                         DELETE      /bills/:id(.:format)                   bills#destroy
    done_admin_bill      POST        /admin/bills/:id/done(.:format)        admin/bills#done
  undone_admin_bill      POST        /admin/bills/:id/undone(.:format)      admin/bills#undone
        admin_bills      GET         /admin/bills(.:format)                 admin/bills#index
                         POST        /admin/bills(.:format)                 admin/bills#create
     new_admin_bill      GET         /admin/bills/new(.:format)             admin/bills#new
    edit_admin_bill      GET         /admin/bills/:id/edit(.:format)        admin/bills#edit
         admin_bill      GET         /admin/bills/:id(.:format)             admin/bills#show
                         PATCH       /admin/bills/:id(.:format)             admin/bills#update
                         PUT         /admin/bills/:id(.:format)             admin/bills#update
                         DELETE      /admin/bills/:id(.:format)             admin/bills#destroy
              login      GET         /login(.:format)                       sessions#new
                         POST        /login(.:format)                       sessions#create
             logout      DELETE      /logout(.:format)                      sessions#destroy
```

### 2回目のプルリク
---

さて、commitも綺麗に整えて、プルリクを送る。  
一人でやっているのに、無駄に「恐れ入りますが」といったメッセージを入れてみる。  

<a href="https://gyazo.com/05cf3b881e1caec6eec42699ea5ede2d"><img src="https://i.gyazo.com/05cf3b881e1caec6eec42699ea5ede2d.png" alt="Image from Gyazo" width="600" border=1/></a>

説明が必要なコードにコメントを無駄に入れてみた。  

<a href="https://gyazo.com/972e0649bd3450eca3f4d7109cfa95c0"><img src="https://i.gyazo.com/972e0649bd3450eca3f4d7109cfa95c0.png" alt="Image from Gyazo" width="600" border=1/></a>  

さて、マージして、ブランチ削除 + issueクローズを行う。  
そして、新しくブランチ #3 を作成して、作業を再開する。  

こうやっていると、1人なのにチームで開発している感がして、ぼっちでも楽しい！  
かもね。。。  

<br>

### ルーティングの再設定
---

ルーティングを設定し終えて、流派はあるが、基本的にはresourcesにmemberやcollectionを使うのは望ましくないとの話を受けた。  
その際、以下の記事を紹介された。  

[DHH流のルーティングで得られるメリットと、取り入れる上でのポイント \- KitchHike Tech Blog](https://tech.kitchhike.com/entry/2017/03/07/190739)  

先ほどのルーティングは、doneとundoneというアクションを追加してしまったので、  
これをDHH流のルーティング設定に置き換えてみる。  

<a href="https://gyazo.com/9bb0f1197f5d252c0d6ce5f2400fff61"><img src="https://i.gyazo.com/9bb0f1197f5d252c0d6ce5f2400fff61.png" alt="Image from Gyazo" width="800" border=1/></a>  

```rb
Rails.application.routes.draw do
  root to: 'bills#index'

  resources :bills

  namespace :admin do
    resources :bills do
      resource :completion, only: [:update, :destroy], module: "bills"
    end
  end

  resources :sessions, only: [:new, :create, :destroy]
end
```

```
               Prefix      Verb        URI Pattern                                     Controller#Action
---------------------      ------      ---------------------------------               -------------------
                 root      GET         /                                               bills#index
                bills      GET         /bills(.:format)                                bills#index
                           POST        /bills(.:format)                                bills#create
             new_bill      GET         /bills/new(.:format)                            bills#new
            edit_bill      GET         /bills/:id/edit(.:format)                       bills#edit
                 bill      GET         /bills/:id(.:format)                            bills#show
                           PATCH       /bills/:id(.:format)                            bills#update
                           PUT         /bills/:id(.:format)                            bills#update
                           DELETE      /bills/:id(.:format)                            bills#destroy
admin_bill_completion      PATCH       /admin/bills/:bill_id/completion(.:format)      admin/bills/completions#update
                           PUT         /admin/bills/:bill_id/completion(.:format)      admin/bills/completions#update
                           DELETE      /admin/bills/:bill_id/completion(.:format)      admin/bills/completions#destroy
          admin_bills      GET         /admin/bills(.:format)                          admin/bills#index
                           POST        /admin/bills(.:format)                          admin/bills#create
       new_admin_bill      GET         /admin/bills/new(.:format)                      admin/bills#new
      edit_admin_bill      GET         /admin/bills/:id/edit(.:format)                 admin/bills#edit
           admin_bill      GET         /admin/bills/:id(.:format)                      admin/bills#show
                           PATCH       /admin/bills/:id(.:format)                      admin/bills#update
                           PUT         /admin/bills/:id(.:format)                      admin/bills#update
                           DELETE      /admin/bills/:id(.:format)                      admin/bills#destroy
             sessions      POST        /sessions(.:format)                             sessions#create
          new_session      GET         /sessions/new(.:format)                         sessions#new
              session      DELETE      /sessions/:id(.:format)                         sessions#destroy
```

<br>

### Billモデルの作成
---

以下のような形でモデルを作成する。  

<a href="https://gyazo.com/488d48d0f8a34b1047c3084160e9a03d"><img src="https://i.gyazo.com/488d48d0f8a34b1047c3084160e9a03d.png" alt="Image from Gyazo" width="800" border=1/></a>  

```
bin/rails g model Bill status:boolean name:string date:date item:string description:text price:integer
```

マイグレーションファイルにNOTNULL制約や一意制約を加え、マイグレートする。  
マイグレーションファイルは、下記のとおりである。  

```rb
class CreateBills < ActiveRecord::Migration[6.0]
  def change
    create_table :bills do |t|
      t.boolean :status, default: false
      t.string :name, null: false, unique: true
      t.date :date, null:false
      t.string :item, null: false
      t.text :description
      t.integer :price, null: false

      t.timestamps
    end
  end
end
```

ここで疑問を抱く。  
Modelでvalidationの制約をかけるのと、migrationで制約をかけるのに何の違いあるのか。  
また、テーブルへの制約のかけ方には、どのようなものがあるのか。  

[Ruby on Rails \- railsでアプリを作っています。DBのnull:falseとモデルでのvalidatesでのpresence trueの違いを教えてください｜teratail](https://teratail.com/questions/59911)  
[【Rails】「テーブルのカラムに定義するNot Null制約」と「モデルに定義するバリデーション（presence: true）」の挙動の違い。 \- Qiita](https://qiita.com/wonder_meet/items/fa804f0d436a29c97460)  
[Rails　テーブルの制約について \- Qiita](https://qiita.com/oteko7/items/2b03fa13d2f1c91e67e2)  

勉強になった。  
テーブルで制約をかけるのは保険のようなものであって、きちんとrailsのモデルの方でもvalidationをかける必要があるらしい。  

<br>

### validationの設定
---

続けて、バリデーションを設定する。  

```
t.boolean :status   →  値が存在すること
t.string :name      →  値が存在すること、値が一意であること、値に空白が存在しないこと（コールバックにて、保存する際に空白等を取り除く形とする）
t.date :date        →  値が存在すること
t.string :item      →  値が存在すること
t.text :description →  NULLを許容する
t.integer :price    →  値が１以上の整数であること
```

以上のような考え方に基づき、バリデーションを書く。  
以下の記事を参考にした。  

[Active Record バリデーション \- Railsガイド](https://railsguides.jp/active_record_validations.html#valid-questionmark%E3%81%A8invalid-questionmark)  
[Railsバリデーションまとめ \- Qiita](https://qiita.com/h1kita/items/772b81a1cc066e67930e)  

```rb
# Bill.rb
# コールバックにて、nameを保存する際に空白等を取り除く処理は、後ほど追加する）

class Bill < ApplicationRecord
  validates :status, inclusion: {in: [true, false]}
  validates :name, presence: true, uniqueness: true
  validates :date, presence: true
  validates :item, presence: true
  validates :price, numericality: { greater_than: 0 }
end
```

ここまで終わったので、Githubにpushしておく。




