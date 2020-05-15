## 現場Rails Chapter05-1 「テスト（自動テスト）」

---

<BR><BR>

### Chapter05-1 「テストについて」  
### Chapter05-2 「テストを書くことのメリット」

---

テストを自動化することで、以下のようなメリットがある。  

- 手動で書いたコードによって問題起きていないか確認をするのは大変
- テストが気軽に出来るようになれば、新機能実装やリファクタリングにも挑戦しやすくなる
- Ruby・Rails・gemのアップデートが行われていく中で、自動テストなしには対応が困難

なお、逆説的ではあるが、自動テストを前提とすることで以下のメリットが得られる。  

- プログラムの仕様についてのドキュメントになる
- 仕様やインターフェイスを考えるきっかけになる
- テストを意識してコードを書けば、管理しやすいコードになる

また、TDD(Test Driven Development)という考え方があり、  
自動テストは単純に便利な確認ツールに留まらないような側面がある。  

<BR><BR>

### Chapter05-3 「テスト用ライブラリ」  

---

使う３つのライブラリについて。  
とはいっても、試さないことにはイメージが沸かない。。。  

<br>

#### RSpec
---

BDD(Behaviour-Driven Development)のためのテスティングフレームワーク。  
BDDとは、動く仕様書＋テストであるBehaviourを書き、その後でプロダクトコードを書く開発手法。  

Rspecは、テスティングフレームワークで一番人気が高いらしい。  
[Rubyプログラマが一番よく使うテスティングフレームワークはRSpecのようです（2018年3月調べ） \- give IT a try](https://blog.jnito.com/entry/2018/03/09/085614)  

<br>

#### Capybara
---

E2E(End-to-End)テスト用フレームワーク。  
RSpecなどのテスティングライブラリと組み合わせて使用する。  

Webアプリのブラウザの操作をシミュレーションできる。  
よって、Javascript関係の動画が確認できる。  

<br>

#### FactoryBot
---

テスト用データを用意してくれるgem。  

RailsではFixtureという仕組みが標準で備わっているが、  
現場ではFactoryBotを活用することが多いらしい。  

<BR><BR>

### Chapter05-4 「テストの種類」  

---

#### システムテスト

E2Eテストに相当。  
ブラウザを通して、アプリの挙動を確認。  

RSpecでは、System Spec もしくは Feature Spec と言われる。  

#### 結合テスト

機能の連続が想定どおり動くか確認するテスト。  

RSpecでは、Request Spec と言われる。  

#### 機能テスト

コントローラ単位のテスト。  

RSpecでは、Controller Spec と言われる。  

<BR><BR>

### Chapter05-5 「System Spec」  

---

RSpec、Capybara、FactoryBotの準備を行う。  

#### RSpec

GemfileにRSpecのgemを加え、bundle installを行い、bin/rails g rspec:installを行う。  
エラーになった。  

以下のとおり対応すると、さくっとエラーが解決した。  

[Rails 5 - Could not find generator 'rspec:install'](https://stackoverflow.com/questions/39542169/rails-5-could-not-find-generator-rspecinstall)  

#### Capybara

Rails new をした際に、既にインストール済であるとのこと。  
RSpecで使うための準備を行う。  

spec_helper.rbにコードを書き加えていく。  

```rb
# spec_helper.rb
# 該当部分のみ抜粋

# capybaraを導入
require 'capybara/rspec'

RSpec.configure do |config|

  # System Specを使うためのドライバをインストール
  # ブラウザにHeadless Chromeを使用
  config.before(:each, type: :system) do
    driven_by :selenium_chrome_headless
  end

  〜その他のコードなど

end
```

以下に設定についての詳細な説明があった。  
[RSpecコトハジメ ~初期設定マニュアル~ \- Qiita](https://qiita.com/naoki_mochizuki/items/1d3026a32786642fc762)  

#### FactoryBot

GemfileにFactoryBotcのgemを加え、bundle installを行う。  

<BR><BR>

### Chapter05-6 「RSpecの基本形」  

---

いきなりコードが出てくるため、意味がよく分からず。
その後の現場railsの文書にもつらみを感じたので、ネット検索に逃げ出す。  

すると、以下を発見し、前段部分を読了。  
概念を理解するのに、非常に役立った。  

仕様を説明するドキュメントだというのがよく理解できた。  
（むしろテストをどう書くのかについては、まだイメージが掴めず）  

[使えるRSpec入門・その1「RSpecの基本的な構文や便利な機能を理解する」 \- Qiita](https://qiita.com/jnchito/items/42193d066bd61c740612)  


<BR><BR>

### Chapter05-7 「FactoryBotでテストデータを作成できるよう準備する」  

---

まず、ファクトリフォルダ内にファクトリのファイルを作成する。  

```rb
# spec/factories/users.rb

# Userクラスのファクトリを作成している
# factory名とクラスが異なる場合、以下のとおり書くこともできる
# factory :admin_user, class: User do

FactoryBot.define do
  factory :user do
    name {'テストユーザー'}
    email {'test1@example.com'}
    password {'password'}
  end
end
```

```rb
# spec/factories/tasks.rb

FactoryBot.define do
  factory :task do
    name {'テストを書く'}
    description {'RSpecを準備して、コードを書く'}
    
    # 先ほどのuserという名前のファクトリをTaskモデルに定義されたuser<br>
    # という名前の関連（アソシエーション）を生成するの利用するの意味

    # association :user, facotry : admin_user と書くこともできる
    user
  end
end
```

<BR><BR>

### Chapter05-8 「タスクの一覧表示機能のSysem Spec」  

---

早速、一覧画面に遷移したら作成済のタスクが表示されているというSpecを書く。  
まず、設計図となるよう、コメント付きで見取り図を作成する。  


```rb
# spec/sysetem/tasks_spec.rb

require 'rails_helper'

describe 'タスク管理機能', type: :system do
  describe '一覧表示機能' do
    before do
      # ユーザーAを作成しておく
      # 作成者がユーザーAであるタスクを作成しておく
    end

    context 'ユーザーAがログインしているとき' do
      before do
        # ユーザーAでログインする
      end

      it 'ユーザーAが作成したタスクが表示される' do
        # 作成済のタスクの名称が画面上に表示されていることを確認
      end
    end
  end
end

```

以降、コメント部分を順番に作成していく。  

#### ユーザーAを作成しておく

FactoryBot.createメソッドを使い、:userファクトリを指定してオブジェクトを作成。  
:userファクトリで指定のとおり、Userオブジェクトが作成される。  

```rb
user_a = FactoryBot.create(:user)
```

なお、以下のとおり属性を変更してデータを作ることができる。  

```rb
user_a = FactoryBot.create(:user, name: 'ユーザーA', email: 'a@example.com')
```

#### 作成者がユーザーAであるタスクを作成しておく

FactoryBot.createメソッドを使い、:taskファクトリを指定してオブジェクトを作成。  
:taskファクトリで指定のとおり、Taskオブジェクトが作成される。  

```rb
FactoryBot.create(:task)
```

なお、以下のとおり属性を変更してデータを作る。  

```rb
# user: user_a と記載することにより、紐付けができる  
# 再利用の予定がないため、ローカル変数に代入しない  

FactoryBot.create(:task, name: '最初のタスク', user: user_a)
```

#### ユーザーAでログインする

細いステップがあるので、まず洗い出しを行う。  

1. ログイン画面にアクセス
2. メールアドレスを入力
3. パスワードを入力
4. ログインするボタンをクリック  

各操作を記述するにあたって、Capybaraに用意されているメソッドを活用する。  

```rb
  visit login_path

  # fill_in 'labelがついている要素を指定', with: '入力値'
  fill_in 'メールアドレス', with: 'a@example.com'
  fill_in 'パスワード', with: 'password'

  click_button 'ログインする'
```

#### ユーザーAが作成したタスクが表示される

「ページに'最初のタスク'があるか確認する」ためのコードを書く。  

```rb
expect(page).to have_content '最初のタスク'
```

なお、have_contentの部分を「Matcher」という。  

#### テストしてみる

これまで書いたコードを見取り図に当てはめる。  
すると、以下のとおりになる。  

```rb
# spec/sysetem/tasks_spec.rb

require 'rails_helper'

describe 'タスク管理機能', type: :system do
  describe '一覧表示機能' do
    before do
      # ユーザーAを作成しておく
      user_a = FactoryBot.create(:user, name: 'ユーザーA', email: 'a@example.com')
      # 作成者がユーザーAであるタスクを作成しておく
      FactoryBot.create(:task, name: '最初のタスク', user: user_a)
    end

    context 'ユーザーAがログインしているとき' do
      before do
        # ユーザーAでログインする
        visit login_path
        fill_in 'メールアドレス', with: 'a@example.com'
        fill_in 'パスワード', with: 'password'
        click_button 'ログインする'
      end

      it 'ユーザーAが作成したタスクが表示される' do
        # 作成済のタスクの名称が画面上に表示されていることを確認
        expect(page).to have_content '最初のタスク'
      end
    end
  end
end

```

以下のコードをターミナルに打ち込み、テストを実行する。  

```
$ bundle exec rspec spec/system/tasks_spec.rb
```

さて、想定していたとおりにならず、エラーが発生する。。。  
「rb:3: syntax error」という文言があったので、３行目を確認すると「：」を忘れていることが明らかに。  
（なお、上記のコードは修正済のものを掲載）  

再度挑戦する。  
やけに時間がかかった気もするが、無事テストに成功した！  

```
Finished in 12.59 seconds (files took 7.87 seconds to load)
1 example, 0 failures
```