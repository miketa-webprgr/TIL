# form_withを用いた非同期での実装

## 事前学習

非同期・AjaxについてはQiita記事等を読んでイメージを掴むことができた。  
ただ、具体的な実装方法、細かい仕組みについては理解するのが難しい。  

そこで、まず以下の記事を読んでみた。  
今回の実装内容にかなり近い。  

> - [Rails5と BootstrapでAjax\-modalform \- Qiita](https://qiita.com/niwaken/items/ffbce52fb024fd369f24)  
> - [Ajaxを用いた動的なコメント投稿・削除機能の実装で学ぶRuby on Rails \- 銀行員からのRailsエンジニア](https://ysk-pro.hatenablog.com/entry/2018/02/10/101739)  
> - [Rails Bootstrap with Modal Form \- Qiita](https://qiita.com/tsunemiso/items/edbc58becf55875c4fdb)  

また、こちらの記事も参考になりそうだった。  
今回の実装内容とずれてくる部分があるが、説明が丁寧だった。  

> - [Ajax\(非同期通信\)についてわかりやすさ重視でまとめてみた\(Rails使用のデモ付\) \- Qiita](https://qiita.com/__tambo__/items/409ccf256e84017ea307)  
> - [Railsで remote: true と js\.erbを使って簡単にAjax\(非同期通信\)を実装しよう！\(いいね機能のデモ付\) \- Qiita](https://qiita.com/__tambo__/items/45211df065e0c037d032)  

## 今回のケース（コードリーディング）

### `posts/show.html.slim`について

今回の場合、`posts/show.html`と`posts_controller.rb`のshowアクションを修正する。  

`posts/show.html`だが、スクリーンショットに該当コードを貼ってみた。  
分かりやすいのか、微妙なところだけれど、まず実際の画面を見ながらイメージを掴む。  

<img src="04_issue_note.png" width=500px border=1><br>  

なお、`posts/show.html`だが、その一部分で以下が`render`されるので確認しておく。  

- `_comments.html.slim`
- `_form.html.slim`

次に、`posts_controller.rb`のshowアクションを確認する。  
このアクションで定義したインスタンス変数は、`posts/show.html`に引き継がれる。   

```rb:posts_controller.rb
# posts_controller.rb の showアクションのみ
def show
  @post = Post.find(params[:id])
  @comments = @post.comments.includes(:user).order(created_at: :desc)
  @comment = Comment.new
end
```

`@post`の部分には既に作った部分なので省略する。  
ここは投稿自体に関係する部分であり、コメントには直接関係してこない。  

`@comments`だが、ここで該当の投稿に紐づく一連のコメントをインスタンス変数に格納している。  
これにより、一連のコメントが表示される。  

また、新しくコメントのインスタンスを生成することにより、  
新しいコメントの投稿を投稿することを可能にしている。  

### `_comments.html.slim`について

`_comments.html.slim`であるが、内容はシンプルである。  

```slim:_comments.html.slim
.comments-box
  = render comments
```

`render comments`の部分であるが、奥が深い。  
激しく色々なことが省略されている。  

まずは、この記事を読む必要がある。  
「複数形のインスタンス変数を渡す」を参照する必要がある。  
また、最初の記事だけで十分だとは思うが、それ以外の記事も参照してよい。  

> -[【Rails基礎】ややこしい部分テンプレートの省略形について簡単にまとめてみた｜TechTechMedia](https://techtechmedia.com/partial-template-rails/)  
> -[\[Rails\] 部分テンプレート使用時のrenderの書き方 \- Qiita](https://qiita.com/kojiro3/items/4ce52494a4e69bc443a2)  
> -[hamlでの部分テンプレートの呼び出し方の種類（省略形含む） \- Qiita](https://qiita.com/mgmmy/items/4de27770338ac3194e33)  

つまり、先ほどのコードは、以下のものと同義である。  
ここで、`_comment.html.slim`が呼び込まれていることが分かる。  

```slim:_comments.html.slim
.comments-box
  - @comments.each do |comment|
    = render partial: 'tweet', locals: {comment: comment}
```

繰り返しが行われる`_comment.html.slim`であるが、以下のとおりとなっている。  
`@comments`に含まれている一連のcommentを一つずつ表示していく。  

なお、`remote: true`とすることにより、ajax通信が使われるように設定している。  

```slim:_comment.html.slim
div id="comment-#{comment.id}"
  .row.no-gutters
    / ユーザーアイコンを表示
    .col-2
      = image_tag 'profile-placeholder.png', size: '40x40', class: 'rounded-circle'
    / ユーザー名を表示し、その右にコメント本文を表示
    .col-9
      span.font-weight-bold.pr-1
        = comment.user.username
      = comment.body
    / ログインユーザーであれば、編集と削除のボタンを表示
    .col-1
      - if current_user&.own?(comment)
        = link_to comment_path(comment), class: 'mr-3', method: :delete, data: {confirm: '本当に削除しますか？'}, remote: true do
          = icon 'far', 'trash-alt', class: 'fa-sm'
        = link_to edit_comment_path(comment), remote: true do
          = icon 'far', 'edit', class: 'fa-sm'
  hr
```

### `_form.html.slim`について

`_form.html.slim`であるが、コードは以下のとおりである。  

```slim:_form.html.slim
= form_with model: [post, comment], class: 'd-flex mb-0 flex-nowrap justify-content-between', remote: true do |f|
  = f.text_field :body, class: 'form-control input-comment-body', placeholder: 'コメント'
  = f.submit '投稿', class: 'btn btn-primary btn-raised'
```

form_with modelにて、複数のparamsを送っている。  
この記事を参照するとよい。  

> - [【Rails】form\_with/form\_forについて【入門】 \- Qiita](https://qiita.com/snskOgata/items/44d32a06045e6a52d11c#23-form_with-model-modela-modelb)  

以上の記事の内容から、commentが空である時は、commentコントローラーのcreateアクションを呼び出し、  
commentが空でない場合は、updateアクションを呼び出すことが分かる。  

なお、`local: true`ではなく、`remote: true`となっている。  
Railsでは、`create.html.slim`などではなく、`create.js.slim`といったファイルを呼び出している。  

また、`placeholder`とはフォーム内に記入されている入力例のことである。  

### `create.js.slim`について

コードは下記のとおりである。  
まず、slimの書き方について確認したい。  

```slim:create.js.slim
- if @comment.errors.present?
  | alert("#{@comment.errors.full_messages.join('\n')}");
- else
  | $('.comments-box').prepend("#{j render('comments/comment', comment: @comment)}");
  | $('.input-comment-body').val('');
```

まず、`js.slim`なので、これはJSファイルである。  
・・・と思いきや、そうでもないらしく、ここは注意が必要である。  

> - [KozakuraRyuuichiさんとだいそんさんのやり取り](https://github.com/KozakuraRyuuichi/trn_insta_clone/pull/6#discussion_r433299695)  

また、基本的な前提として、以下を踏まえておくとよいだろう。  

- 従来どおり、`-`を使うことでrubyのロジックを書くことができる  
- `=`を使うことで、rubyのロジックを使って何かを表示することができる  
- `-`や`=`を付けない場合、JSのコードを書いていると見なされる
  - `|`であるが、これはテキストを直接書く場合に必要となる
  - JSのコード内にRubyのロジックを書く場合、`#{ }`を使う

あと、基本的な知識がないとそもそもRuby・Rails系のことが書いてあるのか、  
それともJSのことが書いてあるのか勘違いする可能性があるので整理しておく。  

JS関係のメソッドは以下のとおりである。  
また、メソッドではないが、`$`についても踏まえておく必要がある。

- alert
- prepend
- val

`$`については、jQueryでは頻出する。
DOM構造を読み終わってから発火するという意味を持つ。  
これはissue02のSwiperの実装でも扱ったものである。  

`prepend`については、以下の記事を参照するとよい。  

> -[prepend\(content\) \- jQuery 日本語リファレンス](http://semooh.jp/jquery/api/manipulation/prepend/content/)  
> -[【jQuery入門】prepend\(\)の使い方と要素の先頭に追加する方法まとめ！ \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/46746)

`val`については、HTMLタグ内に記述されているvalue属性を取得したり変更したりするものである。  
`val('')`が含まれる行のコードは、投稿後に、投稿フォームに入力した内容を空にするために加えられている。  

あと、JS関係ではないが、`\n`と`j render`についても気になるかと思う。  

`\n`については、改行を意味する。改行コードというらしい。  

`j render`について調べたところ、以下の記事を発見した。  
`j`とは`escape_javascript`のエイリアスメソッドらしく、改行コード、シングルクオート、ダブルクオートを  
JavaScript用にエスケープしてくれるActionViewのヘルパーメソッドである。  

> - [RailsでAjax基本形（Scaffoldで学ぶ） \- Qiita](https://qiita.com/mm36/items/684f36f22e79d0a27ae9)  
> - [ActionView::Helpers::JavaScriptHelper(escape_javascript)](https://api.rubyonrails.org/classes/ActionView/Helpers/JavaScriptHelper.html#method-i-escape_javascript)  
> - [Action View の概要 \- Railsガイド](https://railsguides.jp/action_view_overview.html#javascripthelper)  

### `edit.js.slim`について

```slim:edit.js.slim
| $("#modal-container").html("#{escape_javascript(render 'modal_form', comment: @comment)}");
| $("#comment-edit-modal").modal('show');
```

再掲となるが、以下の記事が参考になる。  

> -[Rails5と BootstrapでAjax\-modalform \- Qiita](https://qiita.com/niwaken/items/ffbce52fb024fd369f24)  
> -[Rails Bootstrap with Modal Form \- Qiita](https://qiita.com/tsunemiso/items/edbc58becf55875c4fdb)  

なお、ここでなぜか`j`ではなく`escape_javascript`と書かれているが、特に意図はないと思われる。  

最初の１行目については、`#modal-container`とidが付与されている箇所の差し替えを意味する。  
次の２行目については、`#comment-edit-modal`をモーダルで表示することを意味する。  

`application.html.slim`において、常に空の`shared/_modal.html.slim`が  
`#modal-container`と  紐付けられる形で`render`されてきたが、JSの力を使ってその状態を差し替え、  
投稿済みのコメントをモーダルウィンドウとして表示させるのがこのコードの狙いである。  

差し替える内容は、`render 'modal_form', comment: @comment`になる。  

`render 'modal_form', comment: @comment`で示される`modal_form`は、  
`comments/_modal_form.html.slim`のことなので、コードの中身を詳しく見てみる。  

```slim:modal_form.html.slim
.modal#comment-edit-modal
  .modal-dialog
    .modal-content
      .modal-header
        h5.modal-title コメント編集
        button.close aria-label="Close" data-dismiss="modal" type="button"
          span aria-hidden="true"  ×
      .modal-body
        = render 'form', post: nil, comment: comment
```

なお、ここではpost変数にはnilを引継ぎ、comment変数には該当ユーザーのcommentのidを引き継ぐ。  

ここで理解する上で重要なのが、以下の２点だ。  

- commentをcreateする場合は、postへの紐付けが必要になるため、postのidを持ってくる必要がある
- commentをedit, update, destroyする場合、そのような作業は不要であるため、postはnilでよい

### `update.js.slim`について

```slim:update.js.slim
- if @comment.errors.present?
  | alert("#{@comment.errors.full_messages.join('\n')}");
- else
  | $("#comment-#{@comment.id}").html("#{j render('comments/comment', comment: @comment)}");
  | $("#comment-edit-modal").modal('hide');
```

まず、if文でエラーがある場合とない場合で分岐していることに着目する。  

エラーがある場合、`alert`（JS）によりポップアップウインドウを表示する。  
エラーメッセージを全て出力し、それぞれ改行して表示する。  

エラーがない場合、`#comment-1`などのidが付与されている部分をJSで置き換える。  
その場合、`_comment.slim.html`をrenderする。  

そして、表示していたモーダルのウィンドウを閉じる。  

なお、`#comment-1`などと書かれている箇所のHTMLは以下のとおりである。  

<a href="https://gyazo.com/93a03badd152a90dfff31063cfb56c0f"><img src="https://i.gyazo.com/93a03badd152a90dfff31063cfb56c0f.png" alt="Image from Gyazo" width="200" border=1/></a>  

### `destroy.js.slim`について

```slim:destroy.js.slim
| $('#comment-#{@comment.id}').remove()
```

これは、`#comment-1`などのidが付与されている部分をJSで取り除くものである。  
これにより、該当箇所が削除される。  
