# Rails6系で「everyday Rails」（RSpec）を勉強したいので、伊藤さんの記事を参考にして頑張ってみた（part3）

## はじめに

この記事は、以下のQiita記事の続編です。
> Rails6系で「everyday Rails」（RSpec）を勉強したいので、伊藤さんの記事を参考にして頑張ってみた（part1）
> Rails6系で「everyday Rails」（RSpec）を勉強したいので、伊藤さんの記事を参考にして頑張ってみた（part2）

伊藤さんがアップロードされたyoutube動画を見ながら、私個人が
どのように対応していったか記しているだけなので、動画を見れば大体事足ります。  

> [【動画付き】Everyday RailsのサンプルアプリをRails 6で動かす際に必要なテストコードの変更点 \- give IT a try](https://blog.jnito.com/entry/2019/10/25/053521)
> [永久保存版！？伊藤さん式・Railsアプリのアップグレード手順 \- Qiita](https://qiita.com/jnchito/items/0ee47108972a0e302caf)  
> [【前編】永久保存版！？伊藤さん式・Railsアプリのアップグレード手順](https://youtube.com/watch?v=hT68fhuWbHU)
> [【後編】永久保存版！？伊藤さん式・Railsアプリのアップグレード手順](https://youtube.com/watch?v=SnwNFMauzjM)

なので、あまりこの記事の意義はないのですが、文字の方が探しやすい  
こともあるかと思うので、参考までに活用いただければと思います。

## Railsのメジャーバージョンを上げる

さて、Railsの`Gemfile`を開き、Railsのバージョンを6.00に上げる。  
現在では、6.0.3.1が最新版だが、怖いので、とりあえず6.00に上げる。  

```
gem 'rails', '~> 6.0.0'
```

`Gemfile`を変更した後、`bundle update`を行う。 

## rails app:updateを実行する

ターミナルで`rails app:update`を実行する。

前回と同様に、routes.rbだけ上書きをせず、`n`と回答する。  
そのほかは、`y`と回答する。


## 必要に応じて上書きされた設定を元に戻す

> Yで上書き実行すると、それまで使っていた重要な設定が失わることがあります。
> その場合はgitのdiffをチェックしながら、上書きされて消えてしまった設定を自力で戻していきます。

Rails5.2のバージョンアップの時と同様に対応する。  
以下のファイルを変更した。

```rb
# config/application.rb

〜 省略 〜

module Projects
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 5.2

    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration can go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded after loading
    # the framework and any gems in your application.

    # 以下が消えていたので、戻した。
    config.generators do |g|
      g.test_framework :rspec,
        view_specs: false,
        helper_specs: false,
        routing_specs: false
    end
  end
end
```

```rb
# config/environments/test.rb
# 以下のコードを最後に追加する

# Keep files uploaded in tests from polluting the Rails development
# environment's file uploads
Paperclip::Attachment.default_options[:path] = \
  "#{Rails.root}/spec/test_uploads/:class/:id_partition/:style.:extension"
```

## railsdiff.orgを参考にして、新しく追加されたgem等を確認する

次に以下を開き、差分を更新していく。  

このあたりも、アプリのことをよく分かっており、かつ勘所がないと困難なので、  
伊藤さんの動画の指示に従っていく。  

[RailsDiff](http://railsdiff.org/5.2.3/6.0.3.1)  

以下のファイルを変更した。  

```
# Gemfile

source 'https://rubygems.org'

git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.1'

gem 'rails', '~> 6.0.0'
gem 'sqlite3', '~> 1.4' # 更新
gem 'puma', '~> 3.11'
gem 'sass-rails', '~> 5' # 更新
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.2'
gem 'turbolinks', '~> 5'
gem 'jbuilder', '~> 2.7' # 更新
gem 'bootsnap', '>= 1.4.2', require: false # 更新

〜 以下省略 〜
```

```rb
# config/application.rb

module Projects
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 6.0 # ここを6.0に変更

    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration can go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded after loading
    # the framework and any gems in your application.

    config.generators do |g|
      g.test_framework :rspec,
        view_specs: false,
        helper_specs: false,
        routing_specs: false
    end
  end
end
```

`Gemfile`を反映させるため、`bundle update`を実行する。  
成功した。  

## 動作確認を行う

`rails c`については、成功した。  

`rails s`については、動画のとおり`rails db:migrate`を実行すると、  
無事起動することができた。挙動も問題なさそうだった。  

`bin/rspec`を実行し、テストに失敗がないか確認する。  
以下のFailuresが発生した。  

```
Failures:

  1) TasksController#show responds with JSON formatted output
     Failure/Error: expect(response).to have_content_type :json
       Expected "unknown content type (application/json; charset=utf-8)" to be Content Type "application/json" (json)
     # ./spec/controllers/tasks_controller_spec.rb:11:in `block (3 levels) in <main>'
     # /Users/kentasuedomi/.rbenv/gems/2.7.0/gems/bootsnap-1.4.6/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:55:in `load'
     # /Users/kentasuedomi/.rbenv/gems/2.7.0/gems/bootsnap-1.4.6/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:55:in `load'
     # /Users/kentasuedomi/.rbenv/gems/2.7.0/gems/spring-commands-rspec-1.0.4/lib/spring/commands/rspec.rb:18:in `call'
     # -e:1:in `<main>'

  2) TasksController#create responds with JSON formatted output
     Failure/Error: expect(response).to have_content_type :json
       Expected "unknown content type (application/json; charset=utf-8)" to be Content Type "application/json" (json)
     # ./spec/controllers/tasks_controller_spec.rb:21:in `block (3 levels) in <main>'
     # /Users/kentasuedomi/.rbenv/gems/2.7.0/gems/bootsnap-1.4.6/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:55:in `load'
     # /Users/kentasuedomi/.rbenv/gems/2.7.0/gems/bootsnap-1.4.6/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:55:in `load'
     # /Users/kentasuedomi/.rbenv/gems/2.7.0/gems/spring-commands-rspec-1.0.4/lib/spring/commands/rspec.rb:18:in `call'
     # -e:1:in `<main>'

Finished in 56.81 seconds (files took 2.96 seconds to load)
70 examples, 2 failures
```

動画によると、RSpecの書き方が変わったらしい。
`expect`で始まるコードを修正する。  

```rb
# spec/controllers/tasks_controllers/tasks_controller_spec.rb

require 'rails_helper'

RSpec.describe TasksController, type: :controller do
  include_context "project setup"

  describe "#show" do
    it "responds with JSON formatted output" do
      sign_in user
      get :show, format: :json,
        params: { project_id: project.id, id: task.id }
      expect(response.content_type).to eq "application/json; charset=utf-8" # ここを修正
    end
  end

  describe "#create" do
    it "responds with JSON formatted output" do
      new_task = { name: "New test task" }
      sign_in user
      post :create, format: :json,
        params: { project_id: project.id, task: new_task }
      expect(response.content_type).to eq "application/json; charset=utf-8" # ここを修正
    end

    it "adds a new task to the project" do
      new_task = { name: "New test task" }
      sign_in user
      expect {
        post :create, format: :json,
          params: { project_id: project.id, task: new_task }
      }.to change(project.tasks, :count).by(1)
    end

    it "requires authentication" do
      new_task = { name: "New test task" }
      # Don't sign in this time ...
      expect {
        post :create, format: :json,
          params: { project_id: project.id, task: new_task }
      }.to_not change(project.tasks, :count)
      expect(response.content_type).to eq "application/json; charset=utf-8" # ここを修正
    end
  end
end
```

`bin/rspec`を実行し、テストに失敗がないか確認したところ、無事成功した。  

ただし、動画同様に警告が出ているので、対応していくことにする。  

```
DEPRECATION WARNING: update_attributes is deprecated and will be removed from Rails 6.1 (please, use update instead) (called from toggle at /Users/HOGE/Desktop/Github作業フォルダ/RSpecPractice/everydayrails-rspec-2017-master/app/controllers/tasks_controller.rb:67)
```

`update_attributes`を`update`に修正する。  
以下のファイルを修正する。  

- app/controllers/tasks_controller.rb
- app/controllers/projects_controller.rb
- spec/controllers/projects_controller.rb

さて、次に以下の警告を対応する。  

```
DEPRECATION WARNING: Single arity template handlers are deprecated. Template handlers must
now accept two parameters, the view object and the source for the view object.
Change:
  >> Coffee::Rails::TemplateHandler.call(template)
To:
  >> Coffee::Rails::TemplateHandler.call(template, source)
 (called from <main> at /Users/kentasuedomi/Desktop/Github作業フォルダ/RSpecPractice/everydayrails-rspec-2017-master/Rakefile:6)
```

動画で紹介されてないのでどうしたものかと思っていたところ、
以下の記事を発見！ ありがたい！！！

[Rails 6 を動かす際に “DEPRECATION WARNING: Single arity template handlers are deprecated\.” という警告が出た場合の対処 \| Lonely Mobiler](https://loumo.jp/archives/23121)  

結論を読むと、cofee-railsのバージョンが問題らしい。  
Gemfileを以下のとおり設定し、`bundle update`を行う。  

```
gem 'coffee-rails', '~> 5.0.0'
```

警告が消えた！

ついに終了！！！

