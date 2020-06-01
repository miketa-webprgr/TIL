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

<br><br>

## Issue 1.0 - Rails new + 環境構築 
---

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

<br><br>

## Issue 2.0 - ルーティングの設定  
---

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

<br><br>

## Issue 3.1 Billモデルの作成 
---

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

<br><br>

## Issue 3.2 Billsに共通のnavバーを作成
---

ナビゲーションバーを作成する。 

これは、showやeditなどでも共通して使用するものにはなるので、application.html.slim にコードを加える。  
ただし、admin/bills のビューを作成する場合には、application.html.slimからコードを移動させる必要がある。  

現場Railsで使ってきたコードを使って、応用する形で対応する。  

```slim
body
  / navbar-dark.bg-darkとした
  .app-title.navbar.navbar-expand-md.navbar-dark.bg-dark
    .navbar-brand 部費管理マネジャー
    ul.navbar-nav.ml-auto
      li.nav-item= link_to '申請一覧', root_path, class: 'nav-link'
      li.nav-item= link_to '申請する', new_bill_path, class: 'nav-link'
      li.nav-item= link_to '管理者ログイン', new_session_path, class: 'nav-link'
```

なお、bootstrapに関係する部分について、以下のサイトで確認をした。  
今回は、少しだけアレンジを加えて、黒色を基調とした。  

[Navbar \- Bootstrap 4\.2 \- 日本語リファレンス](https://getbootstrap.jp/docs/4.2/components/navbar/)  
[【Bootstrap】Navbarの使い方・カスタマイズ方法を徹底解説 \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/75948)  

[Tables \- Bootstrap 4\.2 \- 日本語リファレンス](https://getbootstrap.jp/docs/4.2/content/tables/)  
[Bootstrapでテーブルを利用する方法 \- Qiita](https://qiita.com/AquaMeria/items/b94d1d9ba074f04336b9)  

少し試したが、体系的に勉強をしないと仕組みを理解できない気がした。
だが、今回はbootstrap周りを理解することは目標ではないため、勉強はそれなりにして次の作業に移行する。  

なお、リンク先のページをまだ作成していないため、クリックするとエラーとなってしまう。  

<br><br>

## Issue 3.3 Billsのindexビューを完成させる
---

次に、ページの本体部分（container)部分を作成する。  
ここは、オリジナルで作成しなければならない。  

無駄にマージンやパディングを調整して時間がかかってしまったが、以下のような形で落ち着いた。  

```slim

h1.mt-5.mb-5 申請一覧

h4.pt-5.pb-3 請求中

table.table.table-hover
  thead.thead-default
    tr
      th= Bill.human_attribute_name(:date)
      th= Bill.human_attribute_name(:name)
      th= Bill.human_attribute_name(:item)
      th= Bill.human_attribute_name(:price)
    tbody
      - @bills.each do |bill|
        tr
          td= bill.date
          td= bill.name
          td= bill.item
          td= bill.price

h4.pt-5.pb-3 精算済

table.table.table-hover
  thead.thead-default
    tr
      th= Bill.human_attribute_name(:date)
      th= Bill.human_attribute_name(:name)
      th= Bill.human_attribute_name(:item)
      th= Bill.human_attribute_name(:price)
    tbody
      - @bills.each do |bill|
        tr
          td= bill.date
          td= bill.name
          td= bill.item
          td= bill.price
```

なお、マージンではなくてパディングを使った箇所、もしくはその逆などがあるかもしれない。  

マージンやパディングの意味は分かるけど、実践の中でどう使い分けたらよいか、いまいち分かっていない。  

また、このままだと寂しいし、ページの見栄えが確認しづらい。  
そこで、rails consoleを使って、データを格納してみた。  

本来はおかしいのだが、ロジックを書くのが面倒なので、上の表にも下の表にも同じデータが表示されるようにした。  

```rb
# bills_controller.rb

class BillsController < ApplicationController

  def index
    @bills = Bill.all
  end

  〜 他のアクションについても中身が空の状態で作成した 〜

end
```

その結果、以下のとおりとなった。  

<a href="https://gyazo.com/cf57dde74d39274bfe6fe328e1f3c53a"><img src="https://i.gyazo.com/cf57dde74d39274bfe6fe328e1f3c53a.png" alt="Image from Gyazo" width="800" border=1/></a>  

<br><br>

## Issue 3.4 Billsのshowビューを完成させる
---

indexの基本的な内容をそのまま使いつつ、下段部分の精算日や説明に関するテーブルを追加する。  
indexとshow、その他の今後作るnewやeditの画面も、比較できるよう併せて掲載する。  

<a href="https://gyazo.com/77296b35890fccb1569b129ba12b27e0"><img src="https://i.gyazo.com/77296b35890fccb1569b129ba12b27e0.png" alt="Image from Gyazo" width="226"/></a>  

<a href="https://gyazo.com/afc0b6d728b586f2816ab6946ddbb72e"><img src="https://i.gyazo.com/afc0b6d728b586f2816ab6946ddbb72e.png" alt="Image from Gyazo" width="449"/></a>  


なお、画像を添付できるようにするには、Active Storage の機能を使う必要があり、  
多少の手間がかかるので、とりあえずその部分は作らないこととする。  

<br>

### 精算日カラムの漏れ
---

ここで、カラムの漏れに気付く。  
精算日カラムがない。  

よって、下記のとおりマイグレーションファイルを作成し、修正する。  
なお、立替日を示すdateのカラム名は紛らわしいので、カラム名を変更する。  

まず、カラムを追加するマイグレーションファイルを作成する。  
なお、精算日カラム名は、completed_onとする。

```
$ bin/rails generate migration AddColumnToBills completed_on:date
```

以下のとおり、マイグレーションファイルが生成される。  
生成された後、マイグレートする。  

```rb
class AddColumnToBills < ActiveRecord::Migration[6.0]
  def change
    add_column :bills, :completed_on, :date
  end
end
```

続いて、立替日カラム名をdateからpaid_onに変更する。  

```
$ bin/rails generate migration rename_date_column_to_bills
```

以下のとおり、マイグレーションファイルが生成される。  
生成された後、マイグレートする。  

```rb
class RenameDateColumnToBills < ActiveRecord::Migration[6.0]
  def change
    rename_column :bills, :date, :paid_on
  end
end
```

なお、作業にあたっては、以下を参考にした。  

[マイグレーションファイルの命名規則 \- Qiita](https://qiita.com/yasumasaabe/items/825e960bbac216424329)  
[Ruby on Rails カラムの追加と削除 \- Qiita](https://qiita.com/azusanakano/items/a2847e4e582b9a627e3a)  
[Rails カラム名変更方法 \- Qiita](https://qiita.com/libertyu/items/93acd8733e34b1d0a63c)  

<br>

### リンクを貼る
---

追加した項目名（先ほどの画像でいうところの「サッカーボール」）をクリックすると、  
その申請データの詳細が確認できるようにする。  

index.html.slimについて、下記のとおり修正する。  
なお、カラム名の変更があったので、該当箇所の修正も併せて行うこと。  

```slim
h1.mt-5.mb-5 申請一覧

h4.pt-5.pb-3 請求中

table.table.table-hover
  thead.thead-default
    tr
      / dateからpaid_onに修正する
      th= Bill.human_attribute_name(:paid_on)
      th= Bill.human_attribute_name(:name)
      th= Bill.human_attribute_name(:item)
      th= Bill.human_attribute_name(:price)
    tbody
      - @bills.each do |bill|
        tr
          / dateからpaid_onに修正する
          td= bill.paid_on
          / ヘルパーメソッドにより、billと記載することでshowアクションを呼び出してくれる
          td= link_to bill.name, bill
          td= bill.item
          td= bill.price

/ 以下の表についても同様に修正する

```

<br>

### show.html.slimを作る
---

リンクを貼れたが、リンク先ビューファイルがない状態である。  
ここで、show.html.slimを作成し、コードを書き込んでいく。  

なお、パーシャル化できる部分があるので、そこは_basictable.html.slimにまとめていく。 
まとめる内容は、以下のとおりとする。  

```slim
table.table.table-hover
  thead.thead-default
    tr
      th= Bill.human_attribute_name(:paid_on)
      th= Bill.human_attribute_name(:name)
      th= Bill.human_attribute_name(:item)
      th= Bill.human_attribute_name(:price)
    tbody
      - @bills.each do |bill|
        tr
          td= bill.paid_on
          td= link_to bill.name, bill
          td= bill.item
          td= bill.price
```

なお、以下のとおり、index.html.slim や、show.html.slim に記載する。  
これで、_basictable.html.slimを参照し、該当コードを持ってくることができる。

```slim
= render partial: 'basictable', locals: {bill: @bill}
```

show.html.slim のコードは、index.html.slim を参考にしつつ、以下のとおりとした。  

```slim
/ いずれコントローラもしくはモデルでの処理にて、status(:boolean)の値を参照して、
/ falseであれば「申請中」、trueであれば「精算済」と表示するよう設定する
/ 差し当たり、「申請中」という表示で固定しておく

h1.mt-5.mb-5 申請中

= render partial: 'basictable', locals: {bill: @bill}

table.table.table-hover.mt-5.mb-5
  tbody
    tr
      th= Bill.human_attribute_name(:completed_on)
      td= @bill.completed_on
    tr
      th= Bill.human_attribute_name(:description)
      td= @bill.description

= link_to '編集', edit_bill_path(@bill), class: 'btn btn-primary mr-3'
= link_to '削除', bill_path(@bill), method: :delete, class: 'btn btn-danger'
```

また、bills_controller.rbの設定が必要になる。  
以下のとおり、showアクションを定義した。  

```rb
# bills_controller.rb
# showアクションのみ記載

  def show
    @bills = Bill.all
    @bill = Bill.find(params[:id])
  end
```

結果、以下のとおり画面を作ることができた。  

<a href="https://gyazo.com/f7bc33bf291bb103c230f0cdd5f0941b"><img src="https://i.gyazo.com/f7bc33bf291bb103c230f0cdd5f0941b.png" alt="Image from Gyazo" width="700" border=1/></a>  

<br><br>

## Issue 3.5 Billsのeditビューを完成させる
---

さて、ここではレイアウトはshowと同一であっても、入力可能な状態にする必要がある。  
edit.html.slimのコードは、今までのものとは大きく異なってくる。  

Bootstrapの勉強はしたくなかったが、とはいえ、個人的にここでbootstrapのレイアウトを適用しないというのは、  
納得がいかなかったので、以下の記事などを参照しながら作業を進めた。  

まとめて書くとエラーの原因が分からないので、ちょっとずつコードを書き加えてはブラウザで確認するという作業を繰り返した。  
おかげで、無事作り上げることができた。ここはそれなりに満足である。  

とりあえず、form-groupで囲って、クラスをform-controlにするとイイ感じにまとめてくれることまで理解した。  
その他、細かい設定については、追々の課題ということにして、次に行く。  

[【Rails】form\_forの使い方をマスターしよう！ \| Pikawaka \- ピカ1わかりやすいプログラミング用語サイト](https://pikawaka.com/rails/form_for#f.date_select)  
[Bootstrapでフォームを利用する方法 \- Qiita](https://qiita.com/AquaMeria/items/fe3fd9ebeb4c7171da3b)  
[Forms \- Bootstrap 4\.1 日本語リファレンス](https://getbootstrap.jp/docs/4.1/components/forms/#form-groups)  

また、form_withの使い方についてだが、何度もお世話になっているこの記事をまた確認した。  

[【Rails】form\_with/form\_forについて【入門】 \- Qiita](https://qiita.com/snskOgata/items/44d32a06045e6a52d11c)  

```slim
/ edit.html.slim

/ いずれコントローラもしくはモデルでの処理にて、status(:boolean)の値を参照して、
/ falseであれば「申請中」、trueであれば「精算済」と表示するよう設定する
/ 差し当たり、「申請中」という表示で固定しておく

h1.mt-5.mb-5 申請中

= form_with model: @bill, local: true do |f|
  table.table.table-hover
    thead.thead-default
      tr
        th= Bill.human_attribute_name(:paid_on)
        th= Bill.human_attribute_name(:name)
        th= Bill.human_attribute_name(:item)
        th= Bill.human_attribute_name(:price)
    tbody
      tr
        td
          .form-group
            = f.datetime_field :paid_on, class: 'form-control'
        td
          .form-group
            = f.text_field :name, class: 'form-control'
        td
          .form-group
            = f.text_field :item, class: 'form-control' 
        td
          .form-group
            = f.number_field :price, class: 'form-control' 
  
  = f.submit nil, class: 'btn btn-primary mr-3'
```

あと、忘れずにコントローラにeditアクションを追加する。  

```rb
# bills_controller.rb
# editアクションの部分のみ記載

def edit
  @bill = Bill.find(params[:id])
end
```

以下が作成画面である。  

<a href="https://gyazo.com/56a4e11731ef928846c47d2850eb171a"><img src="https://i.gyazo.com/56a4e11731ef928846c47d2850eb171a.png" alt="Image from Gyazo" width="800" border=1/></a>  

余談であるが、画面ばっかり作っているので、正直Railsやっている感じがあまりしない。  
まあ、自分で決めたタスクの進め方がそうなっているので、仕方がないのだけれど。  
（通常であれば、ここでupdate機能の実装あたりをやるのだろうけど、なぜかnew作りにいきます笑）  

あと、気が付いた！  
全く脈絡ないが、indexとshowでそもそもパーシャル化しているのがおかしい。  

今のままだとデータが一つだけなので問題が生じていないが、複数のデータが記録されると、  
showのビューが明らかにおかしいことになる。Githubのissueに追加し、機会を見つけて修正しよう。

<br><br>

## Issue 3.6 Billsのnewビューを完成させる
---

あー、ついにきた。  

これは流石にedit作ったので、すぐ終わる案件だ。  
これこそ、パーシャル化できる案件のはず。  

form_with は、modelの中身を判断して、空だったら createアクションに飛ばしてくれる。  
空でなければupdateアクションに飛ばしてくれる。

だから、edit.html.slim の大半がパーシャル化できる！  

パーシャル化する部分は、以下のとおり、_form.html.slimに寄せた。

```slim
/ _form.html.slim

= form_with model: @bill, local: true do |f|
  table.table.table-hover
    thead.thead-default
      tr
        th= Bill.human_attribute_name(:paid_on)
        th= Bill.human_attribute_name(:name)
        th= Bill.human_attribute_name(:item)
        th= Bill.human_attribute_name(:price)
    tbody
      tr
        td
          .form-group
            = f.datetime_field :paid_on, class: 'form-control'
        td
          .form-group
            = f.text_field :name, class: 'form-control'
        td
          .form-group
            = f.text_field :item, class: 'form-control' 
        td
          .form-group
            = f.number_field :price, class: 'form-control' 
  
  = f.submit nil, class: 'btn btn-primary mr-3'
```

そしたら、new.html.slimはこんなにシンプルに書けた。  
これは快感！（夜中のテンションです）

```slim
/ new.html.slim

h1.mt-5.mb-5 精算申請

p.col-md-10.mb-5 以下のとおり部の費用を立て替えたので、返金をお願いします。

= render partial: 'form', locals: {bill: @bill}
```

次に、edit.html.slimも書き換える。  

```slim
/ edit.html.slim

/ いずれコントローラもしくはモデルでの処理にて、status(:boolean)の値を参照して、
/ falseであれば「申請中」、trueであれば「精算済」と表示するよう設定する
/ 差し当たり、「申請中」という表示で固定しておく

h1.mt-5.mb-5 申請中

= render partial: 'form', locals: {bill: @bill}
```

あと、忘れずにコントローラにnewアクションを追加する。  

```rb
# bills_controller.rb
# newアクションの部分のみ記載

  def new
    @bill = Bill.new
  end
```

以下が作成画面である。  
サクッと終えることができたので、嬉しい！  

<a href="https://gyazo.com/9f8d39dc410cda038a03516a8327f4a1"><img src="https://i.gyazo.com/9f8d39dc410cda038a03516a8327f4a1.png" alt="Image from Gyazo" width="800" border=1/></a>  

<br>

### パーシャル化の修正
---

なお、ここでパーシャル化がおかしい部分を修正する。  
index, show, edit, new のコードを確認する。  

すると、edit と new のみしかパーシャル化出来ず、index と show はパーシャル化すべきでないことが分かる。  
（共通部分の分量が中途半端であるため、パーシャル化のメリットが少ないと判断した）

よって、以下のようにコードを修正する。  

```slim
/ index.html.slim

h1.mt-5.mb-5 申請一覧

h4.pt-5.pb-3 請求中

table.table.table-hover
  thead.thead-default
    tr
      th= Bill.human_attribute_name(:paid_on)
      th= Bill.human_attribute_name(:name)
      th= Bill.human_attribute_name(:item)
      th= Bill.human_attribute_name(:price)
    tbody
      - @bills.each do |bill|
        tr
          td= bill.paid_on
          td= link_to bill.name, bill
          td= bill.item
          td= bill.price

h4.pt-5.pb-3 精算済

table.table.table-hover
  thead.thead-default
    tr
      th= Bill.human_attribute_name(:completed_on)
      th= Bill.human_attribute_name(:name)
      th= Bill.human_attribute_name(:item)
      th= Bill.human_attribute_name(:price)
    tbody
      - @bills.each do |bill|
        tr
          td= bill.completed_on
          td= bill.name
          td= bill.item
          td= bill.price
```

```slim
/ edit.html.slim

/ いずれコントローラもしくはモデルでの処理にて、status(:boolean)の値を参照して、
/ falseであれば「申請中」、trueであれば「精算済」と表示するよう設定する
/ 差し当たり、「申請中」という表示で固定しておく

h1.mt-5.mb-5 申請中

table.table.table-hover
  thead.thead-default
    tr
      th= Bill.human_attribute_name(:paid_on)
      th= Bill.human_attribute_name(:name)
      th= Bill.human_attribute_name(:item)
      th= Bill.human_attribute_name(:price)
  tbody
    tr
      td= @bill.paid_on
      td= @bill.name
      td= @bill.item
      td= @bill.price

table.table.table-hover.mt-5.mb-5
  tbody
    tr
      th= Bill.human_attribute_name(:completed_on)
      td= @bill.completed_on
    tr
      th= Bill.human_attribute_name(:description)
      td= @bill.description

= link_to '編集', edit_bill_path(@bill), class: 'btn btn-primary mr-3'
= link_to '削除', bill_path(@bill), method: :delete, class: 'btn btn-danger'
```

showアクションについても、無駄に @bills = Bill.all という処理が行われていたので削除した。  

<br>

### edit.html.slim と new の画面修正
---

作業していて気が付いた。  
画面で表示すべき内容に不足がある。  

詳細に関する記入欄がない。  
また、edit画面については、精算日に関する記入欄がない。  

なお、画像投稿等の欄については、後回しにする。  

##### 目標

<a href="https://gyazo.com/afc0b6d728b586f2816ab6946ddbb72e"><img src="https://i.gyazo.com/afc0b6d728b586f2816ab6946ddbb72e.png" alt="Image from Gyazo" width="600" border=1/></a>  

<br>

##### 現実

<a href="https://gyazo.com/9f8d39dc410cda038a03516a8327f4a1"><img src="https://i.gyazo.com/9f8d39dc410cda038a03516a8327f4a1.png" alt="Image from Gyazo" width="600" border=1/></a>

<br>

<a href="https://gyazo.com/56a4e11731ef928846c47d2850eb171a"><img src="https://i.gyazo.com/56a4e11731ef928846c47d2850eb171a.png" alt="Image from Gyazo" width="600" border=1/></a>  


目標に合わせるため、コードを以下のとおり修正する。  

newには「completed_on」の欄がないが、editにはあるのでパーシャル化ができないのではないかと考えていたが、  
アドバイスをもらい、無事にパーシャル化することができた。  

実施にあたっては、以下の記事を参考にした。  
[new\_record?で場合分け ❏Rails❏ \- Qiita](https://qiita.com/ITmanbow/items/3f0be316fedec174d545)  

```slim
/ _form.html.slim
/ unlessを使って、@billがある場合とない場合の分岐を作る

= form_with model: @bill, local: true do |f| 
  table.table.table-hover
    thead.thead-default
      tr
        th= Bill.human_attribute_name(:paid_on)
        th= Bill.human_attribute_name(:name)
        th= Bill.human_attribute_name(:item)
        th= Bill.human_attribute_name(:price)
    tbody
      tr
        td
          .form-group
            = f.datetime_field :paid_on, class: 'form-control'
        td
          .form-group
            = f.text_field :name, class: 'form-control'
        td
          .form-group
            = f.text_field :item, class: 'form-control' 
        td
          .form-group
            = f.number_field :price, class: 'form-control'

  table.table.table-hover.mt-5.mb-5
    tbody
      tr
        - unless @bill.new_record?
          th= Bill.human_attribute_name(:completed_on)
          td= @bill.completed_on
      tr
        th= Bill.human_attribute_name(:description)
        td
          .form-group
            = f.text_area :description, class: 'form-control', size: "30x10"

  = f.submit nil, class: 'btn btn-primary mr-3'
```

```slim
/ edit.html.slim

/ いずれコントローラもしくはモデルでの処理にて、status(:boolean)の値を参照して、
/ falseであれば「申請中」、trueであれば「精算済」と表示するよう設定する
/ 差し当たり、「申請中」という表示で固定しておく

h1.mt-5.mb-5 申請中

= render 'form', bill: @bill
```

```slim
/ new.html.slim

h1.mt-5.mb-5 精算申請

p.col-md-10.mb-5 以下のとおり部の費用を立て替えたので、返金をお願いします。

= render 'form', bill: @bill
```

画面は以下のとおりとなった。  

<br>

<a href="https://gyazo.com/0d329757757e0b9910e68da69324747b"><img src="https://i.gyazo.com/0d329757757e0b9910e68da69324747b.png" alt="Image from Gyazo" width="600" border=1/></a>  

<br>

<a href="https://gyazo.com/637bea40ee564599e7441ed110a2856d"><img src="https://i.gyazo.com/637bea40ee564599e7441ed110a2856d.png" alt="Image from Gyazo" width="600" border=1/></a>  

<br><br>

## Issue 3.7 日本語化
---

ここで、面倒になっていた日本語化に着手する。  
以下によると、ja.ymlファイルのダウンロードは不要らしい。  

[\[初学者\]Railsのi18nによる日本語化対応 \- Qiita](https://qiita.com/shimadama/items/7e5c3d75c9a9f51abdd5#5-jayml%E3%81%AB%E6%97%A5%E6%9C%AC%E8%AA%9E%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)  

よって、まずデフォルトの設定を日本に合わせる。  
config/application.rbに追記する。  

```rb
# config/application.rb

require_relative 'boot'
require 'rails/all'

Bundler.require(*Rails.groups)

module BuhiManager
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 6.0

    # Qiita記事を参考にして、日本語化する
    # https://qiita.com/shimadama/items/7e5c3d75c9a9f51abdd5
    config.time_zone = 'Tokyo'
    config.active_record.default_timezone = :local
    config.i18n.default_locale = :ja

    # i18nの複数ロケールファイルが読み込まれるようpathを通す
    config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}').to_s]
  end
end
```

これだけで、デフォルトのロケールファイルを「svenfuchs/rails-i18n」は確かに反映された。  
具体的には、Create Bill や Update Bill といったボタンが、「登録する」や「更新する」に変更されていることが確認できた。  

続いて、ja.ymlファイルを作成し、モデルの属性などを日本語化する。  

```yml
# locales/ja.yml

ja:
  activerecord:
    models:
      bill: 申請 

    # model毎に定義したいattributesを記述
    attributes:
      bill:
        status: 未精算・精算済
        name: 名前
        paid_on: 立替日
        item: 内容
        description: 詳細 
        price: 金額
        completed_on: 精算日
                
  # 全てのmodelで共通して使用するattributesを定義
  attributes:
    created_at: 申請日時
    updated_at: 更新日時
```

無事、反映された！  

また、金額の表記方法があまり気に食わない（コンマがないなど）ので、以下の記事を参考に設定する。  
[通貨のI18nは気をつけた方がいい \- Qiita](https://qiita.com/awakia/items/d29fdbf0cab0b27da48f)  

関数を使えば一発で終わるらしい。  
ということで、ざっと適用できるところには適用した。  

```slim
/ 以下は参考例
/ index.html.slim や edit.html.slim などに適用

td= number_to_currency bill.price
```

なお、フォームへの適用は難しそうなのでとりあえずスルーした。  
画面は以下のとおりとなった。参考例として、index.html.slimを貼付する。  

<br>

<a href="https://gyazo.com/9713ab47a5d5980cb537ee2858ff2ded"><img src="https://i.gyazo.com/9713ab47a5d5980cb537ee2858ff2ded.png" alt="Image from Gyazo" width="600"/></a>

<br><br>

## Issue 3.8 Createアクションの実装
---

さて、コントローラ及びモデルに処理の実装を行っていく。  

とはいっても、ビューと違って、コントローラに何を実装するか全く考えられていないので、  
まず何の処理が必要か洗い出すところから始める。

<br>

<dl>
<dt>indexアクション</dt>
<dd>Billモデルから全てのデータを引っ張り出し、インスタンス変数に格納（そして表示）</dd>
<dt>showアクション<dt>
<dd>Billモデルから該当のidのデータを引っ張り出し、インスタンス変数に格納（そして詳細表示）</dd>
<dt>editアクション</dt>
<dd>Billモデルから該当のidのデータを引っ張り出し、インスタンス変数に格納（そして編集表示）</dd>
<dt>newアクション</dt>
<dd>Billモデルの新しいオブジェクトを作成し、インスタンス変数に格納</dd>
<dt>createアクション</dt>
<dd>Billモデルの新しいオブジェクト（引数はparamas）を保存</dd>  
<dd>保存した後、index.html.slimに遷移し、フラッシュメッセージを表示</dd>  
<dd>保存できない場合、現在のページから移動せず、エラーメッセージを表示</dd>  
<dt>updateアクション</dt>
<dd>Billモデルの該当のidのデータ（引数はparamas）を更新</dd>  
<dd>更新した後、index.html.slimに遷移し、フラッシュメッセージを表示</dd>
<dd>更新できない場合、エラーメッセージを表示</dd>
<dt>destroyアクション</dt> 
<dd>Billモデルの該当のidのデータを削除。フラッシュメッセージを表示。</dd>
<dd>削除前に確認メッセージを表示</dd>
</dl>

<br>

さて、index・show・edit・newアクションは作成済であるため、残り３アクションを実装していく。  
まず、createアクションから実装する。  

```rb
# bills_controller.rb
# createアクションのみ記載

class BillsController < ApplicationController

  def create
    @bill = Bill.new(bill_params)

    if @bill.save
      redirect to @bill
    else
      render :new
    end
  end

  # updateやdestroyアクションは省略

  private

  def bill_params
    params.require(:bill).permit(:status, :name, :paid_on, :item, :description, :price, :completed_on)
  end

end
```

よし、現場railsの本からほぼパクってきているし、ミスのしようもないだろう！  
とりあえず何のデータも入力せずに登録してみて、同じページがレンダーされるか確認する。  

・・・

<a href="https://gyazo.com/60fc52fadc1044b3aa126398e036e473"><img src="https://i.gyazo.com/60fc52fadc1044b3aa126398e036e473.png" alt="Image from Gyazo" width="600"/></a>  

え、エラー！？  

<br>

### NoMethodError undefined method `date' の解決
---

けど、ここでRubyのインスタンスやらクラスの勉強が役に立つ。  

NoMethodErrorということは、当然だがBillクラスのインスタンス変数である@billにメソッドがないということ。  
そして、定義されていないdateメソッドがあるとのこと。  

これが分かるだけでも、非常に精神衛生的に良い。  
さて、伊藤大先生のエラーが起きた時のQiitaを見る。  

[プログラミング初心者歓迎！「エラーが出ました。どうすればいいですか？」から卒業するための基本と極意（解説動画付き） \- Qiita](https://qiita.com/jnchito/items/056325421b7e36f02335)  

なるほど。  

伊藤先生、きちんとエラーメッセージは解読しました。エラーが起きている場所も特定しています。  
@billをセーブしようとしている時に、dateメソッドがないということですよね！  

ここで予測を立てる。  
もしかして、値がnilだから問題が起きているのではないか。  

以下にもそれらしきことが書いてある！  

[Rubyのエラーメッセージundefined methodの解決方法【初心者向け】 \| TechAcademyマガジン](https://techacademy.jp/magazine/19776)  

・・・rails console を使って、調べてみた！  
どうやらdateメソッドはたしかにない！  

ただし！！！  
日付の値があっても、そのインスタンスにdateメソッドがない！！！  

<br>

### dateメソッドってあるの？
---

え、というかdateメソッドってそもそもあるのか。。。  

Rubyのレファレンスマニュアルを確認してみる。  
[オブジェクト指向スクリプト言語 Ruby リファレンスマニュアル \(Ruby 2\.7\.0 リファレンスマニュアル\)](https://docs.ruby-lang.org/ja/latest/doc/index.html)  

うん、dateメソッドない。。。  
じゃあ、どういうことだ。。。  

なぜ、こっちが勝手にdateメソッドなんて作っていないにもかかわらず、undefined method `date' と出てくるんだ？？？  

え、もしかして。。。  

```rb
# bill.rb
# Billモデルのファイル

class Bill < ApplicationRecord
  validates :status, inclusion: {in: [true, false]}
  validates :name, presence: true, uniqueness: true
  # ここに犯人がいた！ dateって書いてある
  validates :date, presence: true
  validates :item, presence: true
  validates :price, numericality: { greater_than: 0 }
end
```

カラム名は修正したけど、バリデーションは修正し忘れていたというオチでした。  

バリデーションがめちゃくちゃなので、以下のとおり書き換える。  

```rb
# bill.rb
# Billモデルのファイル

class Bill < ApplicationRecord
  validates :status, inclusion: {in: [true, false]}
  # nameカラムにuniquenessは不要。同一人物が１回しか申請できないのでは困る。
  validates :name, presence: true
  # dateカラムはもう削除したので、paid_onに変更する。
  validates :paid_on, presence: true
  validates :item, presence: true
  validates :price, numericality: { greater_than: 0 }
end
```

<br>

### Flashメッセージを追加し、createアクションが機能するか確認する
---

さて、動作確認を行う。  
ここで、createアクションの挙動が分かりやすいよう、以下のとおり設定する。  

- コントローラにflashメッセージを格納させる設定を追加
- redirect先のshow.html.slimにてフラッシュメッセージが表示されるコードを追加  

```rb
# bills_controller.rb
# createアクションの該当部分のみ記載

  def create
    @bill = Bill.new(bill_params)

    if @bill.save
      redirect to @bill, notice: "#{@bill.item}（#{@bill.price}円）について、精算申請を提出しました。"
    else
      render :new
    end
  end

  # updateやdestroyアクションは省略
  # privateメソッドのbill_paramsアクションも省略
end
```

```slim
/ show.html.slim

/ いずれコントローラもしくはモデルでの処理にて、status(:boolean)の値を参照して、
/ falseであれば「申請中」、trueであれば「精算済」と表示するよう設定する
/ 差し当たり、「申請中」という表示で固定しておく

h1.mt-5.mb-5 申請中

- if flash.notice.present?
  .alert.alert-success.mt-5.mb-5= flash.notice

  〜 以下、省略 〜
```

<br>

new.html.slimの画面から登録すると、以下のとおり表示されるようになった。  
めでたし、めでたし。  

<a href="https://gyazo.com/31d2e0bbca2eec2905b695bd3db183f2"><img src="https://i.gyazo.com/31d2e0bbca2eec2905b695bd3db183f2.png" alt="Image from Gyazo" width="600" border=1/></a>  

また、エラーメッセージが出るよう、以下のコードを_form.html.slimに追加した。  

```slim
/_form.html.slim
/ 同ファイルの一番上に以下のコードを記載

- if bill.errors.present?
  ul#error_explanation
    - bill.errors.full_messages.each do |message|
      li= message

〜 以下、省略 〜
```

すると、エラーメッセージが表示されるようになった。  

<a href="https://gyazo.com/9629bf77549c0238e86fbb95dcd14ec1"><img src="https://i.gyazo.com/9629bf77549c0238e86fbb95dcd14ec1.png" alt="Image from Gyazo" width="600" border=1/></a>  

<br><br>

## Issue 3.9 Updateアクションの実装
---

