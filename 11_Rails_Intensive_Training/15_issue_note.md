# Issue15 モデルスペックを実装

## どんな感じ？

- 今回はテストのため、画像は貼付しない
- モデルに関するバリデーション・スコープ・メソッドについて、想定したとおりの動きとなっているかテストする
- `bundle exec rspec`を実行して、テストをパスすればよい

## 求められている機能実装・実装条件について

- ユーザーモデルとポストモデルのスペックは最低限書いてください
- その他のモデルは任意とします

## RSpec(rspec-rails)の導入方法

このサイトを参考にした。  

- [Ruby on Rails のテストフレームワーク RSpec 事始め \- Qiita](https://qiita.com/tatsurou313/items/c923338d2e3c07dfd9ee#rspec-%E3%81%AE%E5%AE%9F%E8%A1%8C)

テストは、開発環境ではなくてテスト環境で行うものであるが、RSpec generator が開発環境で使えると便利なため、  
ここに書いて`bundle install`するのがよい。  

また、`gem 'spring-commands-rspec'` を導入してから、`bundle exec spring binstub rspec`のコマンドを  
実行すると、rspecの実行が早くなる。  

```rb
group :development, :test do
  gem 'rspec-rails'
  gem 'spring-commands-rspec'
end
```

また、繰り返しになるが、テスト環境でテストを行うため、テスト環境下で必要なデータベース（およびレコード）を作成する必要がある。  
よって、`rails db:migrate RAILS_ENV=test`をするのを忘れないようにすること。  

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

