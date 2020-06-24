 # EverydayRails RSpecによるRailsテスト入門

## Chapter 05 コントローラスペック

### 前置き

- 現在では、以下が推奨されている。ただ、知っておくことには意義がある！
  - コントローラのテスト(機能テスト層とも呼ばれます)を削除
  - モデルのテスト(単体テスト)か、より高いレベルの統合テストと置き換える

### HTTPレスポンスが成功するか確認する

```rb: spec/controllers/home_controller_spec.rb
require 'rails_helper'

RSpec.describe HomeController, type: :controller do 
  describe "#index" do
  # 正常にレスポンスを返すこと
    it "responds successfully" do
      get :index
      expect(response).to be_successful
    end
  end
end
```

### HTTPレスポンスコードが特定のものであるか確認する

```rb: spec/controllers/home_controller_spec.rb
require 'rails_helper'

RSpec.describe HomeController, type: :controller do 
  describe "#index" do
  # 正常にレスポンスを返すこと
    it "responds successfully" do
      get :index
      expect(response).to have_http_status "200"
    end
  end
end
```

### 認証が必要なコントローラスペック

```rb: spec/rails_helper.rb
RSpec.configure do |config|
  # 設定ブロックの他の処理は省略 ...

  # コントローラスペックで Devise のテストヘルパーを使用する
  config.include Devise::Test::ControllerHelpers, type: :controller end
end
```

`gem devise`を使っている場合、以上のとおり確認する。  
`has_secure_password`を使っている場合、自分でヘルパーメソッドを設定する。  

なお、ヘルパーにてsigin_inを定義したら、以下のとおり書く。

```rb: spec/controllers/projects_controller_spec.rb
require 'rails_helper'

RSpec.describe ProjectsController, type: :controller do
  describe "#index" do
    # 認証済みのユーザーとして
    context "as an authenticated user" do
      before do
        @user = FactoryBot.create(:user)
      end
  
      # 正常にレスポンスを返すこと 
      it "responds successfully" do
        sign_in @user
        get :index
        expect(response).to be_success
      end
    end
  end
end
```

また、ログインしていないゲストの場合、リダイレクトされる設定となっている。  
その場合、以下のとおりコードを書く。  

```rb: spec/controllers/projects_controller_spec.rb
# RSpec.describe部分は省略

# ゲストとして
context "as a guest" do
  # サインイン画面にリダイレクトすること 
  it "redirects to the sign-in page" do
    get :index
    expect(response).to redirect_to "/users/sign_in" 
  end
end
```

`params`を送る必要がある場合、以下のとおりコードを書く。  
具体的には、show画面などが想定される。  

また、以下の例だと、Userの下に紐づくprojectデータを作成しているので、
参考にするとよい。  

```rb:spec/controllers/projects_controller_spec.rb
require 'rails_helper'

RSpec.describe ProjectsController, type: :controller do
  # インデックスのテストが並ぶ ...
  describe "#show" do
    # 認可されたユーザーとして 
    context "as an authorized user" do
      before do
        @user = FactoryBot.create(:user)
        @project = FactoryBot.create(:project, owner: @user)
      end
    
      # 正常にレスポンスを返すこと
      it "responds successfully" do
        sign_in @user
        get :show, params: { id: @project.id }
        expect(response).to be_success
      end
    end
  
  # 認可されていないユーザーとして
    context "as an unauthorized user" do
      before do
        @user = FactoryBot.create(:user)
        other_user = FactoryBot.create(:user)
        @project = FactoryBot.create(:project, owner: other_user)
      end
    
    # ダッシュボードにリダイレクトすること 
      it "redirects to the dashboard" do
        sign_in @user
        get :show, params: { id: @project.id }
        expect(response).to redirect_to root_path
      end
    end
  end
end
```

### ユーザー入力をテストする

ユーザーが新しくプロジェクトを作成する場合、  
つまり、POSTメソッドを使う場合、以下のとおり書く。  

`attributes for`を使っているところや、`change`マッチャに着目すること。  
`change`マッチャは、そのプロジェクトの数が1つ増えたか確認している。  

```rb: spec/controllers/projects_controller_spec.rb
describe "#create" do
  # 認証済みのユーザーとして
  context "as an authenticated user" do
    before do
      @user = FactoryBot.create(:user)
    end
  
    # プロジェクトを追加できること 
    it "adds a project" do
      project_params = FactoryBot.attributes_for(:project) 
      sign_in @user
      expect {
        post :create, params: { project: project_params }
        }.to change(@user.projects, :count).by(1) 
    end
  end
end
```

また、ここでは細かく記載しないが「Everyday Rails」では、  
PATCHメソッドを使ってアップデートするような場合、reloadというメソッドを使っている。  


### ユーザー入力のエラーをテストする

基本的な手法は、従来どおり。  
詳細は記載しない。

`FacotoryBot`の`trait`で`nil`のデータを送ってエラーになるか確かめたり、  
`change`マッチャを使って、プロジェクトの数が増えないか確認している。

### HTML 以外の出力を扱う

- コントローラが担うべき責務の一つに、適切なフォーマットでデータを返すという役割がある。
- `JSON`に限定したテストを書くとどうなるか

基本的には変わらないが、`get :show`のところでフォーマットを指定し、  
最後にフォーマットが期待した`application/json`になるか確認している。  

```rb: spec/controllers/tasks_controller_spec.rb
require 'rails_helper'

RSpec.describe TasksController, type: :controller do
  before do
    @user = FactoryBot.create(:user)
    @project = FactoryBot.create(:project, owner: @user)
    @task = @project.tasks.create!(name: "Test task")
  end

  describe "#show" do
    # JSON 形式でレスポンスを返すこと
    it "responds with JSON formatted output" do
      sign_in @user
      get :show, format: :json,
      params: { project_id: @project.id, id: @task.id }
      expect(response.content_type).to eq "application/json"
    end 
  end

# 他のテストは省略 ...
end
```

### Q&A

success と http status は両方チェックしないといけませんか?必ずしも必須ではありません。
どちらかひとつで十分な場合もありますが、それはあなたのコントローラが HTTP クライアントに
返すレスポンスの複雑さによります。

## Chapter 06 フィーチャスペックで UI をテストする

統合テスト（Feature Specs → System Specs）
- モデルとコントローラが他のモデルのコントローラとうまく一緒に動作することを確認する

### Capybaraの設定方法

- `capybara`を`Gemifile`のテスト環境部分に書き加え、`bundle install`する
- `spec/rails_helper.rb`を開き、`capybara`のライブラリを追加する

### Capybara の DSL

```rb
# 全種類の HTML 要素を扱う
scenario "works with all kinds of HTML elements" do
  # ページを開く
  visit "/fake/page"
  # リンクまたはボタンのラベルをクリックする
  click_on "A link or button label"
  # チェックボックスのラベルをチェックする
  check "A checkbox label"
  # チェックボックスのラベルのチェックを外す
  uncheck "A checkbox label"
  # ラジオボタンのラベルを選択する
  choose "A radio button label"
  # セレクトメニューからオプションを選択する
  select "An option", from: "A select menu"
  # ファイルアップロードのラベルでファイルを添付する
  attach_file "A file upload label", "/some/file/in/my/test/suite.gif"

  # 指定した CSS に一致する要素が存在することを検証する
  expect(page).to have_css "h2#subheading"
  # 指定したセレクタに一致する要素が存在することを検証する
  expect(page).to have_selector "ul li"
  # 現在のパスが指定されたパスであることを検証する
  expect(page).to have_current_path "/projects/new"
end
```

### セレクタのスコープ

例えば、`rubyonrails.org`をクリックしたい場合、  
以下のとおりコードを書けばよい。  

```html
<div id="node">
  <a href="http://nodejs.org">click here!</a>
</div>
<div id="rails">
  <a href="http://rubyonrails.org">click here!</a>
</div>
```

```rb
within "#rails" do
  click_link "click here!"
end
```

### findメソッド

[Capybaraチートシート \- Qiita](https://qiita.com/morrr/items/0e24251c049180218db4)

```rb
language = find_field("Programming language").value
expect(language).to eq "Ruby"

find("#fine_print").find("#disclaimer").click
find_button("Publish").click
```

### フィーチャスペックをデバッグする

Capybaraは、デフォルトはヘッドレスブラウザ。
ただ、Rails がブラウザ に返した HTML を見ることはできます。  

```rb
scenario "guest adds a project" do
  visit projects_path
  # これを追加するとHTMLが見れる
  save_and_open_page
  click_link "New Project"
end
```

手動で開いてもよいが、`Launchy gem`を使えば自動的に開くようになる。  

### JavaScript を使った操作をテストする

Capybaraのデフォルトのヘッドレスブラウザだと、JSが確認できない。  
その場合、`js: true`というオプションを渡す。  

```rb: spec/features/tasks_spec.rb
require 'rails_helper'

RSpec.feature "Tasks", type: :feature do
  # ユーザーがタスクの状態を切り替える 
  scenario "user toggles a task", js: true do

```

デフォルトだと、`firefox`が使われることとなっているが、  
`chrome`を使いたい場合、以下のとおり対応する必要がある。  

`rails_helper.rb`ファイルはきれいな状態を保っておくため、  
独立したファイルに新しい設定を書く。

```rb: spec/rails_helper.rb
# 以下の部分がコメントアウトされているので、外すこと
Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }
```

続いて、次のような設定を追加する。  

```rb: spec/support/capybara.rb
Capybara.javascript_driver = :selenium_chrome
```

`webdrivers gem`を`Gemfile`に追加する。  

```Gemfile
group :test do
  gem 'capybara', '~> 2.15.4' gem 'webdrivers'
  gem 'launchy', '~> 2.4.3'
end
```

### ヘッドレスドライバを使う

次のようにドライバを変更してください。

```rb: spec/support/capybara.rb
Capybara.javascript_driver = :selenium_chrome_headless
```

### JavaScript の完了を待つ

```rb
# 本当に遅い処理を実行する
scenario "runs a really slow process" do
  using_wait_time(15) do 
    # テストを実行する
  end
end
```

## Chapter 07 リクエストスペックで API をテストする

### そもそもAPIとは

APIを触ったことがなく、amazonやらyoutubeやらとうまく連携して、  
いい感じにアプリを作ることができるサービスという認識しかなかった。  

なので、まずRSpecの勉強をする前に、APIについての理解を深めることとした。  

[WebAPIについての説明 \- Qiita](https://qiita.com/busyoumono99/items/9b5ffd35dd521bafce47)  
[Web APIとは？ （LINE bot API・グルナビAPI） \- Qiita](https://qiita.com/Masato338/items/6fb1ac277c965905e019)  

また、`JSON.parse`というものが出てきたので、調べてみた。  

[JSON\.parse\(\) \- JavaScript \| MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)  

あくまでイメージでの世界でしかないが、WebAPIを使うと、paramsをyoutubeなどのWebサーバーに送ることで、  
youtube側がJSON形式でデータを返してくれるので、それを活用して、いい感じにサービスを構築できるという  
ことが分かった。  

そして、このサンプルプロジェクトでは、APIサーバーも作られており、  
JSON形式でデータを返してくれるということがよく分かった。
（違う形もあるのだろうけど、こういうタイプのものは多いらしい）。

その辺りの制御は、`controllers/api/projects/controller.rb`にて  
行われていることも、大枠が分かっていく中で、自然と理解できた。  

これは、`http://puzzles-engineer.github.io/`の第１問ができないと、
流石に厳しいのではないかと感じた。  

### GET リクエストをテストする

APIのコントローラは、以下のとおりとなっている。  
テストはindex部分について書かれているので、index部分に限定して引用する。  

```rb: controllers/api/projects_controller.rb
module Api
  class ProjectsController < ApplicationController

    prepend_before_action :authenticate_user_from_token!

    def index
      @projects = current_user.projects
      render json: @projects
    end

    private

    def authenticate_user_from_token!
      user_email = params[:user_email].presence
      user = user_email && User.find_by(email: user_email)
      if user && Devise.secure_compare(user.authentication_token, params[:user_token])
        sign_in user, store: true
      else
        render json: { status: "auth failed" }
        false
      end
    end

  end
end
```

```rb: projects_api_spec.rb
describe 'Projects API', type: :request do
  it 'loads a project' do
    user = FactoryBot.create(:user)
    FactoryBot.create(:project, name: "Sample Project")
    FactoryBot.create(:project, name: "Second Sample Project", owner: user)

    # このAPIではユーザーのメールアドレスとサインインするためのトークンが必要になる
    # api_projects_path(indexアクション）にアクセスし、GETメソッドにてparamsを送る
    get api_projects_path, params: {
      user_email: user.email,
      user_token: user.authentication_token
    }

    expect(response).to have_http_status(:success)

    # jsonに格納されるデータは以下のとおり
    # [{"id"=>2, "name"=>"Second Sample Project", "description"=>"A test project.", "due_on"=>"2020-07-01", "created_at"=>"2020-06-24T07:26:53.964Z", "updated_at"=>"2020-06-24T07:26:53.964Z", "user_id"=>1, "completed"=>nil}]
    json = JSON.parse(response.body)

    expect(json.length).to eq 1
    project_id = json[0]["id"]

    get api_project_path(project_id), params: {
      user_email: user.email,
      user_token: user.authentication_token
    }

    expect(response).to have_http_status(:success)
    json = JSON.parse(response.body)
    expect(json["name"]).to eq "Second Sample Project"
  end
end
```

### POST リクエストをテストする

以下のとおりとなる。  
GETと考え方は変わらない。create部分に限定して引用する。  

```rb: controllers/api/projects_controller.rb
module Api
  class ProjectsController < ApplicationController

    prepend_before_action :authenticate_user_from_token!

    def create
      @project = current_user.projects.new(project_params)

      if @project.save
        render json: { status: :created }
      else
        render json: @project.errors, status: :unprocessable_entity
      end
    end

    private

    def authenticate_user_from_token!
      user_email = params[:user_email].presence
      user = user_email && User.find_by(email: user_email)
      if user && Devise.secure_compare(user.authentication_token, params[:user_token])
        sign_in user, store: true
      else
        render json: { status: "auth failed" }
        false
      end
    end

    def project_params
      params.require(:project).permit(:name, :description, :due_on)
    end
  end
end
```

```rb: spec/requests/projects_api_spec.rb
require 'rails_helper'
  describe 'Projects API', type: :request do
  # 最初のサンプルコードは省略 ...
  # プロジェクトを作成できること
  it 'creates a project' do
    user = FactoryBot.create(:user)
    project_attributes = FactoryBot.attributes_for(:project)
    expect {
      post api_projects_path, params: {
        user_email: user.email,
        user_token: user.authentication_token,
        project: project_attributes
      }
    }.to change(user.projects, :count).by(1)

    expect(response).to have_http_status(:success)
  end
end
```

### コントローラスペックをリクエストスペックで置き換える

ここまでのテストは、あくまでAPI部分に関係するテストだったが、
コントローラ部分も含めて、総合的にテストすることができる。  

以下の例では、`sign_in @user`が出てくるが、ログイン機能が動くか、  
総合的にテストすることができる。  

```rb: spec/requests/projects_spec.rb
require 'rails_helper'

RSpec.describe "Projects", type: :request do 
  # 認証済みのユーザーとして
  context "as an authenticated user" do
    before do
      @user = FactoryBot.create(:user)
    end

  # 有効な属性値の場合
  context "with valid attributes" do
    # プロジェクトを追加できること
    it "adds a project" do
      project_params = FactoryBot.attributes_for(:project)
      sign_in @user
      expect {
        post projects_path, params: { project: project_params }
      }.to change(@user.projects, :count).by(1)
    end
  end
end
```

なお、細かい設定方法が書いてあったが、解読するまでには至りそうもなかったので、  
とりあえずは、きちんと設定ができるように頑張りたい。
