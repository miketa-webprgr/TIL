# 【第2回】 Railsガイドの「Action View フォームヘルパー」をひたすらまとめていく 【基本的なフォームを作成する】

## この連載の概要（繰り返し）

- フォーム部分の理解が不十分だと感じたので、Railsガイドで勉強していく
  - [Action View フォームヘルパー \- Railsガイド](https://railsguides.jp/form_helpers.html)
  - ざっくりと言うと、form_with関係とも言えますが、それ以外も対象となってます
- クリスマスまでを目処に、ちょこちょことまとめて記事をあげていく
  - ひとりアドベントカレンダーなる斬新なアイデアをかろりーなさんから頂きました笑
- Crieitにも近々掲載していきたい
- 小難しくしないのをコンセプトとしたいので、不正確な表現も増えそう
  - そもそも初学者なのでふつーに間違いもありそうですが、そこはご愛敬ということで

## 第2回の概要

- 「１ 基本的なフォームを作成する」を取り上げる
- ちなみに、9章まである。12回ぐらいの連載で終わるかな

## フォームヘルパーのヘルパーって？

- この勉強をしたいと思ったのは、フォームについて勉強したいと思ったから
- とはいえ、Railsガイドのタイトルは「Action View フォームヘルパー」
  - ということで、まずヘルパーについて簡単に触れる

Viewフォームヘルパーとは、ControllerやModelではなく、Viewで使うことを想定して、  
フォームを簡単かつ便利に作るためのヘルパー（道具）のこと。  

例えば、content_tagヘルパーというものがあるのだけれど、これはまさにその典型例。  

```rb
content_tag(:div, "header", class: ["one", "two"])
#=> "< div class="one two" >header< /div >"
```

- [Rails tips: ビューの\`content\_tag\`のあまり知られていないオプション（翻訳）｜TechRacho（テックラッチョ）〜エンジニアの「？」を「！」に〜｜BPS株式会社](https://techracho.bpsinc.jp/hachi8833/2018_04_10/54701)

という観点からすれば、フォームヘルパーとは、

- **フォームを簡単かつ便利に作るためのヘルパー（道具）**のこと。  

ということは、ヘルパー全般に言えるけれども、簡単かつ便利になる前のHTMLタグたちを理解  
しなければ、そもそもの意味は理解できないとうこと。  

そして、原理や理屈を踏まえた上で、ヘルパーをどううまく使えるか考えていくのが大事。  

ゴリゴリ`#{}`を使って、その中にrubyを展開していってもいいけど、ヘルパーを使うと  
綺麗に表現することができるようになる。はず。。。  

## 基本的なフォームを作成する

Railsガイドでは、まずform_tagの説明から始まる。  
ここで、form_tag・form_for・form_withの概要を確認する。  

- form_tagはモデルに紐づけない場合、form_forはモデルに紐付ける場合に使う
- 昔（Rails5.1より前）は、form_forとform_tagがあった
- form_forとform_tagの2つは、form_withに統合された
- この記事でも説明している
  - [【Rails 5】\(新\) form\_with と \(旧\) form\_tag, form\_for の違い \- Qiita](https://qiita.com/hmmrjn/items/24f3b8eade206ace17e2)

ということで、まずはform_forというモデルに紐づけないシンプルなケースを確認しよう。  
今では、form_withで書くことが一般的なので、その場合も確認する。  

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

- accept-charset:
- name="utf8"のinput:

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

ただ``controller: "people", action: "search"`を何気なく書くと、エラーになってしまう。  
例えば、以下のように書くとactionが第２引数と認識されるのでエラーになる。  

```rb
form_tag(controller: "people", action: "search", method: "get", class: "nifty_form")
```

よって、ハッシュを使うとよい。  

```rb
form_tag({controller: "people", action: "search"}, method: "get", class: "nifty_form")
```

## 余談：form_withの場合

とは言われても、ぶっちゃけあまりform_tagなんて使わないと思うので、  
form_withについても理解しておきたい。  

ということで、form_withメソッドについても調べてみた。  

```rb
form_with(model: nil, scope: nil, url: nil, format: nil, **options, &block)
```

ここでRuby自体の理解が大事になるのだが、こういった引数の書き方はキーワード引数という。  
キーワード引数についは特に指定をしないとき、デフォルトの設定が採用される。  

例えば、root_pathへのフォームを作成する場合、このようなコードとなる。  

```erb
<%= form_with url: root_path do |form| %>
  フォーム内容
<% end %>
```

この場合、modelは指定されていないが、その場合はデフォルトの`model:nil`が採用されている。  
他のオプションについても、同様である。  

その他のオプション等が何を意味するかについては、後ほどの機会で。  

## 1.3 フォーム要素生成に使うヘルパー

分かりやすいので、そのまま掲載します笑  

> Railsには、~フォーム要素を生成するためのヘルパーが多数用意されています。  
> これらの基本的なヘルパーは名前が_tagで終わっており (text_field_tagやcheck_box_tagなど)  
> それぞれただ1つの`<input>`要素を生成します。  

続けます。  

> これらのヘルパーの1番目のパラメータは、inputの名前と決まっています。  
> ・・・たとえば、フォームに`<%= text_field_tag(:query) %>`というコードが含まれていたとすると、  
> コントローラで`params[:query]`と指定すればこのフィールドの値にアクセスできます。

少し具体的にみてみましょう。  

```erb
<%= form_with url:root_path do %>
  <%= check_box_tag(:pet_dog) %>
  <%= label_tag(:pet_dog, "I own a dog") %>
  <%= check_box_tag(:pet_cat) %>
  <%= label_tag(:pet_cat, "I own a cat") %>
  <%= submit_tag("Search") %>
<% end %>
```

これで、こんなHTMLタグが生成されます。  

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
〜tagの第１引数を使って、こんな感じで書いてあげれば大丈夫です。  

```rb
# pet_dogにチェックを入れて、pet_catにチェックをしていない場合
params[:pet_dog] #=> 1という値が取得できる
params[:pet_cat] #=> nilになる（paramsが送られていないため）
```

## 1.4 その他のヘルパー

HTMLのinputタグは、山ほど種類があります。  
20種類以上あります。  

ということは、その数だけ、Railsのヘルパーもあります。  
Railsガイドでは、これだけ紹介されていますが、あくまで一例です。  

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