# Issue15 モデルスペックを実装

## どんな感じ？

<a href="https://gyazo.com/e74fe5a0cb10c21a0b0cd35d99b262a1"><img src="https://i.gyazo.com/e74fe5a0cb10c21a0b0cd35d99b262a1.png" alt="Image from Gyazo" width="600"/></a>

- モデルに関するバリデーション・スコープ・メソッドについて、想定したとおりの動きとなっているかテストする
- `rspec`もしくは`bin/rspec`を実行して、テストをパスすればよい

## 求められている機能実装・実装条件について

- ユーザーモデルとポストモデルのスペックは最低限書いてください
- その他のモデルは任意とします

## RSpecとFactoryBotについて

Railsには、Minitestというテストのフレームワークがあります。  
また、テスト用のデータ作成には、fixtureというものがあります。  

Minitestやには一定のメリットがあるみたいですが、世間一般では、  
Minitestに代わるものとしてRSpec、fixtureに代わるものとしてFactoryBotがより普及しています。  

伊藤さんが解説を書いています。  
私はとりあえず長いものに巻かれていきます。  

- [MinitestとRSpec、FixturesとFactoryGirlの良いところ悪いところをコードを書いて比較してみた \- give IT a try](https://blog.jnito.com/entry/2015/05/06/074510)

## 環境構築

文末に設定方法に関する記事のリンクを貼ったので、詳細はそちらを参照してください。  
ここでは、概略のみを記します。  

テストは、開発環境ではなくてテスト環境で行うものであるが、RSpec generator が開発環境で使えると便利なため、  
`group :development, :test`配下にgemを置いて、`bundle install`するのがよい。  

また、`gem 'spring-commands-rspec'`を導入してから、`bundle exec spring binstub rspec`のコマンドを  
実行すると、`bin/rspec`が使えるようになり、rspecの実行速度が速くなる。  

なお、まとめて100項目程度のテストを実行する場合、`bundle exec rspec`の方が速いらしい。  

あと、これはお好みであるけど、`gem 'pry-byebug'`や`gem 'pry-rails'`をテスト環境でも使いたい場合、  
`group :development, :test`配下にgemを移動してあげるとよい。  
（その他の設定方法もあるみたいです → [Running ruby debug in rspec? \- Stack Overflow](https://stackoverflow.com/questions/5446594/running-ruby-debug-in-rspec)）  

なので、全て実践すると、イメージはこんな感じ。  
併せて、FactoryBotも導入しておきましょう。  

```rb
group :development, :test do
  gem 'factory_bot_rails'
  gem 'rspec-rails'
  gem 'spring-commands-rspec'
  gem 'pry-byebug'
  gem 'pry-rails'
end
```

`rails g`した場合に自動生成するファイルについては、`config/application.rb`で設定する。  

```rb
# config/application.rb

module InstaClone
  class Application < Rails::Application
    config.generators do |g|
      # ModelとSystemスペック以外は使わないので、以下のような設定にした
      g.test_framework :rspec,
        view_specs: false,
        helper_specs: false,
        controller_specs: false,
        routing_specs: false,
        request_specs: false
    end
  end
end
```

また、繰り返しになるが、テスト環境でテストを行うため、テスト環境下で必要なデータベース（およびレコード）を作成する必要がある。  
よって、`rails db:create RAILS_ENV=test`と`rails db:migrate RAILS_ENV=test`をするのを忘れないようにすること。  

RSpecの導入にあたっては、`rails g rspec:install`を実行する。  
実行すると、`rails_helper.rb`、`spec_helper.rb`、`.rspec`の設定ファイルが生成される。  

`.rspec`ファイルに`--format documentation`を書き込むと、テスト結果がドキュメント形式で出力してくれます。  
`--format documentation`をつけておくと、テスト実行時に頑張ってくれている感が出ます笑  

ただ、テストの数が多くなってくると、`--format documentation`はうざいかもしれないです。  

```text
# --format documentation なし

❯ bin/rspec
Running via Spring preloader in process 75516
..............................*......*.**........

成功したものについては、...で簡略化している
Pendingのものや失敗したものについては、詳細を表示してくれる
```

```text
# --format documentation あり

❯ bin/rspec
Running via Spring preloader in process 75988

Post
  scopeのbody_containメソッド
    Post.body_contain(miketa)をすると、２つのpostsを返す
    Post.body_contain(handsome)をすると、１つのpostsを返す
    Post.body_contain(dyson)をすると、postsを返さない

・・・という感じで1つ1つのテストについて詳細を表示してくれる
```

なお、Modelスペックのテストを作成する場合、`bin/rails g rspec:model test`を使って`spec/models/test_spec.rb`のテストを  
作成することができる。テストファイルには、`require 'rails_helper'`を忘れないようにする（ただ、自動で入るはず）。

## FactoryBotについて

ざっくりとしたイメージでいうと、FactoryBotの使い方としては、  

1. Factoryを定義する
    - FactoryはActiveRecordと紐づいている
    - モデルの数だけ作るとよい
    - 値を定義するときは、`{ }`で囲む。インスタンス生成時に動的に定義される。
      - 動的に定義をしないと、インスタンスを何個も作る場合にトラブルが起きる。
2. Factoryの定義を元にして、インスタンス等を作成する
    - 保存までしない場合は、build
    - 保存までする場合は、create
    - ハッシュで各属性の値を取得したい場合は、attributes_for
    - アソシエーション元のDBを保存までする必要がなく、テストの速さを優先したい場合はbuild_stubbed（小さいアプリは使う必要なさそう）
    - sequenceを使うと、自動で`testuser1, 2 , 3 ...`という形でインスタンスを作成できる
    - アソシエーションを定義する場合は、端的にassociation元のモデル名を小文字で書くだけ

具体的な事例を書いておくと、こんな感じ。  

なお、imagesの部分は、carrierwaveの公式wikiをみながら、訳もわからず書いています。  
gemに関わる部分については、公式のwikiを見ると色々と書いてあることが多い気がします。  

```rb
FactoryBot.define do
  factory :post do
    sequence(:title) { |n| "(Post#{n})" }
    body { |n| "This is the description of pictures! " }
    images { [Rack::Test::UploadedFile.new(Rails.root.join('spec/support/sample1.png'), 'image/png'),
              Rack::Test::UploadedFile.new(Rails.root.join('spec/support/sample2.png'), 'image/png'),] }
    user
  end
end
```

attributes_forやbuild_stubbedはとりあえずそんなに使わないかもしれません。  
（いや、実務だとbuild_stubbedとか結構使うのかもしれないですけど）  

あとは、Traitなどのその他の機能はここに書いてある。  
`GETTING_STARTED.md`というタイトルの割には、めちゃくちゃ充実している。  
これを真面目に読んでいると、なかなかモデルテストを書き出すことはできない（笑）  

- [FactoryBotの公式GitHub](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md)

とりあえず、このチートシートに書いてあることが大体把握できれば、充分すぎると思う。  

- [cheat\-ruby\-sheets/factory\_bot\.md at master · brennovich/cheat\-ruby\-sheets](https://github.com/brennovich/cheat-ruby-sheets/blob/master/factory_bot.md)

英語だけど、概略を説明してくれているので、公式が公開している動画は結構観る価値があると思う。  
イメージを掴むのにおすすめ。ヒゲもじゃのおじさんが解説してくれる。  

- [factory\_bot Ruby Gem Tutorial \| Online Video Tutorial by thoughtbot](https://thoughtbot.com/upcase/videos/factory-bot)

## モデルスペックの書き方

とりあえず、伊藤さんの以下のQiita記事を読んで脳内インデックスを作るとよいと思います。  
モデルテストについては、３番目と４番目の記事は読む必要ありません。  

- [使えるRSpec入門・その1「RSpecの基本的な構文や便利な機能を理解する」 \- Qiita](https://qiita.com/jnchito/items/42193d066bd61c740612)
- [使えるRSpec入門・その2「使用頻度の高いマッチャを使いこなす」 \- Qiita](https://qiita.com/jnchito/items/2e79a1abe7cd8214caa5)

RSpecは、こんな構造になっています。  
他にも色々とありますが、とりあえずこれだけ覚えればいいような気がします。  

- describeで大きくグループ分けする
- contextでテストケースについて書く（ざっくりと、短く分かりやすいのがいいらしい）
- itで想定される結果について書く

```rb
RSpec.describe User, type: :model do
  describe '~' do
    context '~' do
      it `~` do
      end
    end
  end
end
```

`shared_context`、`shared_examples`、他`subject`などがありますが、これはDRYに書くためのものなので、  
この辺りはテストをある程度書いてから整える上で使っていけばよいかと思われます。  

始めるにあたっては、具体的なコードを書く前に、このRSpecの構造に沿って`it ~ do`の〜部分をまず書き出して、  
正常系と異常系を意識して、具体的に失敗すべきケースと成功すべきケースをざっと書き出すのがよいかと思います。  

そして、そのあとにコードを書いて、DRY・可読性を意識してリファクタリングするという手順がよいかなと。  
とりあえず、ガシガシ書いて、テスト実装を楽しみましょう笑

### 何のテストを書くべきか

何についてテストを書くべきかについても、伊藤さんが色々と情報発信をしているので、その情報を参考にするとよいと思います。  
基本的には、自分で何かロジックを作った場合に書くことを意識すればよいのかなと感じました。  

逆に、Rails標準の機能やgemの機能については信頼性が高いと思われるので、テストを書く必要性は薄いかと思います。  

ただ、認証や認可周りであれば重要性が高いので、テストをたくさん書く必要があるかもしません。  
ケースバイケース、って感じらしいです。  

### Tips

変数を定義する代わりに、letとlet!を使っていきましょう！  

`eq 10`や`to be_falsey`などのマッチャというRSpec独特のものもあるのですが、以下をとりあえず使ってみれば、  
よいと思います。極端なことを言えば、true/falseもしくは想定した値になるかの2つのパターンだけで、テストを書いて  
いくようなスタンスでも、それなりになんとかなると思います。  

- to
- not_to
- be
- be_truthy
- be_falsey
- change
- include

FactoryBotは、とりあえずcreateとbuildを覚えましょう。他はとりあえずスルーでもいいかも。  

## RSpec関係の参考資料

公式はこちら。  

- [Start from scratch \- RSpec Rails \- RSpec \- Relish](https://relishapp.com/rspec/rspec-rails/v/4-0/docs/gettingstarted)
- [rspec/rspec\-rails: RSpec for Rails 5\+](https://github.com/rspec/rspec-rails)

設定方法についてこちらがよさそう。  
あと、現場Railsも当然ながらおすすめ。  

- [基礎からやり直す Rails RSpec \- Qiita](https://qiita.com/kskinaba/items/38b143e79bdfd44cc42a)
- [RailsアプリへのRspecとFactory\_botの導入手順 \- Qiita](https://qiita.com/Ushinji/items/522ed01c9c14b680222c)
- [【Rails】『RSpec \+ FactoryBot \+ Capybara \+ Webdrivers』の導入＆初期設定からテストの書き方まで \| vdeep](http://vdeep.net/rubyonrails-rspec-factorybot-capybara)
- [現場で使える Ruby on Rails 5速習実践ガイド \| マイナビブックス](https://book.mynavi.jp/ec/products/detail/id=93905)

もちろん、伊藤淳一さんのブログ等はおすすめ。  
とりあえず、このシリーズは必読。  

- [使えるRSpec入門・その1「RSpecの基本的な構文や便利な機能を理解する」 \- Qiita](https://qiita.com/jnchito/items/42193d066bd61c740612)

モデルテストをたくさん書く場合は、shoulda-matachersというを導入するといいらしい。  
これを導入すると、テストを短く書くことができるようになるらしい。  

- [\[Rails4\] Shoulda\-matchersを導入する方法 \- Qiita](https://qiita.com/tacumai/items/4bc60e7f89953c046949)  

発展的に色々と設定したい場合、こちらが参考になりそうだった。  

- [RSpecコトハジメ ~初期設定マニュアル~ \- Qiita](https://qiita.com/naoki_mochizuki/items/1d3026a32786642fc762)

RSpecの書き方について。  
Better Specsが様々なグッドプラクティスを紹介している。  

あと、伊藤さんがテストコードの方針、どこまで書き方にこだわるか、テストコードの意義について、  
などなどと色々と情報発信をしている。  

- [Better Specs\. Testing Guidelines for Developers\.](https://www.betterspecs.org/)
- [【初心者向け】テストコードの方針を考える（何をテストすべきか？どんなテストを書くべきか？） \- Qiita](https://qiita.com/jnchito/items/2a5d3e15761fd413657a#%E3%83%86%E3%82%B9%E3%83%88%E3%82%92%E6%9B%B8%E3%81%8F%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E6%89%8B%E3%81%AE%E9%80%9F%E3%81%95%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)
- [サヨナラBetter Specs\!? 雑で気楽なRSpecのススメ \- Qiita](https://qiita.com/jnchito/items/a90b3b09d008227d3d60)
- [テストコードの期待値はDRYを捨ててベタ書きする ～テストコードの重要な役割とは？～ \- Qiita](https://qiita.com/jnchito/items/eb3cfa9f7db752dcb796#comments)

ちなみに、simplecovというgemを導入すると、カバレッジ率が分かる。  
簡単に導入できるし、網羅できない箇所が分かるのでよい。  

- [simplecov\-ruby/simplecov: Code coverage for Ruby with a powerful configuration library and automatic merging of coverage across test suites](https://github.com/simplecov-ruby/simplecov)
- [simplecovでRspecのテストを書くが楽しくなった話 \- Qiita](https://qiita.com/komatsubara/items/02962feb28a9eb7e9123)

<a href="https://gyazo.com/8fbce18b92cfec9eef33738430f12576"><img src="https://i.gyazo.com/8fbce18b92cfec9eef33738430f12576.png" alt="Image from Gyazo" width="600"/></a>

## FactoryBotの参考資料

公式はこちら。  
使い方も書いてあって、結構分かりやすい。  

- [factory\_bot/GETTING\_STARTED\.md at master · thoughtbot/factory\_bot](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#setup)

日本語だとこれがよい。  
公式Wikiの翻訳に近い感じで紹介している。  

- [FactoryBot\(FactoryGirl\)チートシート \- Qiita](https://qiita.com/morrr/items/f1d3ac46b029ccddd017#%E3%82%A2%E3%82%BD%E3%82%B7%E3%82%A8%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E5%AE%9A%E7%BE%A9%E3%81%99%E3%82%8Bassociation)

あと、公式動画もある。概略が掴めるのでおすすめ。  
ヒゲもじゃのおじさんが解説してくれる。  

- [factory\_bot Ruby Gem Tutorial \| Online Video Tutorial by thoughtbot](https://thoughtbot.com/upcase/videos/factory-bot)

どうでもいいが、FactoryGirlからFactoryBotになった経緯に興味津々。  

- [Repository Name · Issue \#921 · thoughtbot/factory\_bot](https://github.com/thoughtbot/factory_bot/issues/921)