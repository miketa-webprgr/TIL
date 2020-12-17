# 【第3回】 Railsガイドの「Action View フォームヘルパー」をひたすらまとめていく 【モデルオブジェクトを扱う】

## この連載の概要（繰り返し）

- フォーム部分の理解が不十分だと感じたので、Railsガイドで勉強していく
  - [Action View フォームヘルパー \- Railsガイド](https://railsguides.jp/form_helpers.html)
  - ざっくりと言うと、form_with関係とも言えますが、それ以外も対象となってます
- クリスマスまでを目処に、ちょこちょことまとめて記事をあげていく
  - ひとりアドベントカレンダーなる斬新なアイデアをかろりーなさんから頂きました笑
- Crieitにも近々掲載していきたい
- 小難しくしないのをコンセプトとしたいので、不正確な表現も増えそう
  - そもそも初学者なのでふつーに間違いもありそうですが、そこはご愛敬ということで

## 第3回の概要

- 「2 モデルオブジェクトを扱う」を取り上げる

## モデルオブジェクトを扱うフォームヘルパーとは

モデルオブジェクトを扱うフォームヘルパー・・・  
よく見るタイプのやつですが、まず雰囲気を掴む。  

```erb
<%= form_with @article, url: articles_path do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :body, size: "60x12" %>
  <%= f.submit "Create" %>
<% end %>
```

これまで取り上げたヘルパーは、検索など、モデルに紐づかないもの。  
今回は、モデル（≒テーブル）に紐づくヘルパー。  

例えば、検索フォームの場合、テーブルに保存しているレコードを取ってきて、  
そのレコードの情報をフォームに表示させる必要はない。  

ただ、記事を作成したり更新する場合、モデルと連携して、記事のタイトルを表示したり、  
保存したり、更新したり、削除したりする必要が出てくる。  

そこで、form_withやform_forを使わずにコードを書こうとすると、えらく大変なる。  
例えば、こんな感じで書いてみたとする。  

```erb
<%# 新規レコード保存する場合 %>
<% if @article.new_record? %>
  <%= form_tag articles_path do %>
    <%= text_field_tag :title %>
    <%= text_area_tag :body, size: "60x12" %>
    <%= submit_tag "Create" %>
  <% end %>
<%# 既存レコードを更新する場合 %>
<% else %>
  <%= form_tag articles_path, method: :patch do %>
    <%= text_field_tag :title, "#{@article.title}" %>
    <%= text_area_tag :body, "#{@article.body}", size: "60x12" %>
    <%= submit_tag "Create" %>
  <% end %>
<% end %>
```

本来は、`name="article[name]"`とし、`id="article_name"`としたいが、  
すぐに書き方が分からず断念した。  

以上のようなerbファイルだと特にnameが`article[〜]`でまとまらないので、  
controllerで`params.require(:article)`と書くことができず面倒なことになる。  

```html
<form action="/players/:id" accept-charset="UTF-8" method="post">
  <input type="hidden" name="_method" value="patch">
  <input type="hidden" name="authenticity_token" value="SltnhALTlrOwWz0KscbKrLiElefgRJ+6/l1JkyE5LkAx5QMAaI95SJluYw8f2EEKmYtm0cW43GpGtBi4lryUIw==">
    <input type="text" name="name" id="name" value="記事のタイトル">
    <textarea name="team" id="team" cols="60" rows="12">
      記事の中身
    </textarea>
    <input type="submit" name="commit" value="Create" data-disable-with="Create">
</form>
```

・・・とまあ、ごちゃごちゃと書いたが、

- モデルと紐づける上で`form_with model: @model`という書き方は便利
- nameやidを規則に沿って付与してくれる
- @modelがnew_recordであるか判断して、いい感じにリクエスト先やmethodも決めてくれる

以上を理解しておくとよい。  

## inputタグを生成するヘルパー

ちなみに、モデルに紐づかないフォームの場合、`_tag`となっていたが、  
モデルに紐づく場合、`_tag`はつかない。  

```erb
# モデルに紐づく
<%= text_field(:article, :title) %>
<%= text_area(:article, :body, size: "24x6" )>
```

なお、この場合、生成されるHTMLタグはこんな感じ。  

```html
<input type="text" value="記事のタイトル（DBから取得されたもの）" name="article[title]" id="article_title">
<input size="24x6" type="text" value="記事の本文（DBから取得されたもの）" name="article[title]" id="article_body">
```

第１引数にて、紐づくモデルを指定する。  
第２引数にてそのモデルのメソッド（多くの場合、そのmodelのattributes）を指定する。  

なお、`do |f|`と書くような場合、紐づくモデルを引数に指定しない。  

```erb
<%= form_with @article, url: articles_path do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :body, size: "60x12" %>
  <%= f.submit "Create" %>
<% end %>
```

ちなみに、fではフォームビルダーオブジェクトが作られているらしい。  
自分で書きながらまだよく分かってないが、今後解説する過程で理解していきたい。  

## fields_forについて

## resourcesを使うと自動識別してくれる

## 

Railsガイドでは、まず`form_tag`の説明から始まる。  
ここで、`form_tag`・`form_for`・`form_with`の概要を確認する。  

- `form_tag`はモデルに紐づけない場合、form_forはモデルに紐付ける場合に使う
- 昔（Rails5.1より前）は、`form_for`と`form_tag`があった
- `form_for`と`form_tag`の2つは、`form_with`に統合された
- この記事でも説明している
  - [【Rails 5】\(新\) form\_with と \(旧\) form\_tag, form\_for の違い \- Qiita](https://qiita.com/hmmrjn/items/24f3b8eade206ace17e2)

ということで、まずは`form_for`というモデルに紐づけないシンプルなケースを確認しよう。  
今では、`form_with`で書くことが一般的なので、その場合も確認する。  

```erb
# form_tag
<%= form_tag do %>
  Form contents
<% end %>

# form_with
<%= form_with do %>
  Form contents
<% end %>
```

```html
<!-- Railsガイドで紹介されていたHTML -->
<form accept-charset="UTF-8" action="/home/index" method="post">
  <div style="margin:0;padding:0">
    <input name="utf8" type="hidden" value="&#x2713;" />
    <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c432144748a6d4b7a7ddf2b71" />
  </div>
  Form contents
</form>

<!-- Rails 6系で試した場合、すこし違うHTMLが出力された -->
<form action="/" accept-charset="UTF-8" method="post">
  <input type="hidden" name="authenticity_token" value="E3gz0DuIaaH2ah/jibBQN5pOw5fRWu/0EKopuZ3uCkpoxldUUdSGWt9fQeYnrtuRu0EwofSmrCSoQ3iSKmuwKQ==">
  Form contents
</form>
```

前回の記事で触れたとおりだが、以下を確認しよう。  

- action: フォームで送信するURL
- method: HTTPのリクエストのメソッド（get, post, put, patch, deleteなど）
- authenticity_token: CSRF対策のためのトークン。

なお、前回記事では触れなかったが、下記のとおりである。  
そんなに重要ではないと思うけど一応確認する。  

- accept-charset: 文字コードを指定するオプション
- name="utf8"のinput: 文字コードを指定するオプション
    - IEはフォームのaccpet_charsetを無視して、ブラウザの文字コードと同じエンコードだと認識してたらしい
    - それを乗り越えるため、隠しパラメータで文字コードを指定する方法を採用していたらしい

デフォルトでは、**現在のURLに対してPOSTメソッドでHTTPリクエストが送られる。**  
なので、送信先のURLやメソッドに関する記述がなくとも、先ほどのようなHTMLタグが生成される。  

## 1.1 一般的な検索フォーム

モデルに紐づかないフォームとしてありがちなのが、検索フォームだ。  
この場合、actionやmethodがデフォルトのままだと困るので、設定する必要がある。  

また、検索フォームの場合、何のフォームであるか示すため、  
左横なんかに**ラベル**がついていることが一般的だと思う。  

そして、当然検索ワードを入力するtextなどの入力フォームや送信ボタンが必要となる。  

ということで、このような記述になる。  
せっかくなので、form_withバージョンで書いてみる。  

```erb
<%= form_with(url: "/search", method: "get") do %>
  <%= label_tag(:q, "Search for:") %>
  <%= text_field_tag(:q) %>
  <%= submit_tag("Search") %>
<% end %>
```

```html
<form accept-charset="UTF-8" action="/search" method="get">
  <div style="margin:0;padding:0;display:inline">
    <input name="utf8" type="hidden" value="&#x2713;" />
  </div>
  <label for="q">Search for:</label>
  <input id="q" name="q" type="text" />
  <input name="commit" type="submit" value="Search" />
</form>
```

GETメソッドの場合、parameterはURLにくっつけるような形で送信される。  
例えば、googleでmiketaと検索すると、以下のようなURLにリダイレクトされる。  

```text
https://www.google.com/search?q=miketa&rlz=1C5CHFA_enJP891JP891&oq=miketa&aqs=chrome..69i57j69i59l2j69i60l4j69i65.551j0j15&sourceid=chrome&ie=UTF-8
```

不勉強なので詳しいことは分からないが、以下のようなパラメータが送られている。  
本筋から外れるのでこれ以上は踏み入らないが、各メソッドの違いを意識するとよい。  

```text
q: miketa  
rlz: 1C5CHFA-en....  
aqs: chrome..69i...  
sourceid: chrome
ie: UTF-8
```

## 1.2 フォームヘルパーの呼び出しで複数のハッシュを使う

Railsガイドには、以下のように書いてある。  

> form_tagヘルパーは2つの引数を取ります。  
> 1つはアクションへのパスで、もう1つはオプションのハッシュです。  
>
> このハッシュには、フォーム送信のHTTPメソッドやHTMLオプション(フォーム要素のクラスなど)が含まれます。  

引数とは、メソッドのカッコ内のやつです。  

```rb
form_tag(引数はここだよ)
```

引数は、アクションへのパスとオプションのハッシュの２つなので、こうなる。  

```rb
form_tag(
  第１引数 - アクションへのパス（paramsの送り先のURL）, 
  第２引数 - オプションのハッシュ（GETなどのHTTPメソッドやhtmlタグ内のclassの指定など
)
```

第１引数についてはURLを直書きすることもできるが、`users_path`のように書くこともできるし、  
`controller: "people", action: "search"`のように書くこともできる。  

ただ、`controller: "people", action: "search"`を何気なく書くと、エラーになってしまう。  
例えば、以下のように書くとactionが第２引数と認識されるのでエラーになる。  

```rb
form_tag(controller: "people", action: "search", method: "get", class: "nifty_form")
```

よって、ハッシュを使うとよい。  

```rb
form_tag({controller: "people", action: "search"}, method: "get", class: "nifty_form")
```

## 余談：form_withの場合

ぶっちゃけあまりform_tagなんて使わないと思うので、  
form_withについても理解しておきたい。  

ということで、form_withメソッドについても調べてみた。  

```rb
form_with(model: nil, scope: nil, url: nil, format: nil, **options, &block)
```

ここでRuby自体の理解が大事になるのだが、こういった引数の書き方はキーワード引数という。  
キーワード引数を指定をしないとき、デフォルトの設定が採用される。  

例えば、root_pathへのフォームを作成する場合、このようなコードとなる。  

```erb
<%= form_with url: root_path do |form| %>
  フォーム内容
<% end %>
```

この場合、例えばmodelについての引数は指定されていない。  
その場合、デフォルトの`model:nil`が採用される。他のオプションについても、同様である。  

その他のオプション等が何を意味するかについては、長くなりそうなので、後ほどの機会で。  

## 1.3 フォーム要素生成に使うヘルパー

分かりやすいので、そのまま掲載する笑  

> Railsには、〜フォーム要素を生成するためのヘルパーが多数用意されています。  
> これらの基本的なヘルパーは名前が_tagで終わっており (text_field_tagやcheck_box_tagなど)  
> それぞれただ1つの`<input>`要素を生成します。  

ただ、どちらかというと、tagよりfieldの方が目にすることが多いかもしれない。  
fieldについては、後ほど触れる。  

続ける。  

> これらのヘルパーの1番目のパラメータは、inputの名前と決まっています。  
> ・・・たとえば、フォームに`<%= text_field_tag(:query) %>`というコードが含まれていたとすると、  
> コントローラで`params[:query]`と指定すればこのフィールドの値にアクセスできます。

少し具体的にみてみる。  

```erb
<%= form_with url:root_path do %>
  <%= check_box_tag(:pet_dog) %>
  <%= label_tag(:pet_dog, "I own a dog") %>
  <%= check_box_tag(:pet_cat) %>
  <%= label_tag(:pet_cat, "I own a cat") %>
  <%= submit_tag("Search") %>
<% end %>
```

これで、こんなHTMLタグが生成される。  

```html
<form action="/" accept-charset="UTF-8" method="post">
  <input type="hidden" name="authenticity_token" value="R0qYMo0//9dA1ECJ6G6muSXd9JXgss5z7DLPn6mwJL889Py252MQLGnhHoxGcC0fBNIHo8VOjaNU2560HjWe3A==">
  <input type="checkbox" name="pet_dog" id="pet_dog" value="1">
  <label for="pet_dog">I own a dog</label>
  <input type="checkbox" name="pet_cat" id="pet_cat" value="1">
  <label for="pet_cat">I own a cat</label>
  <input type="submit" name="commit" value="Search" data-disable-with="Search">
</form>
```

そして、このフォームから送信されたparamsを取得するには、  
〜tagの第１引数を使って、こんな感じで書いてあげれば大丈夫。  

```rb
# pet_dogにチェックを入れて、pet_catにチェックをしていない場合
params[:pet_dog] #=> 1という値が取得できる
params[:pet_cat] #=> nilになる（paramsが送られていないため）
```

## 1.4 その他のヘルパー

HTMLのinputタグは、山ほど種類がある。  
20種類以上ある。  

ということは、その数だけ、Railsのヘルパーもある。  
Railsガイドでは、これだけ紹介されているが、あくまで一例。  

```erb
<%= text_area_tag(:message, "Hi, nice site", size: "24x6") %>
<%= password_field_tag(:password) %>
<%= hidden_field_tag(:parent_id, "5") %>
<%= search_field(:user, :name) %>
<%= telephone_field(:user, :phone) %>
<%= date_field(:user, :born_on) %>
<%= datetime_local_field(:user, :graduation_day) %>
<%= month_field(:user, :birthday_month) %>
<%= week_field(:user, :birthday_week) %>
<%= url_field(:user, :homepage) %>
<%= email_field(:user, :address) %>
<%= color_field(:user, :favorite_color) %>
<%= time_field(:task, :started_at) %>
<%= number_field(:product, :price, in: 1.0..20.0, step: 0.5) %>
<%= range_field(:product, :discount, in: 1..100) %>
```

フォーム作成前に、どのようなinputタグがあるか、そしてそれに対応する  
railsのヘルパーはどのようなものになるか、確認してみるといいかも。  

## 最後にテスト

- フォームヘルパーは、HTMLの何を簡単に作るものでしょう？  
- getメソッドの場合、パラメータはどこに付与されるでしょう？  
  - ここではざっくりとしか説明してないですが、postメソッドの場合などについても調べてみるとよい
- 以下の場合、どうすればいいでしょう？
  - emailフィールドのフォームを設ける
  - コントローラでparams[:person]として値を取得したい
  - モデルに紐づけず、検索フォームとしたい
  - 検索フォームは、`emails_controller.rb`のsearchアクションで処理する
  - 適当な設定なので、おかしいところがあるかもしれないが、ご容赦ください
  - その場合、ツッコミどころを考えるのも勉強になります笑
- fieldの数を10個ぐらいあげてみよう！

## 補足

やばい、この調子だと日がくれる...  
