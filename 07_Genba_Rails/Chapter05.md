## 現場Rails Chapter05 「テストをはじめよう」

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

<BR><BR>

### Chapter05-9 「他のユーザーが作成したタスクが表示されないことの確認」  

---

続いて、ユーザーAではない、ユーザーBがログインした時にタスク一覧が表示されないか確認する。  
そのためには、表示機能のテストケースを増やしてみる。  

1. ユーザーAとユーザーAのタスクを作成する（済）
2. ユーザーBを作成する
3. ユーザーBでログインする
4. ユーザーAのタスクが表示されていないことを確認する

コードを書いていく。  

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

    # 新規で追加した部分
    context 'ユーザーBがログインしているとき' do
      before do
        # ユーザーBを作成しておく
        FactoryBot.create(:user, name: 'ユーザーB', email: 'b@example.com')
        # ユーザーBでログインする
        visit login_path
        fill_in 'メールアドレス', with: 'b@example.com'
        fill_in 'パスワード', with: 'password'
        click_button 'ログインする'
      end

      it 'ユーザーAが作成したタスクが表示されない' do
        # ユーザーAとは違い「have_no_content」という matcher を使用
        expect(page).to have_no_content '最初のタスク'
      end 
    end
  end
end

```

```
Finished in 4.14 seconds (files took 3.86 seconds to load)
2 examples, 0 failures
```

珍しく、エラーもなく自動テスト成功！  

<BR><BR>

### Chapter05-10 「beforeを利用した共通化」  
### Chapter05-11 「letを利用した共通化」  

---

次に、ユーザーAとユーザーBがログインしているときのbeforeの内容が類似しているため、  
共通化の作業を行う。  

具体的には、上の階層のbeforeに処理を書くことで実現できるとのこと。  

とはいえ、ここで問題が発生。  
なぜなら、ユーザーAやBをログインさせるプロセスは共有化できないからだ。

ユーザーAさんについてテストするときはAさんをログインさせ、  
BさんについてテストするときはBさんをログインさせる、というな形でテストを書き換える必要がある。  

そのためには、letを使う必要がある。  
letを使い、Aさんについて試すときはAさんに関するletを呼び出し、  
Bさんについて試すときはBさんに関するletを呼び出すように設定する。  

すると、コードは以下のとおりになる。  

```rb
# spec/sysetem/tasks_spec.rb

require 'rails_helper'

describe 'タスク管理機能', type: :system do
  describe '一覧表示機能' do
    # ここでletを使う
    let(:user_a){ FactoryBot.create(:user, name: 'ユーザーA', email: 'a@example.com') }
    let(:user_b){ FactoryBot.create(:user, name: 'ユーザーB', email: 'b@example.com') }

    before do
      # ユーザーAとBに関する部分を共通化した
      # ここでlet(:user_a）が呼び出されて、ユーザーAが作成される。そして、タスクも作成される。  
      FactoryBot.create(:task, name: '最初のタスク', user: user_a)
      visit login_path
      # letで指定されたユーザーでログイン
      fill_in 'メールアドレス', with: login_user.email
      fill_in 'パスワード', with: login_user.password
      click_button 'ログインする'
    end

    context 'ユーザーAがログインしているとき' do
      # ユーザーAでログイン（使用するログインユーザーをletで定義）
      let(:login_user){ user_a }

      it 'ユーザーAが作成したタスクが表示される' do
        # 作成済のタスクの名称が画面上に表示されていることを確認
        expect(page).to have_content '最初のタスク'
      end
    end

    context 'ユーザーBがログインしているとき' do
      # ユーザーBでログイン（使用するログインユーザーをletで定義）
      # ここで初めて、ユーザーBが作成される
      let(:login_user){ user_b }

      it 'ユーザーAが作成したタスクが表示されない' do
        # ユーザーAとは違い「have_no_content」という matcher を使用
        expect(page).to have_no_content '最初のタスク'
      end 
    end
  end
end

```

試してみるが、以下のとおり。  

```
Failures:

  1) タスク管理機能 一覧表示機能 ユーザーBがログインしているとき ユーザーAが作成したタスクが表示されない
     Failure/Error: expect(page).to have_no_content '最初のタスク'
       expected not to find text "最初のタスク" in "Taskleaf\nタスク一覧\nログアウト\nログインしました。\nタスク一覧\n新規登録\n名称 Created\n最初のタスク 2020-05-16 09:51:46 UTC 編集削除"
     
     [Screenshot]: tmp/screenshots/failures_r_spec_example_groups_nested_nested_b_ユーザーaが作成したタスクが表示されない_184.png

     
     # ./spec/system/tasks_spec.rb:37:in `block (4 levels) in <top (required)>'

Finished in 7.16 seconds (files took 4.04 seconds to load)
2 examples, 1 failure
```

以下のとおり、スクショまで出してくれるので感動ものである。  
自動テスト、すごいな。。。  

（ただし、今回は自動テストが意図したとおりに動いていないのだけれど）

<a href="https://gyazo.com/af584d4ccf97c8923e199c71361599e1"><img src="https://i.gyazo.com/af584d4ccf97c8923e199c71361599e1.png" alt="Image from Gyazo" width="550" border=1/></a>

なお、ここは本来、自動テストで失敗が起きてはいけないところ。  
改めてRSpecのコードの確認をする。やはり誤りあり。  
（なお、上記のコードは修正済のものを掲載）  

コード修正後、以下のとおりの結果となった。  

```
Finished in 4.25 seconds (files took 3.43 seconds to load)
2 examples, 0 failures
```

最初は、アプリの方にバグがあるのか、それとも自動テストが間違っているのか、  
といった問題に悩まされそうだ。。。  

特に、テストを少しずつ試さずにアプリ開発を進めてしまうと、アプリもバグっているし、  
自動テストもバグっているし、何がなんだか分からない!!!! と言った状態になりそう。。。笑  

自作アプリの際には、こまめに確認を行っていきたい。  


<BR><BR>

### Chapter05-12 「詳細表示機能のSpecを追加する」  

---

自動テストの機能を追加していく。  
続いて、詳細表示機能のSpecも作ってみる。  

```rb
# spec/system/tasks_spec.rb

require 'rails_helper'

describe 'タスク管理機能', type: :system do
  describe '一覧表示機能' do
    # ここでletを使う
    let(:user_a){ FactoryBot.create(:user, name: 'ユーザーA', email: 'a@example.com') }
    let(:user_b){ FactoryBot.create(:user, name: 'ユーザーB', email: 'b@example.com') }
    # beforeの前に呼び出したいため、「let!」を使用
    let!(:task_a){ FactoryBot.create(:task, name: '最初のタスク', user: user_a) }

    〜 省略 〜

    describe '詳細表示機能' do
      context 'ユーザーAがログインしているとき' do
        # ユーザーAでログイン（使用するログインユーザーをletで定義）
        let(:login_user){ user_a }
      
      before do
        # task_aをユーザーAのタスクとして作成し、task_pathにログイン
        visit task_path(task_a)
      end

      it 'ユーザーAが作成したタスクが表示される' do
        expect(page).to have_content '最初のタスク'
      end 
    end 
    
  end
end

```

試してみる。  
すると、無事３つの機能について自動テストを行うことができ、失敗はなしとなった。  

```
Finished in 5.19 seconds (files took 3.79 seconds to load)
3 examples, 0 failures
```

<BR><BR>

### Chapter05-13 「shared_examplesを利用する」  

---

続いて、ユーザーAの一覧表示機能及び詳細表示機能のコードが同一であるため、  
shared_examplesを使って、itで始まるコードの共通化を行う。  

なお、共通化するのは下記のコード。  

```rb
it 'ユーザーAが作成したタスクが表示される' do
  # 作成済のタスクの名称が画面上に表示されていることを確認
  expect(page).to have_content '最初のタスク'
end
```

具体的には「shared_examples_for ’タイトル’」で囲み、  
その後に「it_behaves_like ’タイトル’」で使用する。  

共通化した後、コードは下記のとおりとなる。  

```rb
# spec/system/tasks_spec.rb
# 該当部分のみ抜粋

  # shared_examples_for を使って it から始まるコードを共通化する
  shared_examples_for 'ユーザーAが作成したタスクが表示される' do
    it { expect(page).to have_content '最初のタスク' }
  end

  describe '一覧表示機能' do
    context 'ユーザーAがログインしているとき' do
      # ユーザーAでログイン（使用するログインユーザーをletで定義）
      let(:login_user){ user_a }

      # shared〜で囲んだ部分をここで使う
      it_behaves_like 'ユーザーAが作成したタスクが表示される'

    end

    context 'ユーザーBがログインしているとき' do
      〜 省略 〜
    end
  end

  describe '詳細表示機能' do
    context 'ユーザーAがログインしているとき' do
      let(:login_user){ user_a }
    
      before do
        visit task_path(task_a)
      end

      # shared〜で囲んだ部分をここで使う
      it_behaves_like 'ユーザーAが作成したタスクが表示される'

    end   
  end
end
```

自動テストのエラーは起きず、無事共通化に成功。  
なお、beforeやletなどをまとめる機能として、shared context という機能がある。  

<BR><BR>

### Chapter05-14 「新規作成機能のSystem Spec」  
### Chapter05-15 「Letの上書き」  

---

続いて、以下のタスクを新規で作った場合、うまくバリデーションが機能するか確認するための自動テストを作成する。  
- 「掃除をする」というタスク（文字あり）
- 「　」というタスク（文字なし、つまりnil）

```rb
# spec/system/tasks_spec.rb
# 該当部分のみ抜粋

  describe '新規作成機能' do
    let(:login_user){ user_a }

    before do
      # タスク新規作成画面へ
      visit new_task_path
      # 後に出てくるletから、タスクの内容を持ってくる
      fill_in '名称', with: task_name
      click_button '登録する'
    end

    context '新規作成画面で名称を入力したとき' do
      let(:task_name){'掃除をする'}

      # have_selector を使って、Flashメッセージに「掃除をする」と言う文字が含まれるか確認
      it '正常に登録される' do
        expect(page).to have_selector '.alert-success', text: '掃除をする'
      end

    end

    context '新規作成画面で名称を入力しなかったとき' do
      let(:task_name){''}

      # have_selector を使って、Flashメッセージに「掃除をする」と言う文字が含まれるか確認
      it 'エラーになる' do
        within '#error_explanation' do
          expect(page).to have_content '名称を入力してください'
        end
      end

    end

  end

end

```

本来、エラーが発生してはいけないはずだが、１件発生。  

```rb
Failures:

  1) タスク管理機能 新規作成機能 新規作成画面で名称を入力しなかったとき エラーになる
     Failure/Error:
       within '#error_explanation' do
         expect(page).to have_content '名称を入力してください'
       end
     
     Capybara::ElementNotFound:
       Unable to find css "#error_explanation"
     
     [Screenshot]: tmp/screenshots/failures_r_spec_example_groups_nested_nested_3_nested_2_エラーになる_573.png

     
     # ./spec/system/tasks_spec.rb:86:in `block (4 levels) in <top (required)>'
```

説明もしてくれるので、非常に分かりやすい。  
エラーとなったスクショを確認。  

<a href="https://gyazo.com/fd8bb817d0142899384fa8606b011028"><img src="https://i.gyazo.com/fd8bb817d0142899384fa8606b011028.png" alt="Image from Gyazo" width="550" border=1/></a>  

こんなにガッツリ、「名称を入力してください」とあるにもかかわらず、  
エラーとして認識されるのは、エラーメッセージに記載のとおり、  
「Unable to find css "#error_explanation"」ということだから。  

じっくり見ていくと、コードの記入漏れを発見。  
slimだとidを書く際に「#」をつけるとこの時に初めて知る。  
コメントじゃなかったんですね。。。  

そこを修正すると、無事エラーは消えた。  

なお、以上のコードでは、let(:task_name)が２回出てくるが、その箇所については  
上書きされているとの説明があった。  




<BR><BR>

### Chapter05-16 「RSpecが失敗したときの調査方法」  

---

幸か不幸か、これまでエラーに遭遇してきていたので、  
既にもう実践してきたような内容が掲載されていた。  

Rails Console を使う発想はなかったので、モデル関係のエラーが予想されるような  
場合などには積極的に活用していきたい。  


