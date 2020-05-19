すみません。
現場Railsで困っています。

フォームタグを作ったのですが、上手くcreateアクション（具体的には'/tasks'）に  
飛ばすことができず、なぜか'tasks/new'に飛んでしまい、途方にくれています。

質問が長くなって申し訳ないですが、確認してもらえないでしょうか？

---

新規登録画面

<img src="https://i.gyazo.com/a7ffff0e1e1ada68d232de7f923fcd21.png" height=300px border=1>  

登録ボタンを押した後のエラー

<img src="https://i.gyazo.com/7cfd864c827313ca083e8f6c9284415d.png" height=450px border=1>  

<br>

フォームですが、下記のとおりコードを書いてみました。
現場railsに沿った形で書いています。

``` 
# application.new.html.slim 
# model: @taskのところに注目してください
= form_with model: @task, local: true do |f|
  .form-group
    = f.label :name
    = f.text_field :name, class: 'form-control', id: 'task_name'
  .form-group
    = f.label :description
    = f.text_area :description, rows: 5, class: 'form-control', id: 'task_description'
  =f.submit nil, class: 'btn btn-primary'
```

以下を確認したのですが、このように書いてありました。  
[Pikawaka: 【Rails】form_withの使い方を徹底解説！](https://pikawaka.com/rails/form_with#form_withの引数)  

> データーベースに保存する場合、form_withの引数にはモデルクラスのインスタンスを指定します。  
> モデルクラスのインスタンスとは保存したいテーブルのクラスのインスタンスのことです。  
> 今回はusersテーブルに新たにレコードを作成したいので、コントローラー側で下記のように記述します。  
>
>```
> def new
>  @user = User.new
> end
>```
>
> この@userをform_withの引数に指定するわけです。

> コントローラーで作成したインスタンスがnewメソッドで新たに作成されて何も情報を持っていなければ  
> <b>自動的にcreateアクションへ</b>、findメソッドなどで作成され、すでに情報を持っている場合は  
> updateアクションへ自動的に振り分けてくれます。

この記事が正しいとすれば、現在のコントローラのコードは以下のようになっているので、  
自動的にcreateアクションに飛ばしてくれてもよいかと思います。  

```
# tasks_controller.rb
class TasksController < ApplicationController
  def index
    
  end

  def show
  end

  def new
    ＠task = Task.new
  end

  def edit
  end

  def create
    task = Task.new(task_params)
    task.save!
    redirect_to tasks_url, notice: "タスク「#{task.name」}を登録しました。"
  end

  private

  def task_params
    params.require(:task).permit(:name, :description)
  end

end
```

もしcreateアクションに飛んでいるのであれば、'/tasks'に飛ぶはずです。  
この記事を参考にして、そう考えました。  

[Rails resourcesメソッドとresourceメソッド](https://qiita.com/Tamitchao/items/6f45aa6daf1412b78d10)  

> また、resourcesメソッドは名前付きルートも自動で生成します。
> 
> | 名前付きパス(_path)   | 名前付きパス(_path) | 対応するパス      |
> |:-------------------|:------------------|:----------------|
> | users_path         | users_url         | /users          |
> | user_path(id)      | user_url(id)      | /users/:id      |
> | new_user_path      | new_user_url      | /users/new      |
> | edit_user_path(id) | edit_user_url(id) | /users/:id/edit 

ちなみに、routes.rbファイルのコードは以下のとおりです。  

```
# routes.rb
Rails.application.routes.draw do
  root to: 'tasks#index'
  resources :tasks
end
```

考え方に間違えはないような気がするのですが。。。  
すみません、どこが間違っているか教えてください！！！  

<br>

## 解決方法

---

自己解決した。  
原因は不明。  

```
# new.html.slim 
= form_with model: @task, local: true do |f|

# 以下省略
```

@taskというのは、要はTask.newを持ってきているので、  
何気なく以下のとおり書き換えてみた。   

```
# new.html.slim 
= form_with model: Task.new, local: true do |f|

# 以下省略
```

すると、これまで'tasks/new'にPOSTで飛ぶようになっていたものが、突如として  
'task'にPOSTで飛ぶようになって、一気に問題が解決した。。。  

何だったんだろうと思い、また以下のように戻してみた。  
すると、これが不思議で、前まで動かなかったものが動くようになる。。。

```
# new.html.slim 
= form_with model: @task, local: true do |f|

# 以下省略
```

何とも言えないが、ここに拘り過ぎても仕方がない。  
次に進むことにする。  