## 現場Rails Chapter08 「JavaScriptでページに変化をつける」

---

<BR><BR>

### Chapter08-1 「JavaScriptでページに変化をつける」  
---

index.html.slim の td 要素の部分にカーソルを合わせると、  
その背景色を変えるような処理をJSで行っていく。  

JSについては今後勉強をしていくため、コードは可能な限りコピペで済ます。  

現場railsのとおりjsファイルを作成すると、以下のとおり、  
マウスオーバーした背景色が変更するようになった。  

<a href="https://gyazo.com/fd6a8ebec0888d76bdb2f7ecf69eb15d"><img src="https://i.gyazo.com/fd6a8ebec0888d76bdb2f7ecf69eb15d.gif" alt="Image from Gyazo" width="600" border=1/></a>  

JSは、フロントエンド（ブラウザの方で処理される言語）なので、  
ページの遷移なしに画面を動的に変更することができる。  

なお、開発環境では、Sprocketsがアセットパイプラインがあたかも起動しているように、  
動作してくれるため、HTMLで読み込むような指示がなくとも、JSが自動的に反映される。  

<BR><BR>

### Chapter08-2 「AjaxでRailsサーバと通信する」  
---

JSは、基本的にはフロントエンドの言語ではあるが、場合によっては、  
サーバー側からデータを新たに取得したり、データを裏で更新したい場合がある。  

このようなことを実現させる仕組みとして、Ajaxがある。  
非同期通信をサーバーと行い、ページの再読み込みなしにページを更新するためのプログラミングだ。  

Ajaxは、以下のメリットがある。  
- データの一部更新だけで済む
- 非同期であるため、処理待ちのストレスをページ遷移に比べて感じづらい


<br>

#### Ajaxでタスクを削除する」  
---

タスク削除は、これまでタスクのデータを削除するという処理だけでなく、  
同じ画面をリダイレクトするという処理を行ってきた。  

これをajaxで置き換えると、リダイレクト処理が省けるため、処理の効率化が図れる。  


<br>

#### Ajaxでタスク削除の処理をどのように置き換えるか
---

【Before】  
1. HTTPプロトコルにてDELETEメソッドを呼び出し、DB上から削除
2. 処理が反映された画面を表示するため、リダイレクト

【After】  
1. Ajaxでサーバーにリクエストを飛ばす
2. JSであるため、リダイレクト不要。タスクが非表示になるよう設定する

<br>

#### Ajaxでサーバーにリクエストを飛ばす
---

index.html.slim に、「remote: true」を足す。  
すると、rails がいい感じに ajax でサーバーリクエストを飛ばしてくれる。  

```
# index.html.slim
# remote: true を該当箇所に加えるだけ

= link_to '削除', task_path(t), method: :delete, remote: true, data: { confirm: "タスク「#{t.name}」を削除します。よろしいですか？"}, class: 'btn btn-danger'
```

<br>

#### タスクが非表示になるよう設定する
---

まず、tasks_controller.rbからdestroyメソッドを変更する。  
リダイレクトのコードを削除し、HTTPステータス204が返るようにする。  

```rb
# tasks_controller.rb
# destroyアクションのみ

def destroy
  @task.destroy
  # HTTPリクエストのヘッダーだけを返す。ボディは空。
  head :no_content
end
```


次に、削除したタスクを非表示にするため、JSに変更を加える。  
Ajax通信が成功したか確認し、成功した場合に非表示とするような設定にする。  

また、削除の通信を見分けるため、CSS上の目印も付与する。  

```js
// tasks.js

document.addEventListener('turbolinks:load', function() {
    document.querySelectorAll('.delete').forEach(function(a) {
        a.addEventListener('ajax:success', function() {
            var td = a.parentNode;
            var tr = td.parentNode;
            tr.style.display = 'none';
        });
    });
});
```


```slim
# index.html.slim
# class に delete という目印を加えただけ

= link_to '削除', task_path(t), method: :delete, remote: true, data: { confirm: "タスク「#{t.name}」を削除します。よろしいですか？"}, class: 'btn btn-danger delete'
```
<br>

#### 実装後
---

実際には、ポップアップの削除確認ウインドウが表示されているが、  
リダイレクトしないため、以下のとおり動作が高速になった。  

<a href="https://gyazo.com/89b42bb700076d4aa17d30d2e8e68ae8"><img src="https://i.gyazo.com/89b42bb700076d4aa17d30d2e8e68ae8.gif" alt="Image from Gyazo" width="600" border=1/></a>

なお、この機能は、rails-ujsによって成立しているとのこと。  

<br>

#### コントローラからJSを返して実行する(SJR)
---

先ほどの実装は、アセットパイプラインにてHTML＋CSS＋JSなどを合体させて、  
まずクライアントサイドに送っておき、クライアントサイドの手元にあるJSを利用して、  
タスク削除の処理を行ってきた。これとは違う、SJRという実装方法がある。  

SJRとは、Server-generated Javascript Responsesの略であり、  
ajax通信の度に該当のJSを送ってあげる方式である。  

まず、DOM要素を指定できるように、HTML上のidで識別できるような形で  
index.html.slimのコードを書き直す。  

```
# index.html.slim
# tr の後に「id="t-#{t.id}"」を追加する
# 現場railsに背いて、あえて|t|としたので色々とバグが起きるが、無事解明できた

〜　省略　〜

tbody
  - @tasks.each do |t|
    tr id="t-#{t.id}"

〜　省略　〜

```

続いて、「remote: true」にてajax通信がサーバーに飛んだ後に返すJSのファイルを作成する。  

```js
# views/tasks/destroy.js.erb
# なお、SJRであるメリットを生かして、適切なメッセージを表示させる機能を追加した

document.querySelector("#t-<%= @task.id %>").style.display = 'none';

var message = document.createElement('p');
message.innerText = 'タスク「<%= @task.name %>」を削除しました。残りのタスクは<%= current_user.tasks.count %>件です。';
document.querySelector('table').insertAdjacentElement('beforebegin', message);
```

なお、assets/javascripts/tasks.js は不要なので、コードをコメントアウトする。  
また、tasks_controller.rbのdestroyアクションに書いてある、head: no_content は削除する。  

以下のとおり、実装できた。  
JSのエラーはスルーされてしまうので、上手く動作しないと原因解明が大変だった。  

Railsの開発環境だと、どこでエラーが起きたのが示してくれるので、  
あんなにも憎かったエラー画面のありがたさがやっと分かった。親みたいな存在だ。  

<a href="https://gyazo.com/920d5afa87128bfd51069a511fdbb754"><img src="https://i.gyazo.com/920d5afa87128bfd51069a511fdbb754.gif" alt="Image from Gyazo" width="600" border=1/></a>  

<BR><BR>

### Chapter08-3 「Turbolinks」  
---

Railsが提供するJS機能として、ページ遷移を高速化する Turbolinks がある。  

遷移先のページを Ajax で取得し、取得したページが要求するJSやCSSが現在のものと同一であれば、  
titleやbody要素のみを置き換える。リクエストごとにJSやCSSをブラウザが評価しないため、  
パフォーマンスが向上させることができる、とのこと。  

また、ブラウザの戻るボタンなどの履歴操作や、その際のキャッシュ復元にも対応している。  

なお、Turbolinks は、Rails new した時点でオンになっており、導入作業は不要。  

<br>

#### Turbolinksの注意点
---

Turbolinksは、ページ遷移や戻る、進むと言ったブラウザの動作をフックとして動作する。  
そこで、Tubolinksは、自身の処理状態に応じてイベントを発行する機能を備えている。  

また、Turbolinksを活用することで、本来はページの遷移が想定されているところで、  
Turbolinksの機能により上手くイベントが発火しないことがあるので、注意すること。  

その他、`<script>`のコードはhead内に書かないと、高速化の恩恵が得づらいことに留意すること。  

Turbolinksにはデメリットがあるので、必要に応じて無効化すること。  
- rails new の際に -- skip-turbolinks オプションをつけることで解除できる
- rails new 後に無効化する場合、gem を削除し、JSのマニフェストファイルから該当箇所を削除する。  



