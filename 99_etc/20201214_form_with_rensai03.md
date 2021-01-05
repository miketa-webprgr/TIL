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
まず雰囲気を掴むところから始める。  

```erb
<%= form_with @article, url: articles_path do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :body, size: "60x12" %>
  <%= f.submit "Create" %>
<% end %>
```

これまで取り上げたヘルパーは、検索など、モデルに紐づかないものだが、  
今回は、モデル（≒テーブル）に紐づくヘルパー。  

例えば、検索フォームの場合、テーブルに保存しているレコードを取ってきて、  
そのレコードの情報をフォームに表示させる必要はない。  

ただ、記事を作成したり更新する場合、モデルと連携して、記事のタイトルを表示したり、  
保存したり、更新したり、削除したりする必要が出てくる。  

そこで、form_withやform_forを使わずにコードを書こうとすると、えらく大変なる。  
例えば、こんな感じで書いてみたとする。（思いつきで書いてあるので、突っ込みどころが多そう）  

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
    <%= submit_tag "Update" %>
  <% end %>
<% end %>

# 本来は、`name="article[name]"`とし、`id="article_name"`としたいが、  
# すぐに書き方が分からず断念した。  

# 以上のようなerbファイルだと特にnameが`article[〜]`でまとまらないので、  
# controllerで`params.require(:article)`と書くことができず面倒なことになる。 
```

・・・とまあ、ごちゃごちゃと書いたが、

- モデルと紐づける上で`form_with model: @model`という書き方は便利
- nameやidを規則に沿って付与してくれる
- @modelがnew_recordであるか判断して、いい感じにリクエスト先やmethodも決めてくれる

以上を理解しておくとよい。  

**railsで何かパラメータを送る際は、`form_with model`を使うと決めつけてしまうと、**  
**訳が分からなくなってしまうので、あくまでモデルと紐づける上で便利な一つの手法だと認識する。**  

## form_withメソッドを理解する

さて、ここでRailsガイドに書いている内容から少し外れて、form_withのメソッドの概要を理解する。  
メソッドは、このような形式となっている。  

```rb
form_with(model: nil, scope: nil, url: nil, format: nil, **options)
```

まず、model、scope、url、format、optionsが何を意味しているか確認する。

- model
  - 紐づくモデルを指定する
  - 例えば`@user`など
- scope
  - パラメータの送信先のパスの頭に何かを付け加えたい場合に指定する
  - 例えば、`User.new`の場合、scopeを設定しないと、パラメータは`/users`のパスに送信される
  - これを`admin/users`のパスに送りたい場合、`scope: :admin`とする
  - `routes.rb`にてnamespaceで区切っているような場合なんかによく使う
- url
  - パラメータの送信先のパスを指定する
  - `routes.rb`にてresources

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
