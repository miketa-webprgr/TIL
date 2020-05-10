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

```
# config/routes.rb
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

```
# index.html.slim
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

```
# tasks_controller.rb
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

```
# new.html.slim
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

```
# tasks_controller.rb
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
メンターさんに質問してみる！！！ 質問については、下記のとおり。  

[質問：Form_withのトラブル（controller・Routers.rbかも）](https://github.com/miketa-webprgr/TIL/blob/master/Genba_Rails/Quesiton-Chapter03-3.md)  

というか、自分で書いていたとおり、ここが明らかにおかしい。。。  

> フォームが '/tasks/new' にPOSTメソッドで飛ばすので、createアクションが実行される。  
> （/tasksに飛んでいるわけではないが、/tasks以下にPOSTで飛んでいるので適用される）  

そもそもなぜ '/tasks' にPOSTで飛ばず、'/tasks/new'にPOSTで飛ぶのか。  
form_withについて、railsは「@task = Task.new」と空のオブジェクトであることを認識して、  
createアクションに飛ぶように出来ているとこのこと。  
(「resources :tasks」と設定したから)  

試しに以下のよう書き換えてみる。  
すると、フォームが'/tasks' にPOSTで飛ぶように！  

```
# new.html.slim
# フォーム作成のコードのみ記載し、他を省略する
# @task を Task.new に変換してみた
= form_with model: Task.new , local: true do |f|
  .form-group
    = f.label :name
    = f.text_field :name, class: 'form-control', id: 'task_name'
  .form-group
    = f.label :description
    = f.text_area :description, rows: 5, class: 'form-control', id: 'task_description'
  =f.submit nil, class: 'btn btn-primary'
```

理由はよく分からない。  
原因を探ろうと、また Task.new を @task に戻してみる。  

すると、なぜか今度は'/tasks' にPOSTで飛ぶように。  

不思議であるが、ここに拘るよりも新しいことを学ぶ方が重要だ。  
次に進もう。  

<br>

### Flashメッセージ

---

Railsでは、redirect_toを使ってFlashメッセージを渡すことができる。  
詳細は後ほど勉強していくが、sessionを使って実現しているとのこと。  

調べたところによると、HTTPレスポンスを返す際にそのレスポンスヘッダーに  
そのメッセージを仕込ませるという処理を行っている。ということらしい。  

ここで、今までしてきたHTTPの勉強が生きてくる。
あのレスポンスヘッダーか。しかも、通信がステートレスだから、一度の通信で消える。    
そんな感じだろうか。  

redirect_toの場合、簡潔にflashのnoticeを使えることができるとのこと。
以下のコードはどちらも同じ意味となる。  

以下に詳しい解説がある。
[Pikawaka: 【Rails】完全保存版！flashの使い方についてを徹底解説！](https://pikawaka.com/rails/flash)  

```
redirect_to tasks_url, notice: "タスク「#(task.name)」"を登録しました。"
```
  
```
flash[:notice] = "タスク「#(task.name)」"を登録しました。"
redirect_to tasks_url
```

デフォルトでは:noticeと:alertのみしかないが、任意のFlashキーを追加できるとのこと。  

また、redirect_toではなくてrenderする際にメッセージを出したい場合、
flash.nowを使えばよいらしい。おそらく、以下のようにする。（試していません。。。）  

```
flash.now[:notice] = "タスク「#(task.name)」"を登録しました。"
render tasks_url
```

さて、指示されているように、application.html.slimにコードを書いてみる。  
そして、flashメッセージが出るか試してみる。  

よし、来た！  
<a href="https://gyazo.com/e4cb0c47676f3240ee20070e0c5f617c"><img src="https://i.gyazo.com/e4cb0c47676f3240ee20070e0c5f617c.png" alt="Image from Gyazo" width="542" border=1></a>  

<br>

<br>

### 一覧表示機能を実装する

---

さて、機能を実装するので、アクションの流れを確認する。  

- indexアクションでindex.html.slimに戻る
  - indexアクションに変更を加えて、Taskオブジェクトのデータを全て取得する
  - index.html.slimに変更を加えて、取得データが表示されるようにする

```
# tasks_controller.rb
def index
  @tasks = Task.all
end 
```

Taskオブジェクトの格納先は、@tasksというインスタンス変数とする。  
復習になるが「@」を付けることで、違うメソッドでも@tasksを引っ張ってこれる。  

ビューファイルだが、下記のとおり修正を加える。  

```
# index.html.slim
.mb-3
table.table.table-hover
  thead.thead-default
    tr
      th= Task.human_attribute_name(:name)
      th= Task.human_attribute_name(:created)
    tbody
      - @tasks.each do |t|
        tr
          td= t.name
          td= t.created_at
```

Task.human_attribute_name(:name)について調べる。  
以下の記事を参考した。  

[Railsのモデル名.human_attribute_name(:カラム名)って何だっけ？](https://fuqda.hatenablog.com/entry/2019/04/07/212254)  

要は、ymlにて定義しているnameを参照し、テーブルヘッダーを日本語で表示してくれるらしい。  

「.mb-3」などは、おそらくbootstrap関係なので割愛する。  

実装がうまくいったか確認する。  
<a href="https://gyazo.com/07c69480f88ec88f95bbc04fdceb1cbb"><img src="https://i.gyazo.com/07c69480f88ec88f95bbc04fdceb1cbb.png" alt="Image from Gyazo" width="550" border=1></a>  

<br>

### 詳細表示機能を実装する

---

またアクションの流れを確認する。  

- index.html.slimから、テーブルに記載されている該当の名称をクリック
- showアクションに飛び、該当のデータを探し出す
- show.html.slimに飛び、取得データの詳細情報が確認できる。

まず、index.html.slimにリンクを作成する。  
リンクの作成には、link_toを使う。  

コードは以下のとおりとなる。
```
# index.html.slim
# 該当部分を抜粋
    tbody
      - @tasks.each do |t|
        tr
          td= link_to t.name, t
```

link_toの書き方の基本は以下のとおり。  

```
<%= link_to "テキスト", "リンク先のパス" %>
```

ここから、t.nameが実際に画面に表示される名称であることが分かる。  
問題は「t」の部分である。tオブジェクトから railsが推測して作ってくれるとのこと。  

このあたりの仕組みが、調べたけれどもよく分からない。  
root_pathを使う書き方の省略形みたいなことだろうか。  

なお、root_pathを使う書き方は下記のとおり。  

```
# index.html.slim
# 該当部分を抜粋
    tbody
      - @tasks.each do |t|
        tr
          td= link_to t.name, task_path(t)
```

pikawakaに解説がある。  
[【Rails】link_toの使い方を徹底解説！(Prefixとはの部分)](https://pikawaka.com/rails/link_to#Prefixとは？)  

<a href="https://gyazo.com/f803a306129903f8dd813e9f6ae56fc8"><img src="https://i.gyazo.com/f803a306129903f8dd813e9f6ae56fc8.png" alt="Image from Gyazo" width="344" border=1></a>

<br>

このPrefixの下に書いてある文字+「_path」を書くと、対応するURLを呼び出してくれるらしい。  
ということは、他のURLを呼び出したければ、new_taskやedit_taskも使えると。  
その後に () を付けることで、変数やインスタンス変数を呼び出すことも可能。  

root_pathについては、既に記載したresourcesの記事などに書いてある。  
[Qiita：Rails resourcesメソッドとresourceメソッド](https://qiita.com/Tamitchao/items/6f45aa6daf1412b78d10)  

じゃあ、showアクションに移ろう。

```
# tasks_controller.rb
def show
  @task = Task.find(params[:id])
end
```

taskというインスタンス変数に該当のidのデータを格納する。  
このidというパラメータであるが、showアクションがGETメソッドであるため、  
URLを経由してそのidが引っ張ってこられている。  

index.html.slimにて、`http://~/tasks/:id`へのリンクが作成されているため、
この:idがパラメータとして引っ張ってこられている、ということだ。  

ここはGETとPOSTの違いを改めて確認すると、より理解が深まる。  
[GET、POSTについてわかりやすく解説してみた](https://qiita.com/ryokky59/items/bba97cbfaa899b03e071)  

さて、最後に詳細を表示する画面を作成していく。  
具体的には、show.html.slimにコードを加ていく。

```
# show.html.slim
h1 タスクの詳細

.nav.justify-content-end
  = link_to '一覧', tasks_path, class: 'nav-link'

table.table.table-hover
  tbody
  tr
    th= Task.human_attribute_name(:id)
    td= @task.id
  tr
    th= Task.human_attribute_name(:name)
    td= @task.name
  tr
    th= Task.human_attribute_name(:description)
    # ここの書き方だけ、難しい。。。
    td= simple_format(h(@task.description),{}, sanitize: false, wrapper_tag: "div")
  tr
    th= Task.human_attribute_name(:created_at)
    td= @task.created_at
  tr
    th= Task.human_attribute_name(:updated_at)
    td= @task.updated_at
```

simple_formatの部分だが、なぜこうしているのか。  
たしかめるために、あえてこう書いてみる。

```
# show.html.slim
# 該当部分以外を省略

  tr
    th= Task.human_attribute_name(:description)
    td= @task.description
```

画面とコードの比較を以下に貼付する。  

<br>

<a href="https://gyazo.com/f86c18c827e9ce64abc81b0cf7cb81c3"><img src="https://i.gyazo.com/f86c18c827e9ce64abc81b0cf7cb81c3.png" alt="Image from Gyazo" height="300" border=1></a> <a href="https://gyazo.com/b66c4fcbe5f7d9278b740244a1a99c71"><img src="https://i.gyazo.com/b66c4fcbe5f7d9278b740244a1a99c71.png" alt="Image from Gyazo" height="300" border=1></a>  

<br>

<a href="https://gyazo.com/1230db6ed8a156eabbd5cdfd7774aea3"><img src="https://i.gyazo.com/1230db6ed8a156eabbd5cdfd7774aea3.png" alt="Image from Gyazo" height="250" border=1></a> <a href="https://gyazo.com/63fc1f6babca6fd633627c743435523c"><img src="https://i.gyazo.com/63fc1f6babca6fd633627c743435523c.png" alt="Image from Gyazo" height="250" border=1></a>  

<br>

simple_formatを使うと、`<br>`タグを追記して、うまく表現してくれることが分かった。  

こちらにまとめがある。  
[Qiita:Railsヘルパーメソッド「simple_format」の使い方](https://qiita.com/KA-ZU-KI/items/2749415d396c2e2a497a)  

<br>

### 編集機能を実装する

---

さて、CRUDのCRまでは実装が終わったので、Updateの機能を追加する。  
例のごとく、アクションの流れを確認する。  

1. index.html.slimにリンクを貼り、editのアクションに飛ばすように設定する。
2. show.html.slimでもリンクを貼り、editのアクションに飛ばすように設定する。
3. editアクションに飛ぶと、edit.html.slim（編集画面）を開くようにする。
4. 編集を終えて「登録する」をクリックすると、updateアクションに飛ばすように設定する。
5. updateアクションにて、データを更新し、index.html.slimの画面に戻す。  

こんな感じだろうか。　　

<br>

#### １番と２番に取り掛かる（editアクションに飛ぶようにリンクを作る）
---
まず、１番と２番の作業を行っていく。  
リンクの作成に取り掛かる。  

```
# index.html.slim
# 該当部分のみ抜粋
# 最終行を追加

  td= link_to t.name, task_path(t)
  td= t.created_at
  td= link_to '編集', edit_task_path(t), class: 'btn btn-primary mr-3'
```  

```
# show.html.slim
# 該当部分のみ抜粋
# 最終行として追加

= link_to '編集', edit_task_path, class: 'btn btn-primary mr-3'
```

なお、index.html.slimでは、edit_task_pathメソッドにて変数「t」が引数となっている。　　
ここは、t.idとなっていないが、tと入力されるだけで勝手にrailsが推測し、
params[:id]を取得し、EDITアクションを起こし、GETメソッドで ’tasks/:id/edit’ にアクセスしている。  

また、show.html.slimにあたっては引数がない。
ここでは更にすごくて、そもそも引数を入力することなく、自動的にリンクを作成していくれる。  
それは、前回と違って、何の入力がなくとも、@task.idからparams[:id]を取得することが推測できるからだ。  

ちなみに、index.html.slimをわざわざ「t → t.id」としてもエラーは起きないし、  
show.html.slimを「edit_task_path → edit_task_path(@task.id)」としてもエラーは起きない。  

この推測機能はすごいが、初学者には省略が多すぎて混乱の元になる。。。  

<br>

#### ３番と４番に取り掛かる（edit.html.slimの作成＋updateアクションへのリンク作成）
---

さて、３番に取り掛かる。editアクションの作成だ。  
show.html.slimと同じように、該当データを引っ張ってくる処理を書けばよいので、端的にshowアクションをコピペすればよい。  

```
# tasks_controller.rb

  def edit
    @task = Task.find(params[:id])
  end
```

よし、次にedit.html.slimの作成だ。  
new.html.slimと同じような画面にしたいので、h1以外は変更がない。  

```
# edit.html.slim
h1 タスクの編集

.nav.justify-content-end
  = link_to '一覧', tasks_path, class: 'nav-link'

= form_with model: @task, local: true do |f|
  .form-group
    = f.label :name
    = f.text_field :name, class: 'form-control', id: 'task_name'
  .form-group
    = f.label :description
    = f.text_area :description, rows: 5, class: 'form-control', id: 'task_description'
  = f.submit nil, class: 'btn btn-primary'
```

なお、new.html.slimと異なり、ボタンを押した際の呼び起こすアクションが異なる。  
new.html.slimの場合、createアクションを呼び起こす必要があったが、  
今回はeditアクションを呼び起こす必要がある。  

では、なぜコードは一緒で構わないのか。  
それは、以下に記載があるが、@taskが空でない場合、自動でupdateアクションを呼ぶ仕様となっているからだ。  
奥が深い. これが DRY(Dont't Repeat Yourself)ってやつなのか！？   

[Qiita：【Rails】form_with/form_forについて【入門】](https://qiita.com/snskOgata/items/44d32a06045e6a52d11c)  

<br>

#### ５番に取り掛かる（updateアクションの作成）
---

さて、updateアクションを作成する。  
やることはcreateアクションと近い。参考にするとよい。  

```
# tasks_controller.rb

  def update
    task = Task.find(params[:id])
    task.update!(task_params)
    redirect_to tasks_url, notice: "タスク「#{task.name}」を更新しました。"
  end

# 以下、不正な入力を防ぐためのストロングパラメータに関する記述

  private

  def task_params
    params.require(:task).permit(:name, :description)
  end
```

まず、URLから該当のidのデータをtaskというインスタンスに代入する。  
次に、DB更新するため、update!メソッドを利用する。  
ここで、task_paramsメソッドを呼び出し、不正な入力を防ぐ。  
そして、index.html.slimにリダイレクトさせる。  

以下のとおり、編集機能が実装できた。  

<br>

<a href="https://gyazo.com/3cb196c370bad135272c5b6f672a8bb5"><img src="https://i.gyazo.com/3cb196c370bad135272c5b6f672a8bb5.gif" alt="Image from Gyazo" width="550" border=1/></a>  

<br>


### 新規登録画面と編集画面の共有化（パーシャルの利用）

---

入力フォーム部分が同じであるため、その部分を共通化する。  
共通化した「_form.html.slim」というファイルを作成しい、renderメソッドで読み込む。  

なお、共通化した部分のファイルをパーシャルテンプレートといい、ファイル名をアンダースコアで始める。  
また、読み込む場合、アンダースコアを付けない名前を用いる。  

```
# _form.html.slim
# 共通化する部分
# 「model:@task」を「model:task」に変更

= form_with model: task, local: true do |f|
  .form-group
    = f.label :name
    = f.text_field :name, class: 'form-control', id: 'task_name'
  .form-group
    = f.label :description
    = f.text_area :description, rows: 5, class: 'form-control', id: 'task_description'
  = f.submit nil, class: 'btn btn-primary'
```

```
# new.html.slim + edit.htm.slim に加えるコード

= render partial: 'form', locals: {task: @task}
```

なお、taskと書き換えるような措置を行うのは、インスタンス変数の定義に依存しない、再利用生の高いパーシャルにするためとのこと。  
こちらに解説がある。コントローラがどのビューと結びついているか分かりづらくなることなどがあるとのこと。  

[Qiita: partialではインスタンス変数を参照しない方がいい](https://qiita.com/mom0tomo/items/e1e3fd29729b2d112a48)  

また、こちらに細かい解説がある。  
[Pikawaka: 【Rails】部分テンプレートの使い方を徹底解説！](https://pikawaka.com/rails/partial_template)  
い"}
