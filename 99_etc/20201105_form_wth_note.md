# Form関係のノート

Railsガイドを読んで、ひたすらまとめていくノート。  
使えなくちゃ意味がないので、できれば練習アプリも作成したい。  

- [Action View フォームヘルパー \- Railsガイド](https://railsguides.jp/form_helpers.html)

なお、Railsガイドに入る前に、フォームで使われるHTMLタグについて学習する。  
HTMLタグの意味が理解できていないと、フォームヘルパーが作ってくれるものもきちんと理解できない。  

## 基本的なこと

本当に怪しい人は、ここから勉強すること。  

- [技術基礎研修「クックパッドを支える仕組み」 / Introduction to the Internet \- Speaker Deck](https://speakerdeck.com/osa/introduction-to-the-internet?slide=208)

クライアントから送るものは何か、まず意識するのが大事。  
Chromeなどのウェブブラウザから、遠くどこかのWebサーバーに対してリクエストを送る。  

なので、以下を意識することが大事。  
What, Where, How,（ + Type of input)と覚えるとよいかも。（自己流です）  
2W1HT・・・なんかいい語呂でもないかな。。。  

- どこのURLに
- どのようなメソッドで
- どのようなパラメータを送るか
  - keyとvalueは何か
- フォームの種類は何か
  - HTMLのinputで指定する
  - text_fieldなどがある
  - 実は20種類以上もある。。。

## formタグの概要について

まず、フォームの概要について押さえておく。  
ここはRailsガイドに掲載されていないが、把握すべき基礎的な内容だ。  

```html
<form accept-charset="UTF-8" action="/users" method="post">
  <input type="email" name="user[email]" value="abc@example.com" />
  <input type="text" name="user[name]" value="alice" />
  <input type="text" name="user[age]" value="20" />
</form>
```

action: アクセス先のURL
method: HTTPメソッド
type: フォームの入力値の種類（text, email, radio, checkboxなど）
name: フォームの名前を示している
value: 値

アクションは、あくまでアクセス先のURLである。  
間違いやすいので注意すること。  

paramsハッシュの中身は、以下のとおりとなる。  

`{'user' => {'email' => 'abc@example.com'}, {'name' => 'alice'}, {'age' => 20} }`

このような形で書くことで、params[:user]と書くことで、Userモデルに紐づく全てのハッシュを取得できる。  
また、以下のように取得するハッシュを制限することができる。  

```rb
params.require(:user).permit(:email)
```

### 参考

- [基本的なフォーム \-\- ごく簡単なHTMLの説明](https://www.kanzaki.com/docs/html/htminfo31.html)
- [7\.3 フォームヘルパーを使う](https://railsguides.jp/form_helpers.html#%E3%83%95%E3%82%A9%E3%83%BC%E3%83%A0%E3%83%98%E3%83%AB%E3%83%91%E3%83%BC%E3%82%92%E4%BD%BF%E3%81%86)
- [RUNTEQの講師をやってみてわかった初学者にありがちなパターン20選（後編） \- Qiita](https://qiita.com/DaichiSaito/items/cd66115569b0a75f1bfa)

## Labelタグについて

フォームでは、よくラベルタグを使う。  
フォームを使うメリットは、２つある。  

- ラベルをクリックすると、そのチェックボックスをクリックしたことになる。  
- 音声読み上げソフトが、いい感じに対応してくれる。  

ラベルタグは、インプットタグと紐付ける。  
紐付けにあたっては、labelのforとinputのidを同一にする。  

以下の事例においては、cheeseで紐付けされている。  
繰り返しになるが、nameはパラメータのkeyなので、勘違いしないように。  
（keyというのは用語が適切でないかも。Hashのkey-valueのkey）  

```html
<div class="preference">
    <label for="cheese">Do you like cheese?</label>
    <input type="checkbox" name="cheese" id="cheese">
</div>
```

Form_withでは使われないが、labelタグにinputタグをネストすることでも紐付け可能。  

```html
<label>Do you like peas?
  <input type="checkbox" name="peas">
</label>
```

- [<label> \- HTML: HyperText Markup Language \| MDN](https://developer.mozilla.org/ja/docs/Web/HTML/Element/label)

## Formの隠し要素

Railsでフォームを作ると、文字エンコードとauthenticity_tokenが自動で含まれる。  
authenticity_tokenはXSS対策のため。GETメソッドの場合、このトークンは作成されない。  
Railsサーバーに対して、このトークン抜きでリクエストを送っても拒否される。要注意。  

```erb
<%= form_tag do %>
  Form contents
<% end %>
```

```html
<form accept-charset="UTF-8" action="/home/index" method="post">
  <div style="margin:0;padding:0">
    <input name="utf8" type="hidden" value="&#x2713;" />
    <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c432144748a6d4b7a7ddf2b71" />
  </div>
  Form contents
</form>
```

## フォームは検索でよく使われる

フォームは、DBを保存する際に使うイメージが強いかもしれないが、検索にも使える。  

## inputタグを作るためのヘルパー

大抵、`text_field_tag`といったように、`_tag`で終わる。  
最初の引数は、必ずinputタグのnameに該当するものと決まっている。  

これは、Railsのparamsでいうところのkeyに該当する。  
なので、最初の引数がmiketaだった場合、`params[:miketa]`で必要なパラメータを取得できる。  
そのように、Railsがよしなにinputタグを作ってくれる。  

## inputタグがたくさんあるから、ヘルパーもたくさんある

説明はしませんが、`input type`で指定できる種類は、想像以上にある。  

各種のヘルパーの説明は、pikawakaやRailsAPIドキュメントにお任せしますが、  
とりあえず、Railsガイドに掲載されていたものを全て貼っておく。  

電話番号、URLにも専用のフォームフィールドがある。  
意識しておくとよさそう。（text_fieldでもどうにかなりそうだけれど）  

あと、あくまでこれらはよしなにHTMLタグを生成してくれるものなので、  
細かいオプションについて、HTMLのドキュメントも参照するとよい。  

```erb
<%= check_box_tag(:pet_dog) %>
<%= radio_button_tag(:age, "child") %>
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

## モデルオブジェクトヘルパー

モデルと紐づくモデルオブジェクトヘルパーは、`_tag`で終わらない。  
例えば、`<%= text_field(:person, :name) %>`など。  

モデルと紐づかないフォームヘルパーの場合、最初の引数はinputのnameであったが、  
モデルオブジェクトヘルパーの場合、以下のとおりとなっている。  

- １番目の引数：インスタンス変数名 （必ずしもモデル名とイコールではない）
- ２番目の引数：オブジェクトを呼び出すためのメソッド名（attributesであることが大半）
  - HTTPメソッドと誤解しないように
  - Postモデルのbodyというattributeを呼び出すのであれば、bodyがメソッド名になる
  - メソッド名ということは、attributeじゃなくてもいけるらしい（え！）

なお、２つの引数を組み合わせて、inputのnameを生成する。  
参考事例をRailsガイドから引用する。  

なお、idは１番目と２番目の引数をアンスコでつなげる形で自動生成する。  

```erb
<%= text_field(:person, :name) %>
```

```html
<input id="person_name" name="person[name]" type="text" value="Henry"/>
```

nameの値を取得する場合、`params[:person][:name]`で取得する。  

### form_withもしくはform_for（`@instance`と書くパターンで、よく見るやつ）

これがよく見慣れているやつ。  

```erb
<%= form_with @article, url: {action: "create"}, html: {class: "nifty_form"} do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :body, size: "60x12" %>
  <%= f.submit "Create" %>
<% end %>
```

ここでは、text_fieldの1番目の引数がメソッド名（attributes以外もありえるかは不明）になっている。  
インスタンス変数名ではないので、注意。  

で、さらにすごそうなのがここ。  

> fields_forメソッドを使うと、<form>タグを実際に作成せずに同様のバインディングを設定できます。  
> これは、同じフォームで別のモデルオブジェクトも編集できるようにしたい場合などに便利です。  
> たとえば、Personモデルに関連付けられているContactDetailモデルがあるとすると、  
> 以下のようなフォームを作成すればよいのです。  

```erb
<%= form_for @person, url: {action: "create"} do |person_form| %>
  <%= person_form.text_field :name %>
  <%= fields_for @person.contact_detail do |contact_detail_form| %>
    <%= contact_detail_form.text_field :phone_number %>
  <% end %>
<% end %>
```

え、試してないのでよくわかってないけど、postに紐づくcommentモデルに対してのフォームであっても、  
ネストさせれば同時に更新できるってことですか。。。  

accepts_nested_attributes_forやフォームオブジェクトの代わりになるのか。。。  
よくわからん。  

いや、むしろaccepts_nested_attributes_forやフォームオブジェクトを使う際に使うってことっぽい。  
こればっかりは、実装してみないとよく分からない。  

- [fields\_forの上手な使い方 \- Qiita](https://qiita.com/kouuuki/items/5daf2b5f34273d8457f7)
- [Rails ネストした関連先のテーブルもまとめて保存する \(accepts\_nested\_attributes\_for、fields\_for\) \- Qiita](https://qiita.com/shizuma/items/6f56ca442111ece021b5)

## レコード識別（レコードの状態でHTTPメソッドやアクセス先のURLをよしなに指定してくれる）

`routes.rb`でresourcesもしくはresourceと宣言しておくと便利。  

```rb
## 既存の記事の修正
# 長いバージョン
form_for(@article, url: article_path(@article), html: {method: "patch"})
# 短いバージョン
form_for(@article)
```

普段は短いバージョンに慣れすぎているけど、やっぱりこれは便利！  
createなのか、updateなのか推測してくれる。URLも入力不要！  

namespaceの場合、この書き方でいける！  

## セレクトボックスを簡単に作成する

セレクトボックスをHTML直書きで書くのは大変。  
けど、フォームヘルパーを使うと簡単にできる。  

```html
<select name="city_id" id="city_id">
  <option value="1">Lisbon</option>
  <option value="2">Madrid</option>
  ...
  <option value="12">Berlin</option>
</select>
```

まず、単純なselect_tag。この段階では全く楽できない。  
parameterのkeyを第１引数で指定して、タグは全部各方式である。  

```erb
<%= select_tag(:city_id, '<option value="1">Lisbon</option>...') %>
```

options_for_selectを使うと、こんな感じでいける。  
まだ直書きしなくちゃいけないけど、多少楽になる。  

```erb
<%= select_tag(:city_id, options_for_select([['Lisbon', 1], ['Madrid', 2], ...]) %>
```

options_for_selectの第２引数を指定すると、最初に選択されているオプションを選ぶことができる。  
例えば、こんな感じで。  

```html
<%= options_for_select([['Lisbon', 1], ['Madrid', 2], ...], 2) %>

上のコードから以下の出力が得られます。  

<option value="1">Lisbon</option>
<option value="2" selected="selected">Madrid</option>
```

とりあえず、使いどころがわかってないのでスルーするが、  
ハッシュで任意の値を渡すこともできるらしい。  

## 脱線： content_tag

Railsガイドに突如として現れて、全く知らなかったので調べてみた。  
これは、すごい！！  

- [Rails tips: ビューの\`content\_tag\`のあまり知られていないオプション（翻訳）｜TechRacho](https://techracho.bpsinc.jp/hachi8833/2018_04_10/54701)

```erb
content_tag(:div) do
  content_tag(:strong, "header")
end
#=> "< div >< strong >header< /strong >< /div >"
```

## モデルを扱うセレクトボックス

モデルと紐づけて、セレクトボックスを生成することができる。  

```rb
<%= select(:person, :city_id, [['Lisbon', 1], ['Madrid', 2], ...]) %>
```

ここでのポイント。  

- モデルを扱っているので、selectとなり、`_tag`は取れる  
- ユーザーが既に選択している選択肢を選択済みにしてくれる
  - 具体的には、Person.find_by(city_id: 2)がインスタンス変数として渡されたとする
  - その場合、city_idが2のオプションが選択済になる

フォームビルダーでセレクトボックスを使うこともできる。  
え、フォームビルダー。。。？  

ここの最初の方を読むといい。  

- [Railsで独自のフォームビルダーを作ろう！ \| TECHSCORE BLOG](https://www.techscore.com/blog/2013/01/07/rails%E3%81%A7%E7%8B%AC%E8%87%AA%E3%81%AE%E3%83%95%E3%82%A9%E3%83%BC%E3%83%A0%E3%83%93%E3%83%AB%E3%83%80%E3%83%BC%E3%82%92%E4%BD%9C%E3%82%8D%E3%81%86%EF%BC%81/)

```erb
<%= form_for(@user) do |form| %>
  # ブロック引数の form がフォームビルダーです
<% end %>
```

要は、form_withの中に組み込む場合も使えるよって意味。  
使い方は、こんな感じ。  

```erb
<%= f.select(:city_id) do %>
  <% [['Lisbon', 1], ['Madrid', 2]].each do |c| -%>
    <%= content_tag(:option, c.first, value: c.last) %>
  <% end %>
<% end %>
```

なお、has_manyではなく、belongs_toの関連付けを設定する場合、  
モデル名そのものではなく、`city_id`のように外部キーを渡す必要がある。  

あと、外部キーを直接操作できると、セキュリティ上の問題が生じると書いてある。  
全然ピンとこない。。。  

