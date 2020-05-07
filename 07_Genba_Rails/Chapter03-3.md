## 現場Rails Chapter03-03
## 「タスク管理アプリケーションを作ろう」 「コントローラとビュー」
---

<BR><BR>

### アクションについて
---

効率的にコントローラとビューを作成するには、まずアクションに考えるとスムーズとのこと。  
ユーザーがブラウザでURLを叩いたとき、どのような処理をするのか整理しろということらしい。  
（＋同じURLであっても、GETなのかPOSTなのかといったHTTPメソッドごとにも整理する）  

- URL
- HTTPメソッド
- アクション名
- アクションの機能

この辺りを整理するとよいとのこと。  

なお、アクションとは、リクエストに対しての処理を実行するメソッド。  

[Qiita：【Rails】7つのアクションとそれぞれの役割](https://qiita.com/morikuma709/items/5b21e9853c9d6ea70295)  

コントローラで定義される「def index」などで囲まれるメソッドのことらしい。  
routes.rbのファイルでは「get」や「post」などで書かれるやつのことを指す。  

<br>

### アクションを作成する
---

さて、コマンドラインを叩いていく。

コツとしてindexアクションに対しては、index.html.slimというビューファイルにするとよいらしい。  
（HTTPメソッドがGETの場合に限る） 

HTTPメソッドがGETであるアクション４つを作成する。

```
# tasks = コントローラ名、 index, show, new, edit = アクション名
$ bin/rails g controller tasks index show new edit
Running via Spring preloader in process 37100
      # コントローラの作成
      create  app/controllers/tasks_controller.rb
      # ルートの設定
       route  get 'tasks/index'
              get 'tasks/show'
              get 'tasks/new'
              get 'tasks/edit'
      # アクションに対応するビューファイルの作成
      invoke  slim
      create    app/views/tasks
      create    app/views/tasks/index.html.slim
      create    app/views/tasks/show.html.slim
      create    app/views/tasks/new.html.slim
      create    app/views/tasks/edit.html.slim
      # テストファイル？とかが作成された（コーヒーファイルとは・・・）
      invoke  test_unit
      create    test/controllers/tasks_controller_test.rb
      invoke  helper
      create    app/helpers/tasks_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/tasks.coffee
      invoke    scss
      create      app/assets/stylesheets/tasks.scss
```

なお、現場railsでは、ルーティングを他のアクションと合わせて一括で設定するので、  
先ほどのルートの設定を一旦削除しろとのことなので、従う。  

続いて、以下を記述。  
これでパターンどおりの設定が一括で行えるらしい。  

```:config/routes.rb
# 一度、get ~というルートの設定を削除した上で

resources :tasks
```

え。。。短くない！？  
不安なので、調べてみる。  


[Qiita：Rails resourcesメソッドとresourceメソッド](https://qiita.com/Tamitchao/items/6f45aa6daf1412b78d10)  

要は、railsの規約に従って、勝手に設定してくれるということらしい。  
細かいところは、後ほどまた勉強するとしよう。  

また、指示どおり初期表示画面の設定変更も行う。  

無事に表示された。フォントの感じからして、bootstrpも適用されているっぽい。  
<br>

<img src="https://i.gyazo.com/d8e710f9a601a36a91a356bb7a1ff9cc.png" height=300px border=1>  

<br>

### 新規登録機能の実装
---

新規登録機能の実装を行っていく。  
まずアクションの流れを整理する。  

1. indexアクション（初期画面）
2. newアクション（新規登録画面へ移行）
3. createアクション（登録情報の保存→初期画面へ戻る）

また、各画面で次のアクションに移れるように、リンクも設定していく。  

<br>

###  新規登録リンクの追加
---

さて、早速index.html.slimにnewアクションへのリンクを加えていく。

```index.html.slim
h1 タスク一覧

/ rubyコードを書く時は「=」（画面出力される場合）もしくは「-」（画面出力させない場合）
/ new_task_path はリンク先を指している。URLヘルパーメソッドを使って、URLを呼び出している
/ classの部分はbootstrapのスタイルを適用している
= link_to '新規登録', new_task_path, class: 'btn btn-primary'
```

<br>

### モデルの翻訳情報を追加
---

gemを導入した影響で、そもそもjp.ymlファイルがない。。。  
とりあえず、スルーしていく。

必要であれば、jp.ymlファイルを作成して、翻訳情報を追加していこう。

<br>

### 新規登録画面のためのアクションを実装する
---

さて、newアクションへのリンクは作成したので、  
登録データの新規作成処理コードをコントローラに追記する。  

```tasks_controller.rb
class TasksController < ApplicationController
 # 他のアクションも書いてあるが省略
  def new
    ＠task = Task.new
  end
```


<br>

### アクションへデータを送る「リクエストパラメータ」
---

解説が入る。  

リクエスト（クライアントサイドからサーバーへのデータ送信）では、データを添えられる。  
GETを使う方法とPOSTを使う方法がある。これはSinatraアプリの自作で何度も使った。  

- GETの場合　→　URLを通してパラメータを送信する、もしくはメソッドをgetに指定してフォームで送信。  
- POSTの場合　→　HTMLのinputタグで囲ったデータを送信する。  

<br><br>

### 新規登録画面のビューを実装する
---

先ほどインスタンス変数「@task」に格納した、新しいタスクのオブジェクト「Task.new」を利用して、  
登録フォームを表示する。コードはRubyで書いていく。form_withメソッドを使うとのこと。

```new.html.slim
h1 タスクの新規登録

# これはindex.html.slimに戻るためのコード
.nav.justify-content-end
  = link_to '一覧', tasks_path, class: 'nav-link'

# ここからがフォーム作成のコード
= form_with model: @task, local: true do |f|
  .form-group
    = f.label :name
    = f.text_field :name, class: 'form-control', id: 'task_name'
  .form-group
    = f.label :description
    = f.text_area :description, rows: 5, class: 'form-control', id: 'task_description'
  =f.submit nil, class: 'btn btn-primary'
```

タイプミスもあったが、無事、新規登録画面の表示できた。  
<br>  

<img src="https://i.gyazo.com/d61eb6a4f6b777ffb6874db8744dbcec.png" height=300px border=1>

さて、解説を確認する。  
> from_withは、モデルオブジェクトをつかってHTMLのform要素を作成するためのメソッドです。  
> モデルとフォームは、モデルの属性が・・・

なるほど、分からん。  
ググってみると、form_withについての解説を発見。  

[【Rails】form_withの使い方を徹底解説！](https://pikawaka.com/rails/form_with)  

解説されていたが、データベースに保存する時の記述は以下のとおりとのこと。  
データベースに保存する時は、modelって書いて、そのあとインスタンス変数を書く。  
そうやって言ってくれると分かりやすいんだけどなー。

```
# コピペのため、slim形式ではない
<%= form_with model: モデルクラスのインスタンス do |form| %>
  フォーム内容
<% end %>
```

ちなみに「local:true」はHTMLとしてフォーム送信する場合は必要になるらしい。  
[Qiita：【Rails】form_with (local: true)について](https://qiita.com/hayulu/items/5bf26656d7433d406ede)  

ja.yml関係を設定するとNameとかDescriptionが日本語になるのだろうが、  
ここは一旦スルーする。  

さて、次が「.form-group」以下だ。  
ちなみに、「.form-group」はbootsrapで指定されているスタイル。  

HTMLとの対応を見てみると分かりやすいと思うので、そのまま貼付する。  
bootstrap関係のdivタグがあると分かりづらいので、そこは省略する。

```
<form action="/tasks/new" accept-charset="UTF-8" method="post">
    <input name="utf8" type="hidden" value="&#x2713;" />
    <input type="hidden" name="authenticity_token" value="moNQ~~~==" />
#   ちなみに、authenticity_tokenのコードは、CSRF対策らしい。。。。
#   リロードしたら、valueの値が変わった。これがハッシュ値ってやつなのか。。。
# --------------------------------------------------------------------
#  .form-group
#   = f.label :name
#   = f.text_field :name, class: 'form-control', id: 'task_name'
# --------------------------------------------------------------------
    <label for="name">Name</label>
    <input class="form-control" id="task_name" type="text" name="name" />
# --------------------------------------------------------------------
#  .form-group
#   = f.label :description
#   = f.text_area :description, rows: 5, class: 'form-control', id: 'task_description'
# --------------------------------------------------------------------
    <label for="description">Description</label>
    <textarea rows="5" class="form-control" id="task_description" name="description"></textarea>
# --------------------------------------------------------------------
    <input type="submit" name="commit" value="保存する" class="btn btn-primary" data-disable-with="保存する" />
</form>
```

<br>

### 登録アクションの実装 
---

さて、ビューファイルを作ったので、次は「登録する」ボタンを押した時のアクションを作成する。  
コントローラファイルを開く。  

ここでresourcesについて調べた際にお世話になったQiitaの記事を見てみる。  
アクションファイル名をどう書くべきか確認する。  

[Qiita：Rails resourcesメソッドとresourceメソッド](https://qiita.com/Tamitchao/items/6f45aa6daf1412b78d10)  

| URL                       | アクション | HTTPメソッド | 動作                |
|:--------------------------|:--------|:----------|:------------------ |
| /tasks(.:format)          | index   | GET       | ユーザ一覧画面を表示   |
| /tasks(.:format)          | create  | POST      | ユーザの登録処理      |
| /tasks/new(.:format)      | new     | GET       | ユーザの登録画面を表示 |
| /tasks/:id/edit(.:format) | edit    | GET       | ユーザの編集画面を表示 |
| /tasks/:id(.:format)      | show    | GET       | ユーザの詳細画面を表示 |
| /tasks/:id(.:format)      | update  | PATCH     | ユーザの更新処理      |
| /tasks/:id(.:format)      | update  | PUT       | ユーザの更新処理      |
| /tasks/:id(.:format)      | destroy | DELETE    | ユーザの削除処理      |


また、以下のルビーの「form_with」のコードはHTMLにこのような形で変換される。  
フォームが「/tasks/new」にPOSTメソッドで飛ばすので、createアクションが実行される。  
（/tasksに飛んでいるわけではないが、/tasks以下にPOSTで飛んでいるので適用される）  

```
# form_withのコード 
  = form_with model: @task, local: true do |f|
# HTMLのコード
  <form action="/tasks/new" accept-charset="UTF-8" method="post">
  ~
  </form>
```

・・・前置きが長くなったが、作成するのはcreateアクションということが確認できた。  

現場railsに従って、コードを書いてみた。  
なお、private以降がストロングパラメータに関するコード。  

```tasks_controller.rb
  def create
    task = Task.new(tasks_params)
    task.save!
    redirect_to tasks_url, notice: "タスク「#{task.name」}を登録しました。"
  end

  private

  def tasks_params
    params.require(:task).permit(:name, :description)
  end
```

わざわざTask.newを作り直すのは、HTTPがステートレスであり、  
newアクションで作ったデータベースを保存して呼び出すより簡単だから。  

save!とsaveの違いについては、以下が一番分かりやすかった。  
[【Rails】saveとsave!について](https://masterpiyo.hatenadiary.org/entry/20111212/1323677704)  

トランザクション中にデータを保存したい場合は、save!を使うことが多いとのこと。  
（今回はそのような事例に該当しないような気はするけど。。。）  

redirect_toしているのは、保存したデータを読み込む必要があるから。  
redirect_toは、renderと異なる。  
[【Rails】renderとredirect_toの違いと使い分け](https://qiita.com/morikuma709/items/e9146465df2d8a094d78)

<br>

### Form_withでのトラブル発生

---

ここで、railsサーバーを立ち上げて確認したところ、トラブル発生。
メンターさんに頼る！！！ 質問については、下記のとおり。  

というか、自分で書いていたとおり、ここが明らかにおかしい。。。  

> フォームが「/tasks/new」にPOSTメソッドで飛ばすので、createアクションが実行される。  
> （/tasksに飛んでいるわけではないが、/tasks以下にPOSTで飛んでいるので適用される）  

[質問：Form_withのトラブル（controller・Routers.rbかも）](https://github.com/miketa-webprgr/TIL/blob/master/Genba_Rails/Quesiton-Chapter03-3.md)  