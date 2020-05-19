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

では、confirmアクションで何を行うのか整理する。  
1. 編集内容を反映したパラメータを受け取る。（パラメータを使ってビューで反映）
2. validメソッドにて内容を検証し、エラーがあれば元の画面にエラーを表示する

<br>

#### 確認画面を表示するアクションを追加する
---

tasks_controller.rbに以下のとおり記載。

```rb
# tasks_controller.rbにconfirm?newとconfirm_editアクションを追加する

def confirm_new
  # current_user.tasks部分はassociation
  @task = current_user.tasks.new(task_params)
  # validメソッドにてエラーがないか検証
  render :new unless @task.valid?
end

def confirm_edit
  # current_user.tasks部分はassociation
  # なお、set_taskメソッドと同じ内容のため、before_actionでまとめて記載
  @task = current_user.tasks.find(params[:id])
  # validメソッドにてエラーがないか検証
  render :new unless @task.valid?
end

```

validメソッドについては以下で確認。  
[Railsガイド 1\.4 valid?とinvalid?](https://railsguides.jp/active_record_validations.html#valid-questionmark%E3%81%A8invalid-questionmark)  

なお、routes.rbには以下のとおり記載。  

```rb
# routes.rb
# 該当部分のみ記載

  # resources :tasks を修正し、その下部ディエレクトリにconfirm_newとconfirm_editを追加
  resources :tasks do
    # 現場railsでは、confirm_newアクションしかなかったため、member do がなく、
    # on: :new を足すような形で省略されていた
    member do
      post :confirm, action: :confirm_new
      patch :confirm, action: :confirm_edit
    end
  end
```

なお、コードの記載にあたっては、以下を参照した。  

[Railsガイド 2\.10\.1 メンバールーティングを追加する](https://railsguides.jp/routing.html#%E3%83%A1%E3%83%B3%E3%83%90%E3%83%BC%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B) 

[railsのroutes\.rbのmemberとcollectionの違いをわかりやすく解説してみた。〜rails初心者から中級者へ〜 \- Qiita](https://qiita.com/hirokihello/items/fa82863ab10a3052d2ff#comments)  

以下のとおり、ルーティング設定が完了した。  

```
confirm_task    POST   /tasks/:id/confirm(.:format)    tasks#confirm_new
                PATCH  /tasks/:id/confirm(:format)     tasks#confirm_edit
```

続いて、confirm_taskのビューを作成する。  

```rb
### 作業中 ###

```

また、新規登録画面からの遷移先を変更する。  
createアクションではなく、confirm_newアクションに移動するよう設定変更する。


