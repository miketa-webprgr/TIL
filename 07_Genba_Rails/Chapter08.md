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

<BR><BR>

### Chapter08-4 「モダンなJavaScript管理を行う」  
---

JSのコード量が増えるにつれて、ファイル間の依存関係が煩雑になり、あるコードの変更が他のコードに影響を与える  
などの問題が生じてしまう。また、ES2015以降の仕様でJSを書いた時に、実行できるブラウザに制限が生じてしまう。  

これらの問題を解決するために、RailsではYarnというパッケージマネージャーとWebpackerというライブラリを提供している。  

・・・なるほど？

<br>

#### Yarn
---

Yarn  
- Facebookが開発
- JSのパッケージマネージャー
  - パッケージとは、ライブラリの公開単位であり、コードなどのファイルをひとまとめにしたもの
  - 要は、jQueryやReactなどのバージョン管理等を行うbundler的なものがJSにもある、ということらしい

以下の記事を読んで、学習。  
[JavaScriptのパッケージマネージャーnpmとYarnについて解説します！ \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/96866)  

RailsでYarnを使えるようにするには、Yarnのサイトの案内に従ってインストールする必要がある。  
・・・それ以上の説明は現場railsにないので、調べてみる。  

公式サイトをみると、homebrew がインストールされている場合、以下をコマンド入力するよう指示があった。  
環境による部分もあるのだろうが、サクッと終わった。  

```
brew install yarn
```

Yarnでパッケージを追加する場合、以下を入力する。  

```
$ bin/yarn add Reactなどのパッケージ
```

以上のコマンドを入力することで、指定されたパッケージをインストールできるだけでなく、  
指定されたパッケージをpackage.jsonに追加することができる。  

これは、勘違いしていなければ、RailsでいうところのGemfileのような機能を果たす。  
package.jsonを開発者の間で共有することで、簡単に同じJSライブラリ環境を作ることができるとのこと。  

例えば、開発に新しく参加する場合、package.jsonファイルを共有してもらい、  
以下のコマンドを入力することで、簡単に同じパッケージを導入することができる。  

```
$ bin/yarn add パッケージ名
```

基本的な使い方については、こちらのブログで説明がされている。  
[Yarn の使い方。インストールの方法から使い方まで解説します。 \| オリジナルゲーム\.com](https://original-game.com/yarn-how-to-use/#m_heading-0)  

あと、あまり今回は調べなかったが、そもそもNode.jsについてまだピンときていないところがあるので、
JSの勉強をする際に意識していきたい。（サーバーサイドでJSを使うということは分かるけど、具体的なイメージが沸かない）  

<br>

#### Webpacker
---


Webpackerは、JSのビルドツール「Webpack」のラッパーである。  
・・・日本語でお願いします。  

困り果ててGoogleで検索していたら、以下の記事を発見。  
[webpack学習の基本のき \|](https://www.fundely.co.jp/blog/tech/2020/01/22/180037/)  

- WebpackerとWebpackは、別物である。
- Webpackとは、JavaScriptモジュールバンドラー
  - モジュール =  コンパイルした（いい感じに読み込みやすくする）JS・CSS・画像ファイルなどのアセットファイル
  - バンドラー =  直訳すると束。アセットファイルを束にする機能。
  - 既に取り扱ったSprocketsと同じようなアセットパイプラインのツールらしい

なお、Webpackの導入により、以下のような問題が解決できるらしい。  
・特定のブラウザでES6構文が使えない
・ファイルの取得時間が増える
・規模が大きくなると管理が辛くなる

また、Webpackerとは、Railsのgemであり、Webpackを簡単に扱うためのツールである。  

Rails 6系では、SprocketsからWebpackerに完全移行しているようであり、重要度は極めて高そう。  
[Rails 6の変更点と新機能 \| RE:ENGINES](https://re-engines.com/2019/08/26/rails-6%E3%81%AE%E5%A4%89%E6%9B%B4%E7%82%B9/)  

WebpackerはGemfileであるため、おなじみのGemfileに記載 → bundle install にて導入できる。  
なお、新規アプリの場合、rails new アプリ名 --webpack にて対応できるとのこと。  

インストール後、以下のコマンドにより、Webpackerで必要な設定ファイル・ディレクトリの生成、  
必要なJSパッケージインストールが行われる。  

```
$ bin/rails webpacker:install
```

なお、Sprocketsにおけるマニフェストファイルは、エントリポイントと呼ばれる。  
エントリポイント（applicaion.js)は、app/javascripts/packs内に保存される。  

このpacks内に保存したファイルが、コンパイルされるらしい。  

<br>

#### Webpackerでのコンパイル
---

開発環境においては、自動でコンパイルが行われる。  
本番環境のためのコンパイルは、webpacker::compileというRakeタスクで行う。  

Sprocketsとの共存も出来るもよう。  
[Ruby on Rails で sprockets から Webpacker へ移行し、移行できないものは共存させる方法 \- Qiita](https://qiita.com/tatsurou313/items/645cbf0a3af4c673b5df)  

<BR><BR>

### Chapter08-5 「taskleafにReactを導入してみる」  
---

ここで、Facebook社が開発しているReactというJSのUIライブラリを導入してみる。  
ReactではReactDOMというものも必要になるので、Reactと併せてインストールする。  

Yarnを使ってインストールする。  

```
$ bin/yarn add react react-dom
```

続いて、 app/javascript/taskleaf ディレクトリに hello.js を作成する。  
コードは下記のとおり。  

```js
// app/javascript/taskleaf/hello.js

import React from 'react';
import ReactDOM from 'react-dom';

document.addEventListener('DOMContentLoaded', () => {
  ReactDOM.render(
    React.createElement('div', null, 'Hello World!'),
    document.body.appendChild(document.createElement('div')),
  );
});
```

続いて、エントリポイントのapplicaition.jsに以下を記載する。

```js
// app/javascript/packs/application.js
// hello.jsをコンパイルするための設定だと思われる

import 'taskleaf/hello';
```

そして、application.html.slimにて、javacript_include_tagを書き換える。

```slim
# head内の = javacript_include_tagの部分を以下のコードで上書きする

= javascript_pack_tag 'application'
```

下記のとおり、Reactにて Hello World! と表示できた。  
びっくりするぐらい地味で、実装できたことに気付くの時間がかかった。  

<a href="https://gyazo.com/ab083132189374993b16931e3956bed3"><img src="https://i.gyazo.com/ab083132189374993b16931e3956bed3.png" alt="Image from Gyazo" width="452" border=1/></a>  