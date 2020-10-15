# Issue15 モデルスペックを実装

## どんな感じ？

- 今回はテストのため、画像は貼付しない
- モデルに関するバリデーション・スコープ・メソッドについて、想定したとおりの動きとなっているかテストする
- `rspec`もしくは`bin/rspec`を実行して、テストをパスすればよい

## 求められている機能実装・実装条件について

- ユーザーモデルとポストモデルのスペックは最低限書いてください
- その他のモデルは任意とします

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

あと、モデルテストをたくさん書く場合は、shoulda-matachersというを導入するといいらしい。  
これを導入すると、テストを短く書くことができるようになるらしい。  

- [\[Rails4\] Shoulda\-matchersを導入する方法 \- Qiita](https://qiita.com/tacumai/items/4bc60e7f89953c046949)  

発展的に色々と設定したい場合、こちらが参考になりそうだった。  

- [RSpecコトハジメ ~初期設定マニュアル~ \- Qiita](https://qiita.com/naoki_mochizuki/items/1d3026a32786642fc762)

## FactoryBotの参考資料

公式はこちら。
使い方も書いてあって、結構分かりやすい。  

- [factory\_bot/GETTING\_STARTED\.md at master · thoughtbot/factory\_bot](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#setup)

どうでもいいが、FactoryGirlからFactoryBotになった経緯に興味津々。  

- [Repository Name · Issue \#921 · thoughtbot/factory\_bot](https://github.com/thoughtbot/factory_bot/issues/921)

## RSpec(rspec-rails)の導入方法

テストは、開発環境ではなくてテスト環境で行うものであるが、RSpec generator が開発環境で使えると便利なため、  
ここに書いて`bundle install`するのがよい。  

また、`gem 'spring-commands-rspec'`を導入してから、`bundle exec spring binstub rspec`のコマンドを  
実行すると、`bin/rspec`が使えるようになり、rspecの実行速度が速くなる。  

なお、まとめて100項目程度のテストを実行する場合、`bundle exec rspec`の方が速いらしい。  

```rb
group :development, :test do
  gem 'rspec-rails'
  gem 'spring-commands-rspec'
end
```

また、繰り返しになるが、テスト環境でテストを行うため、テスト環境下で必要なデータベース（およびレコード）を作成する必要がある。  
よって、`rails db:create RAILS_ENV=test`と`rails db:migrate RAILS_ENV=test`をするのを忘れないようにすること。  

Modelスペックのテストを作成する場合、`bin/rails g rspec:model test`を使って`spec/models/test_spec.rb`のテストを  
作成することができる。 テストファイルには、`require 'rails_helper'`を忘れないようにする。（以下、参考例）  

```rb
require 'rails_helper'

RSpec.describe Test, type: :model do
  pending "add some examples to (or delete) #{__FILE__}"
end
```

## FactoryBotの導入方法

## モデルスペックの書き方

何をテストするか → 開発者が書いたコードをテストする
例えば、開発者がコードを書いていないため、@user.posts.buildが成功するかテストしないのが一般的

