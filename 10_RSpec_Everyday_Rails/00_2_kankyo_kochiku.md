# Rails6系で「everyday Rails」（RSpec）を勉強したいので、伊藤さんの記事を参考にして頑張ってみた（part2）

## はじめに

この記事は、以下のQiita記事の続編です。
> Rails6系で「everyday Rails」（RSpec）を勉強したいので、伊藤さんの記事を参考にして頑張ってみた（part2）

伊藤さんがアップロードされたyoutube動画を見ながら、私個人が
どのように対応していったか記しているだけなので、動画を見れば大体事足ります。  

> [【動画付き】Everyday RailsのサンプルアプリをRails 6で動かす際に必要なテストコードの変更点 \- give IT a try](https://blog.jnito.com/entry/2019/10/25/053521)
> [永久保存版！？伊藤さん式・Railsアプリのアップグレード手順 \- Qiita](https://qiita.com/jnchito/items/0ee47108972a0e302caf)  
> [【前編】永久保存版！？伊藤さん式・Railsアプリのアップグレード手順](https://youtube.com/watch?v=hT68fhuWbHU)
> [【後編】永久保存版！？伊藤さん式・Railsアプリのアップグレード手順](https://youtube.com/watch?v=SnwNFMauzjM)

なので、あまりこの記事の意義はないのですが、文字の方が探しやすい  
こともあるかと思うので、参考までに活用いただければと思います。

## 4-c. トラブルが起きやすそうなgemを1つずつアップデートする

伊藤さんによると、次は`bundle outdated | bof --format markdown`の実行によって得られた、
defaultと書いてある`gem`を見ていくべきとのことだが、そもそもdefaultと書かれているgemが存在しない。  

参考までに、伊藤さんの動画のスクショと、私が得られたdefaultと書かれていないgemの一覧を貼付しておく。  
勝手な推測だが、これも伊藤さんが動画を撮った時期と、私がアップデートしている時期が異なるためだと思われる。  

<a href="https://gyazo.com/bafa6a3ee68e7ed93ed0dcb3e0796f54"><img src="https://i.gyazo.com/bafa6a3ee68e7ed93ed0dcb3e0796f54.png" alt="Image from Gyazo" width="500" border=1/></a>  


| gem | newest | installed | requested | groups |
| --- | --- | --- | --- | --- |
| actioncable | 6.0.3.1 | 5.1.7 | | |
| actionmailer | 6.0.3.1 | 5.1.7 | | |
| actionpack | 6.0.3.1 | 5.1.7 | | |
| actionview | 6.0.3.1 | 5.1.7 | | |
| activejob | 6.0.3.1 | 5.1.7 | | |
| activemodel | 6.0.3.1 | 5.1.7 | | |
| activerecord | 6.0.3.1 | 5.1.7 | | |
| activesupport | 6.0.3.1 | 5.1.7 | | |
| arel | 9.0.0 | 8.0.0 | | |
| autoprefixer-rails | 9.7.6 | 6.7.7.2 | | |
| bcrypt | 3.1.13 | 3.1.12 | | |
| bindex | 0.8.1 | 0.5.0 | | |
| bootstrap-sass | 3.4.1 | 3.3.7 | | |
| cocaine | 0.6.0 | 0.5.8 | | |
| coffee-rails | 5.0.0 | 4.2.1 | ~> 4.2 | |
| devise | 4.7.2 | 4.4.3 | | |
| faker | 2.12.0 | 1.7.3 | | |
| geocoder | 1.6.3 | 1.4.9 | | |
| i18n | 1.8.3 | 0.9.5 | | |
| jbuilder | 2.10.0 | 2.6.3 | ~> 2.5 | |
| jquery-rails | 4.4.0 | 4.3.1 | | |
| listen | 3.2.1 | 3.1.5 | < 3.2, >= 3.0.5 | |
| mimemagic | 0.3.5 | 0.3.2 | | |
| mini_portile2 | 2.5.0 | 2.4.0 | | |
| multi_json | 1.14.1 | 1.12.1 | | |
| paperclip | 6.1.0 | 5.1.0 | | |
| puma | 4.3.5 | 3.8.2 | ~> 3.7 | |
| rails | 6.0.3.1 | 5.1.7 | ~> 5.1.1 | |
| railties | 6.0.3.1 | 5.1.7 | | |
| rb-fsevent | 0.10.4 | 0.9.8 | | |
| rb-inotify | 0.10.1 | 0.9.8 | | |
| responders | 3.0.1 | 2.4.0 | | |
| sass | 3.7.4 | 3.4.23 | | |
| sass-rails | 6.0.0 | 5.0.6 | ~> 5.0 | |
| spring | 2.1.0 | 2.0.1 | | |
| sprockets | 4.0.2 | 3.7.2 | | |
| sqlite3 | 1.4.2 | 1.3.13 | | |
| thor | 1.0.1 | 0.20.3 | | |
| tilt | 2.0.10 | 2.0.7 | | |
| turbolinks | 5.2.1 | 5.0.1 | ~> 5 | |
| turbolinks-source | 5.2.0 | 5.0.0 | | |
| tzinfo | 2.0.2 | 1.2.7 | | |
| uglifier | 4.2.0 | 3.2.0 | | |
| warden | 1.2.8 | 1.2.7 | | |
| web-console | 4.0.3 | 3.5.0 | | |
| websocket-driver | 0.7.2 | 0.6.5 | | |

この時点でyoutube動画と条件も違うのだが、ここは伊藤さんの動画でやっているとおり、  
雑に`bundle update`してみることにする。

けど、正直トラブルになりそうで、とても怖い。。。
とりあえず、後で戻せるようにコミットだけしておく。

`bundle update`は成功し、RSpecでのテストを実行する。  

```
Finished in 25.46 seconds (files took 1.14 seconds to load)
70 examples, 0 failures
```

無事、成功した！！
ただ、動画で説明のとおり、以下のような警告が出ている。  

```
Sign-ups
DEPRECATION WARNING: [Devise] `DeviseHelper.devise_error_messages!`
is deprecated and it will be removed in the next major version.
To customize the errors styles please run `rails g devise:views` and modify the
`devise/shared/error_messages` partial.
```

ここで、警告に書かれているとおり、 `rails g devise:views`を実行する。  

実行すると、オーバーライドするか尋ねられる。  

最初は`n`として、次のファイルにて`d`を押して、差分を確認して以下をコピーする。
他のファイルについては、全て`n`とする。

```
-   <%= devise_error_messages! %>
+   <%= render "devise/shared/error_messages", resource: resource %>
```

コピーした内容を使って、該当部分だけ修正する。  

`<%= devise_error_messages! %>`を全て修正する。
修正後は、`<%= render "devise/shared/error_messages", resource: resource %>`になる。  

該当ファイルは以下のとおり。

<a href="https://gyazo.com/3c14c8690a9f5fa4926e72e85ec57ba6"><img src="https://i.gyazo.com/3c14c8690a9f5fa4926e72e85ec57ba6.png" alt="Image from Gyazo" width="400" border=1/></a>  

修正後にRSpecにてテストを行うと、警告が消えた。  

bundle outdated を行うと、以下のとおりとなった。  

| gem | newest | installed | requested | groups |
| --- | --- | --- | --- | --- |
| actioncable | 6.0.3.1 | 5.1.7 | | |
| actionmailer | 6.0.3.1 | 5.1.7 | | |
| actionpack | 6.0.3.1 | 5.1.7 | | |
| actionview | 6.0.3.1 | 5.1.7 | | |
| activejob | 6.0.3.1 | 5.1.7 | | |
| activemodel | 6.0.3.1 | 5.1.7 | | |
| activerecord | 6.0.3.1 | 5.1.7 | | |
| activesupport | 6.0.3.1 | 5.1.7 | | |
| arel | 9.0.0 | 8.0.0 | | |
| coffee-rails | 5.0.0 | 4.2.2 | ~> 4.2 | |
| listen | 3.2.1 | 3.1.5 | < 3.2, >= 3.0.5 | |
| mini_portile2 | 2.5.0 | 2.4.0 | | |
| puma | 4.3.5 | 3.12.6 | ~> 3.7 | |
| rails | 6.0.3.1 | 5.1.7 | ~> 5.1.1 | |
| railties | 6.0.3.1 | 5.1.7 | | |
| sass-rails | 6.0.0 | 5.0.7 | ~> 5.0 | |
| sprockets | 4.0.2 | 3.7.2 | | |
| thor | 1.0.1 | 0.20.3 | | |
| tzinfo | 2.0.2 | 1.2.7 | | |
| web-console | 4.0.3 | 3.7.0 | | |
| websocket-driver | 0.7.2 | 0.6.5 | | |

伊藤さんの動画だと、以下のとおりとなっている。  

<a href="https://gyazo.com/d7730fcd7c6ce2b20168b1ceeaaa9f7c"><img src="https://i.gyazo.com/d7730fcd7c6ce2b20168b1ceeaaa9f7c.png" alt="Image from Gyazo" width="500" border=1/></a>  

細かくは確認できていないが、おそらく同じ状態になったと思われる。  

指示に従い、Rubyのバージョンを新しくする。  
現在の最新のバージョン（安定版）は2.7.1とのことだが、ちょっと試すのも怖い。  
（動画では2.6.5が採用されている）  

とはいえ、伊藤さんによるとRubyは後方互換性を意識しているとのことなので、  
commit だけ一応しておいて、バージョン2.7.1で試してみる。  

```
# ruby 2.7.1 をダウンロードする
rbenv install 2.7.1

# rubyのバージョンを 2.7.1 に指定する
rbenv local 2.7.1

# rubyのバージョンが 2.7.1 になっていることを確認する
rbenv version
```

改めて、`bundle install`を行う。  

すると、gem install bundler 2.1.4を実行せよと指示が出るので、
その指示に従った後、もう一度`bundle install`を行う。  

警告が出るが、`i18n`, `Paperclip`, `Sass`など、
これまで見たものと基本的に変わらない。

RSpecでテストを実行する。 

```
Users/HOGE/.rbenv/gems/2.7.0/bundler/gems/shoulda-matchers-4b160bd19ecc/lib/shoulda/matchers/active_model/validate_inclusion_of_matcher.rb:273:in `<class:ValidateInclusionOfMatcher>': undefined method `new' for BigDecimal:Class (NoMethodError)
```

エラーとなってしまった。  

動画上では警告が出るだけだったが、Rubyのバージョンを上げたためなのか、  
私の方でそもそもRSpecのテストが出来なくなってしまった。  

ただ、動画によると、このエラーはRailsのバージョンを上げれば解決するようなので、  
Railsを5.2.3にアップデートしてみる。  

なお、動画では`Gemfile`を開き、`shoulda-matchers`のブランチ指定を解除していたが、  
そちらについては後ほど対応することとする。  

Gemfileを開き、railsのバージョンを以下のとおり設定する。  

```
gem 'rails', '~> 5.2.3'
```

そして、`bundle update`を実行する。  
終了した後、`rails app: update`を実行する。  

routes.rbだけは`n`を押し、それ以外は`y`を押す。  

ここで差分を確認しながら、自力で必要な設定を戻していく。  
VSCodeを使っていると、以下のとおり確認できる。  

<a href="https://gyazo.com/fc3575eb252d1b190d4cb7af9d5e2c45"><img src="https://i.gyazo.com/fc3575eb252d1b190d4cb7af9d5e2c45.png" alt="Image from Gyazo" width="800" border=1/></a>  

設定は以下のとおり戻したので、参考までに記す。  
動画のとおり、戻している。 

```rb
# config/application.rb

module Projects
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 5.1

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

```rb
# config/environments/test.rb
# 以下のコードを最後に追加する

# Keep files uploaded in tests from polluting the Rails development
# environment's file uploads
Paperclip::Attachment.default_options[:path] = \
  "#{Rails.root}/spec/test_uploads/:class/:id_partition/:style.:extension"
```

### railsdiff.orgを参考にして、新しく追加されたgem等を確認する

以下のとおり説明があるので、対応する。  

> app:updateコマンドを実行しても、Gemfileのようにまったく更新されないファイルもあります。
> ですが、rails newした直後のGemfileを比較すると、デフォルトでインストールされるgemの種類やバージョンには違いがあります。
> Railsのバージョンを上げたのであれば、こういった部分も新しいRailsに合わせておく方が安心です

[railsdiff.org](http://railsdiff.org/5.1.7/5.2.3)  

以上を開き、動画のとおり更新する。  

更新した箇所は下記のとおり、3つのファイル。  

```
# .gitignore

〜 上部は変更がないので省略 〜

# ---- 以下に修正を加えた ------

# Ignore uploaded files in development # これを追加
/storage/* # これを追加
!/storage/.keep # これを追加

/node_modules
/yarn-error.log
 
/public/assets # これを追加
.byebug_history

# Ignore master key for decrypting credentials and more. # これを追加
/config/master.key # これを追加

# Ignore uploads from Paperclip
/public/system
/spec/test_uploads

.ruby-version
```

```
# Gemfile

source 'https://rubygems.org'

git_source(:github) { |repo| "https://github.com/#{repo}.git" } # ここを変更

ruby '2.7.1' # ここを追加

gem 'rails', '~> 5.2.3'
gem 'sqlite3'
gem 'puma', '~> 3.11' # ここを変更
gem 'sass-rails', '~> 5.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.2'
gem 'turbolinks', '~> 5'
gem 'jbuilder', '~> 2.5'
gem 'bootsnap', '>= 1.1.0', require: false # ここを追加

〜 以下、変更がないので省略します 〜
```

```rb
# app/views/layouts/application.html.erb

<!DOCTYPE html>
<html>
  <head>
    <title>Projects</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

# 以下は変更がないので、省略
```

さて、変更を終えたので、`bundle install`を実行する。  
実行したところ、無事終了した。  

### 動作確認を行う

動画に従い、`rails c`を実行する。  
すると、`rails c`は起動できたが、以下の警告メッセージが出てきた。  

```
/Users/HOGE/.rbenv/gems/2.7.0/gems/actionpack-5.2.4.3/lib/action_dispatch/middleware/stack.rb:37: warning: Using the last argument as keyword parameters is deprecated; maybe ** should be added to the call
/Users/HOGE/.rbenv/gems/2.7.0/gems/actionpack-5.2.4.3/lib/action_dispatch/middleware/static.rb:111: warning: The called method `initialize' is defined here
```

なお、動画のとおり、`rails c`の挙動を確認したがエラーとなった。  

```
irb(main):001:0> User.count
Traceback (most recent call last):
        1: from (irb):1
ActiveRecord::StatementInvalid (Could not find table 'users')
```

動画の中で見逃したのかもしれないが、マイグレーションが必要だったのだろうか。  
`git`で`test`というブランチを作り、保険だけ掛けておく。  

マイグレーションを行うと、無事`rails c`は動いた。  
ただ、動画とは異なり、User.countをしても21という数は出てこない。  

また、先ほど出てきた警告メッセージは、引き続き出てくる。  
気になったので調べてみると、どうやらgemが対応できていないことにより挙動らしい。  

[Ruby 2\.7\.0 \+ Rails 6\.0\.2\.1 で警告 warning: Using the last argument as keyword parameters is deprecated を一旦回避 \- Just do IT](https://k-koh.hatenablog.com/entry/2020/02/07/145957)  

問題はなさそうなので、とりあえずスルーすることにする。  

`rails s`にてサーバーが立ち上がるかも確認。  
無事動き、機能も一通り試したところ問題がなさそうだった。  

bin/rspecを実行する。  
以下のエラーが出て、テストが実行できなかった。  

```
/Users/HOGE/.rbenv/gems/2.7.0/bundler/gems/shoulda-matchers-4b160bd19ecc/lib/shoulda/matchers/active_model/validate_inclusion_of_matcher.rb:273:in `<class:ValidateInclusionOfMatcher>': undefined method `new' for BigDecimal:Class (NoMethodError)
```

shoulda-matchersと書いてあるので、ここで放置してきた
`shoulda-matchers`のブランチ指定を解除を盲目的に実行する。  

`Gemfile`を開き、`gem 'shoulda-matchers'`以下に書いてある２行のブランチ指定のコードを削除する。  
そして、`bundle install`を実行する。  

すると、テストが実行でき、以下の失敗が起きた。  
また、Rubyのバージョンが2.7.1に上がったことによると思われるが、大量の警告が出た。  

```
Failures:

  1) Notes user uploads an attachment
     Failure/Error: fill_in "Message", with: "My book cover"
     
     Capybara::ElementNotFound:
       Unable to find field "Message" that is not disabled

〜 省略 〜

  2) Projects user creates a new project
     Failure/Error: fill_in "Name", with: "Test Project"
     
     Capybara::ElementNotFound:
       Unable to find field "Name" that is not disabled

〜 省略 〜

Finished in 46.59 seconds (files took 3.76 seconds to load)
70 examples, 2 failures

Failed examples:

rspec ./spec/system/notes_spec.rb:11 # Notes user uploads an attachment
rspec ./spec/system/projects_spec.rb:4 # Projects user creates a new project
```

なお、以下の警告はRubyが2.7.1にバージョンアップしたことに起因しないらしい。  
動画内でも取り上げられていた。  

```
DEPRECATION WARNING: The success? predicate is deprecated and will be removed in Rails 6.0. Please use successful? as provided by Rack::Response::Helpers. (called from block (3 levels) in <main> at /Users/HOGE/Desktop/Github作業フォルダ/RSpecPractice/everydayrails-rspec-2017-master/spec/controllers/tasks_controller_spec.rb:40)

# 他の箇所にもRSpec関係のファイルにて該当の警告があり
```

動画に従い、まず以下のFailuresから解決する。  

```
Failures:

  1) Notes user uploads an attachment
     Failure/Error: fill_in "Message", with: "My book cover"
     
     Capybara::ElementNotFound:
       Unable to find field "Message" that is not disabled

〜 省略 〜

  2) Projects user creates a new project
     Failure/Error: fill_in "Name", with: "Test Project"
     
     Capybara::ElementNotFound:
       Unable to find field "Name" that is not disabled
```

これは、form_withに関わるエラーであり、HTMLが生成されるときにidが消えてしまうらしい。  

> [config\.load\_defaultsとnew\_framework\_defaults\_x\_x\.rbの関係を詳しく調べてみた \- Qiita](https://qiita.com/jnchito/items/cce3b2795e1c66735310)  

そこで、以下のファイルを修正する。  

```rb
# config/application.rb

module Projects
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.

    # 5.1から5.2に変更する
    config.load_defaults 5.2

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

RSpecのテストを実行してみると、動画のとおり、全てのテストがパスした。  

```
Finished in 14.12 seconds (files took 1.05 seconds to load)
70 examples, 0 failures
```

次に、警告を解決する。

改めて英文の警告を確認すると、successをsuccessfulに書き換えろと指示があるので、  
その指示に従う。すると、警告が消えるらしい。

以下のファイルを開き、指示のとおり対応した。  
- spec/controllers/home_controller_spec.rb
- spec/controllers/projects_controller_spec.rb
- spec/controllers/tasks_controller_spec.rb
- spec/controllers/home_spec.rb

RSpecのテストを実行してみると、警告が消えた。  
（もちろん、Ruby2.7.1の導入に起因すると思われる大量の警告は残ったままであるが笑）

これで、Rails5.2へのアップデートが終了した。  



