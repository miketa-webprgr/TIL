## 現場Rails Chapter07 「機能を追加してみよう」

---

<BR><BR>

### Chapter07-1 「確認画面を挟む」  

---

登録や編集の際に、確認画面を挟みたい。  
具体的には、以下のアクション間に、確認を行うconfirm_newアクションとconfirm_editアクションを挟みたい。  
（練習も兼ねて、現場railsでconfirm_newアクションだけのところ、今回はconfirm_editを追加したい）  

1. newアクションからcreateアクションの間（新規作成画面から登録処理まで）
2. editアクションからupdateアクションの間（編集画面から更新処理まで）

<br>

#### ルーティングについて検討する
---

該当箇所のルーティングを確認する。  

```
           Prefix Verb   URI Pattern                          Controller#Action
             root GET    /                                    tasks#index
            tasks GET    /tasks(.:format)                     tasks#index
                  POST   /tasks(.:format)                     tasks#create
         new_task GET    /tasks/new(.:format)                 tasks#new
        edit_task GET    /tasks/:id/edit(.:format)            tasks#edit
             task GET    /tasks/:id(.:format)                 tasks#show
                  PATCH  /tasks/:id(.:format)                 tasks#update
                  PUT    /tasks/:id(.:format)                 tasks#update
                  DELETE /tasks/:id(.:format)                 tasks#destroy
```

ルーティングについて、どこにconfirm_newとconfirm_editを所属させるか検討する。  
現場railsに従って、tasks/new/confirmにtasks#confirm_newアクションを作成したい。  
また、tasks/:id/confirmにtasks#confirm_editアクションを作成したい。  

<br>

#### 必要な作業を洗い出しする
---

では、confirm_newアクション関係の作業として、何を行う必要があるのか整理する。  
1. newアクションにて新規画面を表示
2. createアクションではなく、confirm_newアクションに飛ばすように変更（new画面のリンクを修正）
3. confirm_newの画面を表示（confirm_newの画面を作成、パラメータ格納についてコントローラに記入）
4. confirm_newにてエラー検証を行う（コントローラに記入）
5. エラーがある場合、newアクションに飛ばす。new画面にてエラーを表示（new画面にてエラー表示させるよう修正）
6. エラーがない場合、new画面で入力されたパラメータをcreateアクションに飛ばす（confirm_new画面のリンクを修正）

では、confirm_editアクションで関係の作業として、何を行う必要があるのか整理する。  
1. editアクションにて編集画面を表示
2. updateアクションではなく、confirm_editアクションに飛ばすように変更（edit画面のリンクを修正）
3. confirm_editの画面を表示（confirm_editの画面を作成、パラメータ格納についてコントローラに記入）
4. confirm_editにてエラー検証を行う（コントローラに記入）
5. エラーがある場合、editアクションに飛ばす。edit画面にてエラーを表示（edit画面にてエラー表示させるよう修正）
6. エラーがない場合、edit画面で入力されたパラメータをupdateアクションに飛ばす（confirm_edit画面のリンクを修正）

その他、DRY（Don't Repeat Yourself）に出来るところは、積極的に挑戦する。（ただし、無理はしない）  

<br>

#### エラーにハマって学んだこと
---

上記にて、ステップを細かく記載したが、当初は作業の洗い出しに漏れや誤りが発生した。  

そのこと自体は問題ではないが、そこから安易にコードの修正や情報検索ばかりに走ってしまい、  
結局、行き当たりばったりのトライアンドエラーの繰り返しになってしまった。

プログラムは、どこか数学や実験に似ている。  
少なくとも慣れるまでは、手順を踏んでコツコツとやるのが重要だ。  

自分の脳味噌には限界がある。
このことをよく肝に命じて、何をやっているかきちんと記録していくこと。  
（どうせ大概の場合、またエラーになるのだから）  

躍起になってガチャガチャやり過ぎて、自分を失ってしまう前に冷静になること。  
時間がかかりそうでも、分析して、全体の見取り図を作っていくこと。  

<br>

#### 参考にしていく・振り返るべき記事
----

チェリー本の著者である伊藤さんのQiita記事を発見したが、  
今後エラーに遭遇して、解決の見込みが立たなければ、即この記事を眺めること。  

[プログラミング初心者歓迎！「エラーが出ました。どうすればいいですか？」から卒業するための基本と極意（解説動画付き） \- Qiita](https://qiita.com/jnchito/items/056325421b7e36f02335)  

あと、改めてこちらの記事の偉大さを実感。  
具体的に書いてある。  

[RUNTEQの講師をやってみてわかった初学者にありがちなパターン20選（前編） \- Qiita](https://qiita.com/DaichiSaito/items/52448ebfcb0db768dcf3)  

<br>


#### ルーティングを設定する
---

まず、目標を確認。  

URL        tasks/new/confirm  
アクション   tasks#confirm_new  
パス名      confirm_new_task

URL        tasks/:id/confirm  
アクション   tasks#confirm_edit  
パス名      confirm_edit_task

なお、routes.rbには以下のとおり記載。  

```rb
# routes.rb
# 該当部分のみ記載

  resources :tasks do 
    post :confirm, action: :confirm_new, on: :new
    patch :confirm, action: :confirm_edit, on: :member, as: 'confirm_edit'
  end
```

なお、コードの記載にあたっては、以下を参照した。  

[Rails のルーティング \- Railsガイド](https://railsguides.jp/routing.html)  

[railsのroutes\.rbのmemberとcollectionの違いをわかりやすく解説してみた。〜rails初心者から中級者へ〜 \- Qiita](https://qiita.com/hirokihello/items/fa82863ab10a3052d2ff#comments)  

以下のとおり、ルーティング設定が完了した。  
この練習を通して、かなりルーティングについて分かってきた気がする！  

```
confirm_new_task   POST   /tasks/new/confirm(.:format)    tasks#confirm_new
confirm_edit_task  PATCH  /tasks/:id/confirm(.:format)    tasks#confirm_edit
```

<br>

#### 作業1・2・3・4
---

先ほど書いた作業リストの内、１〜３を行う。

confirm_new  
1. newアクションにて新規画面を表示
2. createアクションではなく、confirm_newアクションに飛ばすように変更（new画面のリンクを修正）
3. confirm_newの画面を表示（confirm_newの画面を作成、パラメータ格納についてコントローラに記入）
4. confirm_newにてエラー検証を行う（コントローラに記入）

confirm_edit  
1. editアクションにて編集画面を表示
2. updateアクションではなく、confirm_editアクションに飛ばすように変更（edit画面のリンクを修正）
3. confirm_editの画面を表示（confirm_editの画面を作成、パラメータ格納についてコントローラに記入）
4. confirm_editにてエラー検証を行う（コントローラに記入）

なお、confirm_newもconfirm_editも画面に違いがほぼないので、パーシャルを活用する。  

<br>

#### createではなくconfirm_new、updateではなくconfirm_editに飛ばす  
---

```rb
# new.html.slim
# 該当部分のみ記載
# formでのsubmit後のアクセス先について、以下のとおりconfirm_newアクションを指定

= form_with model: @task, local: true, url: {controller: 'tasks', action: 'confirm_new' } do |f|
```

<br>

```rb
# edit.html.slim
# 該当部分のみ記載
# formでのsubmit後のアクセス先について、以下のとおりconfirm_editアクションを指定

= form_with model: @task, local: true, url: {controller: 'tasks', action: 'confirm_edit' } do |f|
```

#### confirm_newとconfirm_editの画面を表示  
---

newもeditも、h1の内容を除いて違いはない。

```rb
# confirm_new.html.slim

h1 登録内容の確認

= render partial: 'confirm', locals: { task: @task }
```

```rb
# confirm_edit.html.slim

h1 修正内容の確認

= render partial: 'confirm', locals: { task: @task }
```

```rb
# _confirm_html.slim

= form_with model: task, local: true do |f|
  table.table.table-hover
    tbody  
      tr
        th= Task.human_attribute_name(:name)
        td= task.name
        # ユーザーから見えない形で決まった値を送るためにhidden_fieldを使う
        = f.hidden_field :name
      tr
        th= Task.human_attribute_name(:description)
        td= simple_format(task.description)
        # ユーザーから見えない形で決まった値を送るためにhidden_fieldを使う
        = f.hidden_field :description
  = f.submit '戻る', name: 'back', class: 'btn btn-secondary mr-3'
  = f.submit '登録', class: 'btn btn-primary'
```

<br>

#### パラメータ格納についてコントローラに記入（ついでにエラーの時の処理も追記）
---

```rb
# tasks_controller.rb

class TasksController < ApplicationController
  # :confirm_editを追加
  before_action :set_task, only: [:show, :edit, :update, :destroy, :confirm_edit]

  ~ 省略 ~

  def confirm_new
    # current_userのtasksで新しいオブジェクトを作成
    @task = current_user.tasks.new(task_params)
    # errorになればnewアクションに戻し、エラーを表示させる
    # errorについては後ほどコードを書く
    render :new unless @task.valid?
  end
  
  def confirm_edit
    # errorになればnewアクションに戻し、エラーを表示させる
    # errorについては後ほどコードを書く
    
    # @taskにTaskモデルから該当のidのものを格納しているので、paramsで書き換える
    # assign_attributesは、DBを更新せずに@taskを更新するメソッド
    @task.assign_attributes(task_params)
    render :new unless @task.valid?
  end

  private

  def set_task
    @task = current_user.tasks.find(params[:id])
  end
```

assign_attributesメソッドについては下記のとおり。  
[ActiveRecord の attribute 更新方法まとめ \- Qiita](https://qiita.com/tyamagu2/items/8abd93bb7ab0424cf084)  

<br>

#### 作業5・6
---

confirm_new
5. エラーがある場合、newアクションに飛ばす。new画面にてエラーを表示（new画面にてエラー表示させるよう修正）
6. エラーがない場合、new画面で入力されたパラメータをcreateアクションに飛ばす（confirm_new画面のリンクを修正）

confirm_edit
5. エラーがある場合、editアクションに飛ばす。edit画面にてエラーを表示（edit画面にてエラー表示させるよう修正）
6. エラーがない場合、edit画面で入力されたパラメータをupdateアクションに飛ばす（confirm_edit画面のリンクを修正）

<br>

#### new画面+edit画面にてエラーを表示させる
---

既にコントローラにて、new及びeditアクションに飛ばすコードは書いた。  
続いて、new画面及びedit画面にてエラーを表示する。  

```rb
# new.html.slim
# 該当部分のみ

h1 タスクの新規登録

# 以前は render 'form'として、formに関する部分をパーシャル化した
# form部分のパーシャル化が難しそうだったので、新たにerror等に関する部分をパーシャル化した
= render partial: 'error', locals: { task: @task }
```

```rb
# edit.html.slim
# 該当部分のみ

h1 修正内容の確認

# 以前は render 'form'として、formに関する部分をパーシャル化した
# form部分のパーシャル化が難しそうだったので、新たにerror等に関する部分をパーシャル化した
= render partial: 'confirm', locals: { task: @task }
```

```rb
# _error_html.slim

.nav.justify-content-end
  = link_to '一覧', tasks_path, class: 'nav-link'

# errorがあれば表示する
- if task.errors.present?
  ul#error_explanation
    - task.errors.full_messages.each do |message|
      li = message
```

<br>

#### エラーがなければ、confirm_newからcreate、confirm_editからupdateへ飛ばす
---

エラーがない場合、createアクションもしくはupdateアクションに飛ばす必要がある。  
（オブジェクトが空である場合はcreateアクション、空でない場合はupdateアクション）  

既にコードを記載のとおり、form_withの部分が以下のとおりとなっており、@taskの値を持ってどちらの  
アクションに飛ばすか自動で振り分けてくれるので、コードの追記・修正は不要である。  

```rb
# _confirm_html.slim
# form_withに関する部分のみ記載

= form_with model: task, local: true do |f|
```

以下の記事を参考にした。  

[【Rails】form\_with/form\_forについて【入門】 \- Qiita](https://qiita.com/snskOgata/items/44d32a06045e6a52d11c#22-form_with-model-model)  

<br>

#### 登録アクションで戻るボタンからの遷移に対応
---

戻るボタンを押した際の実装も追加する。  

このままだと登録ボタンも戻るボタンも区別されていないので、戻るボタンを押しても登録されてしまう。  
そこで、戻るボタンを押した場合の処理もコントローラに記載する。  

戻るボタンを押した場合、@taskにパラメータを格納して、newアクションをrenderする。  

```rb
# tasks_controller.rb
# 該当部分のみ記載

class TasksController < ApplicationController
  before_action :set_task, only: [:show, :edit, :update, :destroy, :confirm_edit]

  〜 省略 〜

  def create
    @task = Task.new(task_params.merge(user_id: current_user.id))

    # 以下のif部分を追加
    if params[:back].present?
      render :new
      # saveされては困るのでreturnを記載
      return
    end

    if @task.save
      logger.debug "task: #{@task.attributes.inspect}"
      redirect_to @task, notice: "タスク「#{@task.name}」を登録しました。"
    else
      render :new
    end

  end

  def update
    # 以下のif部分を追加
    if params[:back].present?
      render :edit
      return
    end

    @task.update!(task_params)
    redirect_to tasks_url, notice: "タスク「#{@task.name}」を更新しました。"
  end
```
