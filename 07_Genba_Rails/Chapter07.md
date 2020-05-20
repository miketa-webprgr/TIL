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

チェリー本の著者である伊藤さんのQiita記事を発見したが、  
今後エラーに遭遇して、解決の見込みが立たなければ、即この記事を眺めること。  

[プログラミング初心者歓迎！「エラーが出ました。どうすればいいですか？」から卒業するための基本と極意（解説動画付き） \- Qiita](https://qiita.com/jnchito/items/056325421b7e36f02335)  

<br>


#### ルーティングを設定する
---

まず、目標を確認。  

URL        tasks/new/confirm  
アクション   tasks#confirm_new  

URL        tasks/:id/confirm  
アクション   tasks#confirm_edit  

なお、routes.rbには以下のとおり記載。  

```rb
# routes.rb
# 該当部分のみ記載

  resources :tasks do
    post :confirm, action: :confirm_new, on: :new
    member do
      patch :confirm, action: :confirm_edit
    end
  end
```

なお、コードの記載にあたっては、以下を参照した。  

[Railsガイド 2\.10\.1 メンバールーティングを追加する](https://railsguides.jp/routing.html#%E3%83%A1%E3%83%B3%E3%83%90%E3%83%BC%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B) 

[railsのroutes\.rbのmemberとcollectionの違いをわかりやすく解説してみた。〜rails初心者から中級者へ〜 \- Qiita](https://qiita.com/hirokihello/items/fa82863ab10a3052d2ff#comments)  

以下のとおり、ルーティング設定が完了した。  
confirm_editアクションのパス名がconfirm_taskで気持ち悪い感じがするが、今回はスルーすることとする。  

```
confirm_new_task   POST   /tasks/new/confirm(.:format)    tasks#confirm_new
    confirm_task   PATCH  /tasks/:id/confirm(.:format)    tasks#confirm_edit
```

<br>

#### 作業１・２・３
---

先ほど書いた作業リストの内、１〜３を行う。

confirm_new  
1. newアクションにて新規画面を表示
2. createアクションではなく、confirm_newアクションに飛ばすように変更（new画面のリンクを修正）
3. confirm_newの画面を表示（confirm_newの画面を作成、パラメータ格納についてコントローラに記入）

confirm_edit  
1. editアクションにて編集画面を表示
2. updateアクションではなく、confirm_editアクションに飛ばすように変更（edit画面のリンクを修正）
3. confirm_editの画面を表示（confirm_editの画面を作成、パラメータ格納についてコントローラに記入）

【続きは明日（なお、コード自体はそれなりに書けている）】  