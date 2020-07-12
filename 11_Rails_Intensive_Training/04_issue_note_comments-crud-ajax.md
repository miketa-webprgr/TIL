# form_withを用いた非同期での実装

## 事前学習

非同期・AjaxについてはQiita記事等を調べてイメージが付いた。  
ただ、具体的な実装方法、細かい仕組みについては理解するのが難しい。  

まず、以下の記事を読んでみた。  
今回の実装内容にかなり近い。  

> - [Rails5と BootstrapでAjax\-modalform \- Qiita](https://qiita.com/niwaken/items/ffbce52fb024fd369f24)  
> - [Ajaxを用いた動的なコメント投稿・削除機能の実装で学ぶRuby on Rails \- 銀行員からのRailsエンジニア](https://ysk-pro.hatenablog.com/entry/2018/02/10/101739)  
> - [Rails Bootstrap with Modal Form \- Qiita](https://qiita.com/tsunemiso/items/edbc58becf55875c4fdb)  

また、こちらの記事も参考になりそうだった。  
実装内容は若干先ほどの記事よりずれてくるかもしれないが、説明が丁寧だった。

> - [Ajax\(非同期通信\)についてわかりやすさ重視でまとめてみた\(Rails使用のデモ付\) \- Qiita](https://qiita.com/__tambo__/items/409ccf256e84017ea307)  
> - [Railsで remote: true と js\.erbを使って簡単にAjax\(非同期通信\)を実装しよう！\(いいね機能のデモ付\) \- Qiita](https://qiita.com/__tambo__/items/45211df065e0c037d032)  

若干混乱してくるが、事前知識としてはそれなりについたように思う。  

## 今回のケース（コードリーディング）

### `posts/show.html.slim`について

今回の場合、`posts/show.html`と`posts_controller.rb`のshowアクションが修正される。  
その一部分に対して、`_comments.html.slim`と`_form.html.slim`がrenderされる。  

まず、と`posts_controller.rb`のshowアクションだが、  
以下のようなコードとなる。

```rb:posts_controller.rb
# posts_controller.rb の showアクションのみ
def show
  @post = Post.find(params[:id])
  @comments = @post.comments.includes(:user).order(created_at: :desc)
  @comment = Comment.new
end
```

ここで、該当の投稿に紐づく一連のコメントをインスタンス変数に格納している。  
これにより、一連のコメントが表示される。  

また、新しくコメントのインスタンスを生成することにより、  
新しいコメントの投稿を投稿することを可能にしている。  

次に、`views/posts/show.html`だが、スクリーンショットに該当コードを貼ってみた。  
分かりやすいのか、微妙なところだけれど。  

<img src="04_issue_note.png" width=500px border=1><br>  

### `_comments.html.slim`について

`_comments.html.slim`であるが、内容はシンプルである。  
端的に@commentsの内容を表示しているだけである（@commentsは、commentsとして引き渡されている）。  

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

つまり、先ほどのコードは、以下のとおりとなる。  
ここで、`_comment.html.slim`が呼び込まれていることが分かる。  

```slim:_comments.html.slim
.comments-box
  - @comments.each do |comment|
    = render partial: 'tweet', locals: {comment: comment}
```

繰り返しが行われる`_comment.html.slim`であるが、以下のとおりとなっている。  
`@comments`に含まれている一連のcommentを一つずつ表示していく。  

なお、`remote: true`となっている箇所でajax通信が使われることに着目するとよい。  

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
そして、従来どおり、`-`を使うことでrubyのロジックを書くことができる。  
また、`=`を使うことで、rubyのロジックを使って何かを表示することができる。  

次にややこしいのが`|`であるが、これはテキストを直接書く場合に必要となる。
そして、alert内にまたRubyのロジックを書くので、`#{ }`を使っている。頭がおかしくなりそうだ。。。  

JS関係で分からないものが、`$`と`prepend`と`val('')`の箇所になるかと思う。  
あと、`\n`と`j render`についても気になるかと思う。  

`$`については、jQueryでは頻出する。
DOM構造を読み終わってから発火するという意味を持つ。  
これはissue02のSwiperの実装などでも扱ったものである。  

`prepend`については、以下の記事を参照するとよい。  

> -[prepend\(content\) \- jQuery 日本語リファレンス](http://semooh.jp/jquery/api/manipulation/prepend/content/)  
> -[【jQuery入門】prepend\(\)の使い方と要素の先頭に追加する方法まとめ！ \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/46746)

`val('')`については、HTMLタグ内に記述されているvalue属性を取得したり変更したりするものである。  
`val('')`が含まれる行のコードは、投稿後に、投稿フォームに入力した内容を空にするために加えられている。  

`\n`については、改行を意味する。改行コードというらしい。  
改行コードではなく、`<br>`タグを埋め込む方法もあるもよう。  

> -[【JavaScript入門】joinで配列を連結する方法\(改行/置換\) \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/23137)  

`j render`について調べたところ、以下の記事を発見した。  
`j`とは`escape_javascript`のエイリアスメソッドらしく、改行コード、シングルクオート、ダブルクオートを  
JavaScript用にエスケープしてくれるActionViewのヘルパメソッドである。  

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

最初の１行目については、`#modal-container`の`modal_form`への差し替えを意味する。  
（なお、`#modal-container`は、`shared/_modal.html.slim`内の特定箇所を指し示している。）  

ここで、escape_javascriptと書かれており、先ほどの箇所との統一感が損なわれいるが、  
特に意図はないと思われる。  

次の２行目については、`#comment-edit-modal`をモーダルで表示するのを意味する。  
（なお、`#comment-edit-modal`は、`comments/_modal_form.html.slim`内の特定箇所を指し示している。）  

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

### `destroy.js.slim`について
