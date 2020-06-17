# Rails6系で「everyday Rails」（RSpec）を勉強したいので、伊藤さんの記事を参考にして頑張ってみた（part1）

## はじめに

タイトルのとおりやってみたいと考えていたところ、伊藤さんの以下の記事を発見した。  

> [【動画付き】Everyday RailsのサンプルアプリをRails 6で動かす際に必要なテストコードの変更点 \- give IT a try](https://blog.jnito.com/entry/2019/10/25/053521)

この記事では、その経過報告を記す。  
なお、経過報告なので冗長な部分がある。

### 伊藤さんのQiita記事について

伊藤さんは「everyday Rails」の翻訳者の一人であり、ブログで積極的な情報発信を行っている。  

> Everyday Railsのサンプルアプリケーションを最新化するためには、順を追って作業を進めていく必要があります。  
> ここでは以前僕がQiitaに書いた以下の記事に従ってアップグレードを進めるものとします。

以上のとおり説明があったので、以下のQiita記事を参考に進めることにした。

だが、進めていく中で分かったが、個人的にはQiita記事はあくまで参考に留め、  
基本的には動画のとおり対応していった方がよいと思われる。  

> [永久保存版！？伊藤さん式・Railsアプリのアップグレード手順 \- Qiita](https://qiita.com/jnchito/items/0ee47108972a0e302caf)  
> [【前編】永久保存版！？伊藤さん式・Railsアプリのアップグレード手順](https://youtube.com/watch?v=hT68fhuWbHU)
> [【後編】永久保存版！？伊藤さん式・Railsアプリのアップグレード手順](https://youtube.com/watch?v=SnwNFMauzjM)

### 動画を見ましょう（これだけ分かれば、別にこれ以降は読まなくてもよいです笑）

（急にここだけ「ですます調」で訴えます）

動画では伊藤さんが行う操作が一部始終確認できます。  
私は、Qiita記事だけを見ながらやろうとしたら、色々とトラブルに見舞われました。  

知識がない人がQiita記事だけの情報でチャレンジすると、色々と勘違いをする可能性や、  
Qiita記事に細かく書いていないエラーに対応できない可能性が高いです。  

その私の残念な様子は、「Rails以外のgemをバージョンアップする」の「第１チャレンジ」に書きました。  
私の失敗はかなり残念ですが、動画を見ていただけると、  
「Qiita記事だけでは初学者は対応できない」ということが分かっていただけるかと思います。  

Qiita記事は、あくまでRSpecのテストアプリを事例として扱いつつ、  
Railsアプリのアップグレード手順を説明するものです。  
その性質上、細かい手順等はQiita記事内に書かれておりません。

繰り返しますが、**動画を見ながらやりましょう！**  

## テストが全部パスすることを確認する

公式のアップグレードガイドに目を通した後、テストが全てパスするか確認するよう指示が出ている。  
その経過について、以下に記す。

### Specテストでいきなりパスしない！

RSpecでのテストは以下のコマンドで実行できる。
記事内の記載とは異なるが、伊藤さんのyoutube動画内で紹介されていた。

```
bin/rspec
```

やってみると、下記の記事で紹介されているようなエラーが出た。  

> [Everyday Railsのセットアップにて、masterブランチのテストを全てパスしたい](https://teratail.com/questions/247071)

どうやら、Rubyのバージョンが問題であるようなので、大人しくRubyを2.4.9に指定することにした。  
私は、Rbenvを使っているのだが、使い方を忘れているので、以下を参照して対応した。  

> [Rubyのバージョン変更 \- Qiita](https://qiita.com/pipi_taro/items/9c9a6ccc110b159b967b)

```
# ruby 2.4.9 をダウンロードする
rbenv install 2.4.9

# rubyのバージョンを 2.4.9 に指定する
rbenv local 2.4.9

# rubyのバージョンが 2.4.9 になっていることを確認する
rbenv version
```

Rubyのバージョンを2.4.9に合わせた。 

### また、Specテストにパスしない！

RSpecで再テストすると、以下のとおりエラーが出た。  

```
Failures:

  1) Tasks user toggles a task
     Got 0 failures and 2 other errors:

     1.1) Failure/Error: visit root_path
          
          Selenium::WebDriver::Error::SessionNotCreatedError:
            session not created: This version of ChromeDriver only supports Chrome version 81

     1.2) Failure/Error: Unable to infer file and line number from backtrace
          
          Selenium::WebDriver::Error::SessionNotCreatedError:
            session not created: This version of ChromeDriver only supports Chrome version 81
          
Finished in 27.03 seconds (files took 1.18 seconds to load)
70 examples, 1 failure

Failed examples:

rspec ./spec/system/tasks_spec.rb:12 # Tasks user toggles a task
```

Pythonの話がベースだが、以下の記事を参考にした。  
[seleniumで「Message: session not created: This version of ChromeDriver only supports Chrome version 75」のエラーが表示される場合の対処法 \- Qiita](https://qiita.com/stoneBK7/items/9f258e8ab9ce2d4fa4f5)  

どうやら、私が使っているChromeのバージョンとの不一致が問題である可能性が高い。  
Chromeのバージョンを確認すると、以下のとおりバージョンが83だった。  

<a href="https://gyazo.com/4f91e03eada35819ff613bbb192bd365"><img src="https://i.gyazo.com/4f91e03eada35819ff613bbb192bd365.png" alt="Image from Gyazo" width="500" border=1/></a>  

エラーメッセージを見直してみると、Everyday Railsの方ではバージョンが81を要求している。  

ここでバージョンのダウングレードをしてもよいのだが、そもそもRailsのバージョンを6にアップグレードした際に、  
`chromedriver-helper`を`webdrivers`に変更するよう指示が出ているので、ここはスルーすることにする。  

## Rails以外のgemをバージョンアップする

さて、指示のとおりブランチを作成した後、Rails以外のgemをバージョンアップする。  
その経過報告を以下に記す。なお、一度失敗しているので、ここの記述はかなり冗長である。

### 第１チャレンジ

gem が最新か確認するように指示があるので`bundle outdated`を実行してみる。  

```
outdated gems included in the bundle:
  * actioncable (newest 6.0.3.1, installed 5.1.1)
  * actionmailer (newest 6.0.3.1, installed 5.1.1)
  * actionpack (newest 6.0.3.1, installed 5.1.1)
  * actionview (newest 6.0.3.1, installed 5.1.1)
  * activejob (newest 6.0.3.1, installed 5.1.1)
  * activemodel (newest 6.0.3.1, installed 5.1.1)
  * activerecord (newest 6.0.3.1, installed 5.1.1)
  * activesupport (newest 6.0.3.1, installed 5.1.1)
  * addressable (newest 2.7.0, installed 2.5.2)
  * archive-zip (newest 0.12.0, installed 0.11.0)
  * arel (newest 9.0.0, installed 8.0.0)
  * autoprefixer-rails (newest 9.7.6, installed 6.7.7.2)
  * bcrypt (newest 3.1.13, installed 3.1.12)
  * bindex (newest 0.8.1, installed 0.5.0)
  * bootstrap-sass (newest 3.4.1, installed 3.3.7) in group "default"
  * builder (newest 3.2.4, installed 3.2.3)
  * byebug (newest 11.1.3, installed 9.0.6) in groups "development, test"
  * capybara (newest 3.32.2, installed 2.15.4, requested ~> 2.15.4) in group "test"
  * childprocess (newest 3.0.0, installed 0.8.0)
  * chromedriver-helper (newest 2.1.1, installed 1.2.0) in group "test"
  * cocaine (newest 0.6.0, installed 0.5.8)
  * coffee-rails (newest 5.0.0, installed 4.2.1, requested ~> 4.2) in group "default"
  * concurrent-ruby (newest 1.1.6, installed 1.0.5)
  * crass (newest 1.0.6, installed 1.0.4)
  * devise (newest 4.7.2, installed 4.4.3) in group "default"
  * erubi (newest 1.9.0, installed 1.7.1)
  * factory_bot (newest 5.2.0, installed 4.10.0)
  * factory_bot_rails (newest 5.2.0, installed 4.10.0, requested ~> 4.10.0) in groups "development, test"
  * faker (newest 2.12.0, installed 1.7.3) in group "development"
  * ffi (newest 1.13.1, installed 1.9.18)
  * geocoder (newest 1.6.3, installed 1.4.9) in group "default"
  * globalid (newest 0.4.2, installed 0.4.0)
  * hashdiff (newest 1.0.1, installed 0.3.4)
  * i18n (newest 1.8.3, installed 0.9.5)
  * io-like (newest 0.3.1, installed 0.3.0)
  * jbuilder (newest 2.10.0, installed 2.6.3, requested ~> 2.5) in group "default"
  * jquery-rails (newest 4.4.0, installed 4.3.1) in group "default"
  * launchy (newest 2.5.0, installed 2.4.3, requested ~> 2.4.3) in group "test"
  * listen (newest 3.2.1, installed 3.1.5, requested < 3.2, >= 3.0.5) in group "development"
  * loofah (newest 2.6.0, installed 2.2.2)
  * mail (newest 2.7.1, installed 2.6.5)
  * method_source (newest 1.0.0, installed 0.9.0)
  * mime-types (newest 3.3.1, installed 3.1)
  * mime-types-data (newest 3.2020.0512, installed 3.2016.0521)
  * mimemagic (newest 0.3.5, installed 0.3.2)
  * mini_mime (newest 1.0.2, installed 0.1.4)
  * mini_portile2 (newest 2.5.0, installed 2.3.0)
  * minitest (newest 5.14.1, installed 5.11.3)
  * multi_json (newest 1.14.1, installed 1.12.1)
  * nio4r (newest 2.5.2, installed 2.0.0)
  * nokogiri (newest 1.10.9, installed 1.8.4)
  * paperclip (newest 6.1.0, installed 5.1.0) in group "default"
  * public_suffix (newest 4.0.5, installed 3.0.0)
  * puma (newest 4.3.5, installed 3.8.2, requested ~> 3.7) in group "default"
  * rack (newest 2.2.3, installed 2.0.5)
  * rack-test (newest 1.1.0, installed 0.6.3)
  * rails (newest 6.0.3.1, installed 5.1.1, requested ~> 5.1.1) in group "default"
  * rails-html-sanitizer (newest 1.3.0, installed 1.0.4)
  * railties (newest 6.0.3.1, installed 5.1.1)
  * rake (newest 13.0.1, installed 12.3.1)
  * rb-fsevent (newest 0.10.4, installed 0.9.8)
  * rb-inotify (newest 0.10.1, installed 0.9.8)
  * responders (newest 3.0.1, installed 2.4.0)
  * rspec-core (newest 3.9.2, installed 3.8.0)
  * rspec-expectations (newest 3.9.2, installed 3.8.1)
  * rspec-mocks (newest 3.9.1, installed 3.8.0)
  * rspec-rails (newest 4.0.1, installed 3.8.0, requested ~> 3.8.0) in groups "development, test"
  * rspec-support (newest 3.9.3, installed 3.8.0)
  * rubyzip (newest 2.3.0, installed 1.2.1)
  * safe_yaml (newest 1.0.5, installed 1.0.4)
  * sass (newest 3.7.4, installed 3.4.23)
  * sass-rails (newest 6.0.0, installed 5.0.6, requested ~> 5.0) in group "default"
  * selenium-webdriver (newest 3.142.7, installed 3.6.0) in group "test"
  * spring (newest 2.1.0, installed 2.0.1) in group "development"
  * sprockets (newest 4.0.2, installed 3.7.1)
  * sprockets-rails (newest 3.2.1, installed 3.2.0)
  * sqlite3 (newest 1.4.2, installed 1.3.13) in group "default"
  * thor (newest 1.0.1, installed 0.20.0)
  * tilt (newest 2.0.10, installed 2.0.7)
  * turbolinks (newest 5.2.1, installed 5.0.1, requested ~> 5) in group "default"
  * turbolinks-source (newest 5.2.0, installed 5.0.0)
  * tzinfo (newest 2.0.2, installed 1.2.5)
  * uglifier (newest 4.2.0, installed 3.2.0) in group "default"
  * vcr (newest 6.0.0, installed 3.0.3) in group "test"
  * warden (newest 1.2.8, installed 1.2.7)
  * web-console (newest 4.0.2, installed 3.5.0) in group "development"
  * webmock (newest 3.8.3, installed 3.0.1) in group "test"
  * websocket-driver (newest 0.7.2, installed 0.6.5)
  * websocket-extensions (newest 0.1.5, installed 0.1.2)
  * xpath (newest 3.2.0, installed 2.1.0)
```

悪夢のような数のgemが。。。  

ここで、激しい勘違いをしていたのだが、in group development と in group test のものを  
まずアップデートしようと伊藤さんは言っているのであって、その結果が以下の表になるということらしい。  

| gem | newest | installed | requested | groups |
| --- | --- | --- | --- | --- |
| bootstrap-sass | 3.4.1 | 3.4.0 | | default |
| capybara | 3.28.0 | 3.12.0 | | test |
| childprocess | 2.0.0 | 0.9.0 | | |
| chromedriver-helper | 2.1.1 | 2.1.0 | | test |
| coffee-rails | 5.0.0 | 4.2.2 | | default |
| dotenv | 2.7.5 | 2.5.0 | | |
| dotenv-rails | 2.7.5 | 2.5.0 | | development, test |

書いてあることと違う・・・と思っていたが、動画を見ると同じように`outdated gems`が表示されていた。  

上記の表は、エクセルでフィルターを掛けて、最終的に整えたものをマークダウンで出しているだけなので、
私だけかもしれないが、記事はよく読みましょう（笑）  

一応、証拠として、貼っておきます。  
何のための証拠なのかよく分かりませんが。  

あと、見返してみると、上の表と動画内の表の中身が違いますね。。。  
試した時期の違いによるものかもしれないですね。。。  

<a href="https://gyazo.com/0174626ff1b1036eb76602f2e54a1bfd"><img src="https://i.gyazo.com/0174626ff1b1036eb76602f2e54a1bfd.png" alt="Image from Gyazo" width="500"/></a>

ちなみに、私はここから明後日の方向に進み出して、最終的に面倒になって、  
`bundle update`をガツンとしたら、RSpecで以下のようなエラーが出た。  

```
There is a version mismatch between the Spring client (2.1.0) and the server (2.0.1).

〜  省略 〜

WARN Selenium [DEPRECATION] Selenium::WebDriver::Chrome#driver_path= is deprecated. Use Selenium::WebDriver::Chrome::Service#driver_path= instead.

〜  省略 〜
```

最初から頑張ってみることにする。  
あと、動画は絶対に見た方がいい。これは確信した。  

### 第２チャレンジ（動画を見ながら）

ここまでの知見を生かし、このステージまで素早く戻ってくる。  

素直に、`Gemfile`に`bundle_outdated_formatter`を導入してみる。

指示どおりやっていくと、以下のとおりのエラーが出る。  
これは動画でも紹介されているとおりである。

```
Resolving dependencies...
Bundler could not find compatible versions for gem "bundler":
  In Gemfile:
    rails (~> 5.1.1) was resolved to 5.1.1, which depends on
      bundler (< 2.0, >= 1.3.0)

  Current Bundler version:
    bundler (2.1.4)
This Gemfile requires a different version of Bundler.
Perhaps you need to update Bundler by running `gem install bundler`?

Could not find gem 'bundler (< 2.0, >= 1.3.0)', which is required by gem 'rails (~> 5.1.1)', in any of the sources.
```

ちなみに、これは先ほど試行錯誤していく中でも出たエラーである。  

私は、エラーメッセージに従う形でbundlerのバージョンを下げたりして、  
ゴニョゴニョやっていったが、伊藤さんによると「`bundle update rails`ならいけるっぽい」  
ということなので、ここはこのコマンドを試してみる。  

やってみると、上手くアップデートできた。  
動画では理屈について少し言及していたが、初学者なので理屈の部分はスルーすることにした。  
  
　 
ちなみに、`bof`が`bundle outdated formatter`の略である。  

その表を手持ちの`libre office`で加工すると、以下のとおりの表となりました。  

<a href="https://gyazo.com/45c7d4c9bb04c5e3aa8ce387ecba953b"><img src="https://i.gyazo.com/45c7d4c9bb04c5e3aa8ce387ecba953b.png" alt="Image from Gyazo" width="500"/></a>

何やら、伊藤さんのものと比べて、かなりシンプルなものになりました。  

さて、Gemfileを開き、以下のgemのバージョン指定を外してみる。  
外すものは、動画で言及されていたものに限定しました。  

```
group :development, :test do
  gem 'rspec-rails' # バージョン指定を解除
  gem 'factory_bot_rails' # バージョン指定を解除
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
end

group :test do
  gem 'capybara' # バージョン指定を解除
  gem 'selenium-webdriver' # バージョン指定を解除
  gem 'chromedriver-helper'
  # Or use poltergeist and PhantomJS as an alternative to Selenium/Chrome
  # gem 'poltergeist', '~> 1.15.0'
  gem 'launchy'
  gem 'shoulda-matchers',
    git: 'https://github.com/thoughtbot/shoulda-matchers.git',
    branch: 'rails-5'
  gem 'vcr'
  gem 'webmock'
end
```

さて、以下を実行する。  

```
bundle update -g development -g test
```

無事成功した。
ちなみに、伊藤さんの動画では、`chromedriver-helper`から、`webdrivers`に移行するよう指示が出ていた。  

[サポートが終了したchromedriver\-helperからwebdrivers gemに移行する手順 \- Qiita](https://qiita.com/jnchito/items/f9c3be449fd164176efa)

詳細は動画のとおりだが、Gemfile内の`chromedriver-helper`を`webdrivers`に書き換えて、  
`bundle install`すればよいらしい。私の場合、`bundle install`は問題なく実行できた。  

さて、RSpecで再テストする。  
何やら、エラーが色々と出てきた。  

```
[DEPRECATED] `Bundler.with_clean_env` has been deprecated in favor of `Bundler.with_unbundled_env`. If you instead want the environment before bundler was originally loaded, use `Bundler.with_original_env` (called at /Users/kentasuedomi/.rbenv/gems/2.4.0/gems/spring-2.0.1/lib/spring/application_manager.rb:95)
2020-06-17 18:31:22 WARN Selenium [DEPRECATION] Selenium::WebDriver::Chrome#driver_path= is deprecated. Use Selenium::WebDriver::Chrome::Service#driver_path= instead.
/Users/kentasuedomi/.rbenv/gems/2.4.0/gems/factory_bot-5.2.0/lib/factory_bot/definition_proxy.rb:99:in `method_missing': undefined method 'message' in 'note' factory
Did you mean? 'message { "My important note." }'
 (NoMethodError)

〜 以下、省略 〜

```

この記事に詳細があるらしい。

> [【翻訳】factory\_bot 4\.11で非推奨になった静的属性（static attributes） \- Qiita](https://qiita.com/jnchito/items/81637bbdf66c2662eacf)

動画で紹介されているとおり、まず以下を実行する。  

```
gem install rubocop-rspec
```

そして、以下を実行する。

```
rubocop \
  --require rubocop-rspec \
  --only FactoryBot/AttributeDefinedStatically \
  --auto-correct

〜 省略 〜

104 files inspected, 10 offenses detected, 10 offenses corrected
```

RSpecでテストする。  
詳細は省略するが、以下の部分だけテストで失敗した。 

```
Failures:

  1) Tasks user toggles a task
     Got 0 failures and 2 other errors:

     1.1) Failure/Error: visit root_path
          
          VCR::Errors::UnhandledHTTPRequestError:

     1.2) Failure/Error: raise VCR::Errors::UnhandledHTTPRequestError.new(vcr_request)
          
          VCR::Errors::UnhandledHTTPRequestError:

```

ただ、この失敗は動画で紹介されているものである。  
詳細は動画で確認してもらいたいが、以下のとおり設定すればテストの失敗は回避できるらしい。  

```rb
# spec/support/vcr.rb

VCR.configure do |config|
  config.cassette_library_dir = "#{::Rails.root}/spec/cassettes"
  config.hook_into :webmock
  config.ignore_localhost = true
  # 下記の config.ignore_hosts で始まる１行を追加
  config.ignore_hosts 'chromedriver.storage.googleapis.com'
  config.configure_rspec_metadata!
end
```

RSpecでテストする。  
すると、何と全てのテストが通る！！！

```
Finished in 9.03 seconds (files took 1.11 seconds to load)
70 examples, 0 failures
```

ちなみに、これでQiita記事でいうところの「4-b. developmentとtestグループのgemを先にアップデートする」まで終わりました。  
動画だと、全編の17分ぐらい経過したところまでです。  

[4-b. developmentとtestグループのgemを先にアップデートする（伊藤さんのQiita記事）](https://qiita.com/jnchito/items/0ee47108972a0e302caf#4-b-development%E3%81%A8test%E3%82%B0%E3%83%AB%E3%83%BC%E3%83%97%E3%81%AEgem%E3%82%92%E5%85%88%E3%81%AB%E3%82%A2%E3%83%83%E3%83%97%E3%83%87%E3%83%BC%E3%83%88%E3%81%99%E3%82%8B)


