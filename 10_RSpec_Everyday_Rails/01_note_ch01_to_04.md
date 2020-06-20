# EverydayRails RSpecによるRailsテスト入門

## Chapter 01 Introduction

- 本書の第一のゴールは私の役に立っている一貫した戦略をあなたに伝えること
- 本書の第二のゴールは日常的によく使うRSpecの機能と構文をあなたが使いこなせるように手助

私が考えるテストの原則

- テストは信頼できるものであること
- テストは簡単に書けること
- テストは簡単に理解できること(今日も将来も)

## Chapter 02 RSpecのセットアップ

### 大まかな流れ

- まず`Bundler`を使って、RSpec をインストールするところから始めます。
- 必要に応じてテスト用データベースの確認とインストールを行います。
- 次にテストしたい項目をテストできるように RSpec を設定します。
- 最後に、新しい機能を追加するときにテスト用のファイルを自動生成できるよう、Rails アプリケー
ションを設定します

### 事前に行うべき設定

1. `gem rspec-rails`をインストールする
2. `config/database.yml`を開き、テストデータベースのアクセス先が設定されているか確認する
3. `rails db:create`を実行し、接続可能なデータベースを作成する

なお、`gem rspec-rails`には、RSpec + RSpecをRailsで使うための機能が入っている

### RSpecのインストール

まず、`bin/rails generate rspec: install`を実行する。

```
create .rspec # 設定ファイル
create spec # スペックファイルを格納するディレクトリ
create spec/spec_helper.rb # ヘルパーファイル
create spec/rails_helper.rb # ヘルパーファイル
```

ドキュメント形式に変更するには、以下の作業を行う。
出力が仕様書のようになる。

```text:.rspec
--require spec_helper
--format documentation
```

なお、調べたら他のフォーマットもあった。
見栄えが良さそうなので、htmlを試してみる。

<a href="https://gyazo.com/291b467e6fc433715641f51f81016512"><img src="https://i.gyazo.com/291b467e6fc433715641f51f81016512.png" alt="Image from Gyazo" width="600" border=1/></a>

次に`rspec binstub`をインストールする。
（アプリケーションの bin ディレクトリ内に rspec という名前の実行用ファイルが作成される。）

```text:Gemfile
group :development do
  # 省略
  gem 'spring-commands-rspec'
end
```

以下を実行するとよい。
これにより、アプリケーションの起動を早くする`Spring`の恩恵が受けられる。

```
bundle exec spring binstub rspec
```

ジェネレータも設定する。
`rails generate`コマンドを使ってアプリケーションにコードを追加する際に、
RSpec 用のスペックファイルも一緒に作ってもら得るようになる。

```rb:config/application.rb
module CookingPost
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 5.2

    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration can go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded after loading
    # the framework and any gems in your application.
    config.i18n.default_locale = :ja
    config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}').to_s]

    # 以下に追加する
    config.generators do |g| g.test_framework :rspec,
      view_specs: false,
      helper_specs: false,
      routing_specs: false
    end

  end
end
```

具体的には、`config/application.rb`を開き、
`config.generators do |g|`で始まるコードを書く。  

- fixtures
- view_specs
- helper_specs
- routing_specs

これをfalseにすれば、`rails generate`された際、
該当ファイルの作成を省略してくれる。

以下は、個人的なメモ。
chromedriver-helperはwebdriversに移行したので、その警告などが出ることがある。

```
-  gem 'chromedriver-helper'
+  gem 'webdrivers', '~> 3.0'
```

以下の部分は、参考になったのでそのまま引用する。

### なぜビューはテストしないのですか?

> 信頼性の高いビューのテストを作ることは非常に面倒だからです。
> さらにメンテナンスしようと思ったらもっと大変になります。
> ジェネレータを設定する際に私が述べたように、
> UI関連のテストは統合テストに任せようとしています。
> これは Rails 開発者 の中では標準的なプラクティスです。

## Chapter 03 モデルスペック

### モデルスペックの作成

```
bin/rails g rspec:model モデル名
```

### モデルスペックには次のようなテストを含める

- 有効な属性で初期化された場合は、モデルの状態が有効(valid)になっていること
- バリデーションを失敗させるデータであれば、モデルの状態が有効になっていないこと
- クラスメソッドとインスタンスメソッドが期待通りに動作すること

### モデルスペックで骨組みを作る

以下のとおり、テストコード自体はまだ書かなくていいので、
何のテストすべきか箇条書きにする。

```rb
# 姓、名、メール、パスワードがあれば有効な状態であること
it "is valid with a first name, last name, email, and password"
```

### 注意点

- 各 example の説明は動詞で始めること（可読性）
- example(itで始まる1行)一つにつき、結果を一つだけ期待すること
  - テストに失敗した時、原因の特定が簡単になる

### 試したマッチャ

- be_valid 、eq 、include 、be_empty
- 詳細はここで紹介されている
  - [使えるRSpec入門・その2「使用頻度の高いマッチャを使いこなす」 \- Qiita](https://qiita.com/jnchito/items/2e79a1abe7cd8214caa5)

### エラーメッセージについて

モデルのバリデーションに抵触するか確認したい場合、  
エラーメッセージがそもそもどうなっているか知る必要がある。  

以下を参照するとよい。  

- [rails\-i18n/ja\.yml at master · svenfuchs/rails\-i18n](https://github.com/svenfuchs/rails-i18n/blob/master/rails/locale/ja.yml)
- アプリ内の`ja.yml`ファイル

### DRYに書く

以下を使って、共通化したり、整理できる。

- describe
- context
- before
- after

## Chapter 04 意味のあるテストデータの作成

テストデータは、`Factory Bot`を使って簡単に作成することができる。

• 他の方法と比較した場合のファクトリの利点と欠点について説明します。
• それから基本的なファクトリを作り、既存のスペックで使ってみます。
• 続いてファクトリを編集し、さらに便利で使いやすくします。
• さらに Active Record の関連を再現する、より高度なファクトリを見ていきます。 • 最後に、ファクトリを使いすぎるリスクについて説明します。

### ファクトリ対フィクスチャ
- Railsにはサンプルデータを生成する手段として、フィクスチャがデフォルトで備わっている。
- テストとは別のフィクスチャファイルに保存された値を覚えておく必要がある。
- フィクスチャはもろくて壊れやすい
- Railsはフィクスチャのデータをデータベースに読み込む際に Active Recordを使わない
- ファクトリには、これらのデメリットがない
- ファクトリを多用すると、テストの起動が遅くなるかもしれない

### Factory Bot をインストールする

使う`gem`は、`factory_bot_rails`である。

自動的にファクトリを作成するように Rails を設定できる。

```rb:config/application.rb
config.generators do |g| g.test_framework :rspec,
  view_specs: false,
  helper_specs: false,
  routing_specs: false,
end
```

### アプリケーションにファクトリを追加する

```
bin/rails g factory_bot:model user
```

これで、ファクトリのファイルが作成できる。
そして、以下のように設定すれば、`FactoryBot.create(:user)と書くだけで、
簡単に新しいユーザーを作成できる。

```rb:spec/factories/users.rb
FactoryBot.define do
  factory :user do
    first_name { "Aaron" }
    last_name { "Sumner" }
    email { "tester@example.com" }
    password { "dottle-nouveau-pavilion-tights-furze" }
  end
end
```

ファクトリでは、整数・ブーリアン・日付など、attributesに渡せるものなら何でも渡せる。
こんな感じで。なお、buildの場合、新しいユーザーがインスタンス化されるだけで、保存されない。

```rb
it "has a valid factory" do
  expect(FactoryBot.build(:user)).to be_valid
end
```

また、ファクトリで作られたデータの属性はオーバーライドできる。
こんな感じで、メールアドレスだけ`nil`にして、ファクトリを利用できる。

```rb
it "is invalid without an email address" do
  user = FactoryBot.build(:user, email: nil)
  user.valid?
  expect(user.errors[:email]).to include("can't be blank")
end
```

生成されるオブジェクトの属性が同一であると（例えばメールアドレス）、
モデルのバリデーションに抵触して困ってしまうことがある。

その場合、シーケンスを使って、このようにコードを書くとよい。

```rb
FactoryBot.define do 
  factory :user do
    first_name: { "Aaron" }
    last_name: { "Sumner" }
    sequence(:email) { |n| "tester#{n}@example.com" }
    password: { "dottle-nouveau-pavilion-tights-furze" }
  end
end
```

### ファクトリで関連を扱う

- Factory Bot は他のモデルと関連を持つモデルを扱うのに便利

```
bin/rails g factory_bot:model note
```

```rb
FactoryBot.define do
  factory :note do
    message { "My important note."}
    association :project
    association :user
  end
end
```

以上のとおり、`association`を指定することにより、ProjectとUserにnoteを紐付けることができる。  
ただ、こう書くと以下のような結果になってしまう。

```
Note
This note's project is #<Project id: 1, name: "Project 1", description: "A test project.", due_on: "2020-06-26", created_at: "2020-06-19 07:20:43", updated_at: "2020-06-19 07:20:43", user_id: 1, completed: nil>

This note's user is #<User id: 2, email: "tester2@example.com", created_at: "2020-06-19 07:20:43", updated_at: "2020-06-19 07:20:43", first_name: "Aaron", last_name: "Sumner", authentication_token: "i3y4eB9aYMCgSUeawT7Z", location: nil>

# 本当は、同一ユーザーのnoteについてテストしたいのに、新しくユーザーを作ってしまう。。。
```

解説として、以下のとおり説明があった。

> この理由はメモのファクトリが関連するプロジェクトを作成する際に関連するユーザー
> (プロジェクトに関連するowner)を作成し、それから2番目のユーザー(メモに関連するユーザー)
> を作成するからです。

一読して分かる人がいるのだろうか。。。

ゆっくり紐解くと、分かった気がする。
あくまで概念的な感じでまとめているだけだが、こんな感じだと思われる。

```rb
FactoryBot.define do
  factory :note do
    message { "My important note."}
    association :project
      ### ここで projects の FactoryBot が呼ばれる

      # factory :project do
        # sequence(:name) { |n| "Project #{n}" }
        # description { "A test project." }
        # due_on { 1.week.from_now }
        # association :owner
          ### associationにより、User id: 1 のユーザーが作成される
      # end

    association :user
      ### ここで user の FactoryBot が呼ばれる
      ### ここで、User id: 2のユーザーが作成される

      # factory :user, aliases: [:owner] do
      #   first_name { "Aaron" }
      #   last_name  { "Sumner" }
      #   sequence(:email) { |n| "tester#{n}@example.com" }
      #   password { "dottle-nouveau-pavilion-tights-furze" }
      # end

  end
end
```

正解は、以下のとおり。

```rb: spec/factories/notes.rb
FactoryBot.define do
  factory :note do
    message { "My important note." }
    association :project
    user { project.owner }
    # association :user
  end
end
```

なお、Projectモデルにて、`belongs_to`で ownerに紐づいているので、`alias`で指定する必要があるとのこと。  

### ファクトリ内の重複をなくす

二つのテクニックがある。

1. 継承
2. Trait

#### 継承

ネストするような形で書く。
該当部分だけ上書きするように書く。

```rb: spec/factories/projects.rb
FactoryBot.define do 
  factory :project do
    sequence(:name) { |n| "Test Project #{n}" } 
    description "Sample project for testing purposes" 
    due_on 1.week.from_now
    association :owner

    # 昨日が締め切りのプロジェクト 
    factory :project_due_yesterday do
      due_on 1.day.ago 
    end

    # 今日が締め切りのプロジェクト
    factory :project_due_today do
      due_on Date.current.in_time_zone
    end

    # 明日が締め切りのプロジェクト
    factory :project_due_tomorrow do
      due_on 1.day.from_now 
    end
  end
end
```

なお、継承しない場合、以下のとおり`Class`の指定が必要。

```rb: spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Test Project #{n}" }
    description "Sample project for testing purposes"
    due_on 1.week.from_now
    association :owner

    # 締め切りが昨日
  factory :project_due_yesterday, class: Project do
    sequence(:name) { |n| "Test Project #{n}" } 
    description "Sample project for testing purposes" 
    due_on 1.day.ago
    association :owner
  end
end
```

### Trait

```rb: spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Test Project #{n}" }
    description "Sample project for testing purposes"
    due_on 1.week.from_now
    association :owner

    # 締め切りが昨日
    trait :due_yesterday do
      due_on 1.day.ago 
    end

    # 締め切りが今日
    trait :due_today do
      due_on Date.current.in_time_zone
    end

    # 締め切りが明日
     trait :due_tomorrow do
      due_on 1.day.from_now 
    end
  end
end
```

なお、Traitの場合、引数は以下のとおり書く。

```rb: spec/factories/projects.rb
describe "late status" do
  # 締切日が過ぎていれば遅延していること
  it "is late when the due date is past today" do
    project = FactoryBot.create(:project, :due_yesterday)
    expect(project).to be_late
  end
  〜 省略 〜
end
```

### コールバック

- 適切にコールバックを使えば複雑なテストシナリオも簡単にセットアップできる
- 一方でコールバックは遅いテストや無駄に複雑なテストの原因になることもある

コールバックは、例えば`traits`と組み合わせるといい感じになる。

```rb: spec/models/project_spec.rb
it "can have many notes" do
  project = FactoryBot.create(:project, :with_notes)
  expect(project.notes.length).to eq 5
end  
```

```rb: spec/models/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Test Project #{n}" }
    description "Sample project for testing purposes"
    due_on 1.week.from_now
    association :owner

# メモ付きのプロジェクト
    trait :with_notes do
      after(:create) { |project| create_list(:note, 5, project: project) }
    end
  end
end
```

ここでは、まず project ファクトリが`create`される。
続いて、`trait`が発動し、そのプロジェクトに紐づくnotesが5つ生成される。
そして、テストが実行される。

### ファクトリを安全に使うには

ファクトリを使うとテスト中に予期しないデータが作成されたり、
無駄にテストが遅くなったりする原因になる。

上記のような問題がテストで発生した場合はまず、ファクトリが必要なことだけを行い、
それ以上のことをやっていないことを確認すること。

