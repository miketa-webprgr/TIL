# Issue08 ユーザーの詳細ページに投稿一覧を表示する

## どんな感じ？

ユーザーの詳細ページに同ユーザーの投稿を一覧表示させる。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/310fd085d2dddf3444c65d80f3f4f5e0.gif" alt="Image from Gyazo" width="500"/></a></a><br>  

投稿写真のサムネイルをクリックすると、該当の投稿ページにアクセスできる。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/fcedcb0dfac8c88e2319882f8feea939.gif" alt="Image from Gyazo" width="500"/></a></a><br>  

## 求められている機能実装・実装条件について

既にgifにて表示したとおり、以下を実装する。  

- ユーザーの詳細ページで同ユーザーの投稿を一覧表示させる。  
- 投稿写真のサムネイルをクリックすると、該当の投稿ページにアクセスできる。  

なお、今回の実装にあたっては、サムネイルが上手く表示できるようcssを追記する。  

また、今回のメインの実装ではないが、ヘッダーのユーザーアイコンをクリックすると、  
自分のユーザー詳細ページにアクセスできるようリンクを設定する。  

## コードリーディング

以下では、コードリーディングを行っていく。  
実装自体は難しいものではないが、renderメソッドやimage_tagメソッドについて、改めて確認しながら進めていく。  

## users/show.html.slimの実装

現時点においては、以下のような状態となっている。  
ここにおいて、投稿のサムネイルが表示されるよう実装を行なっていく。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/5d4e270f0bbe00c0ddf9ea774db6597a.png" alt="Image from Gyazo" width="500"/></a></a><br>  

実装内容は以下のとおりとなっている。  

```slim
/ users/show.html.slim

.container
  .row
    .col-md-6.offset-md-3
    .col-md-10.offset-md-1
      .card
        .card-body
          .text-center.mb-3
            = @user.username
          .text-center
            = render 'follow_area', user: @user

        / 以下からが追記内容
          hr
          .row
            = render partial: 'posts/thumbnail_post', collection: @user.posts
```

renderメソッドであるが、繰り返し表示する場合、  
これまで以下のとおり書くことが多かった。  

```rb
render @posts
```

これは、以下の省略系である。  

```rb
render partial: 'post', collection: @posts
```

この省略系は、postディレクトリ内の`_post.html.slim`ファイルを読み込み、  
インスタンス変数postを引き渡す場合には使えるが、この条件に合致しない場合、  
きちんとpartialファイル名やcollectionオプションについて指定してあげる必要がある。  

なお、パーシャル側で使うローカル変数名は、パーシャル名になる。  
collectionの単数系と勘違いしないように注意すること。  

詳細は、こちらのサイトなどから確認できる。  

- [【Rails】部分テンプレートの使い方を徹底解説！ \| Pikawaka \- ピカ1わかりやすいプログラミング用語サイト](https://pikawaka.com/rails/partial_template#%E7%9C%81%E7%95%A5%E3%81%97%E3%81%9F%E6%9B%B8%E3%81%8D%E6%96%B9)
- [3.4.5 コレクションをレンダリングする --- レイアウトとレンダリング \- Railsガイド](https://railsguides.jp/layouts_and_rendering.html#%E3%82%B3%E3%83%AC%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E3%83%AC%E3%83%B3%E3%83%80%E3%83%AA%E3%83%B3%E3%82%B0%E3%81%99%E3%82%8B)

## users/_thumbnail_post.html.slimの実装

続いて、renderされるparitialファイルの作成を行う。  
コードは下記のとおりとなっている。  

```slim
/ users/_thumbnail_post.html.slim

.col-md-4.mb-3
  = link_to post_path(thumbnail_post), class: 'thumbs' do
    = image_tag thumbnail_post.images.first.url
```

以上においては、image_tagを活用して以下を実現している。  

- 該当の投稿へのリンクを貼っている。  
- 該当の投稿の写真（最初に投稿した写真）を表示する。  

`thumbnail_post.images.first.url`という部分については長くて少し面を食らったが、  
順を追ってゆっくりと考えていけば理解ができる内容である。  
（あと、carrierwaveを導入しているため、urlメソッドが使えるということを思い出そう）  

## application.scssの実装

先ほどの作業により、投稿のサムネイルがユーザーの詳細画面に表示されるようになった。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/b5f837a6567eca14bc1466f5edc7f25a.png" alt="Image from Gyazo" width="500"/></a></a><br>  

ただし、以上のとおり微妙な歪みが表示されてしまっているので、scssで整える必要がある。  

```scss
// application.scss
// 今回追加する部分のみ記載

.thumbs {
  width: 100%;
  position: relative;
  display: block;

  &::before {
    content: "";
    display: block;
    padding-top: 100%;
  }

  img {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    object-fit: cover;
  }
}
```

すると、以下のとおり、綺麗に整えることができた。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/01f1e4213c990d4a7816a41f524a3c09.png" alt="Image from Gyazo" width="500"/></a></a><br>  

なお、CSSがどのように作用して、以下のように整えることができるかについては理解できていないが、  
今回の本題ではないのでそのまま次に進むこととする。  

ざっと確認した限り、`object-fit`というトリミングを中央からしてくれる便利なものがあり、  
それを活用しているらしい。  

- [CSSのobject\-fitで画像を切り抜き・リサイズする](https://code-kitchen.dev/css/object-fit/)
- [HTML・CSS レスポンシブな正方形を作って、その中央にコンテンツを表示させたい。 \- かもメモ](https://chaika.hatenablog.com/entry/2016/07/15/163004)

また、見慣れないものとして、`&::before`があったが、これで擬似要素を追加できるらしい。  
ざっくりというと、htmlを書くことなく、cssでhtml的な内容を追加できるみたいな感じらしい。  
調べると、詳細が色々と出てくる。  

- [CSSの疑似要素とは？beforeとafterの使い方まとめ](https://saruwakakun.com/html-css/basic/before-after)

## ヘッダーにユーザーの詳細画面へのリンクを貼付

ヘッダーにてユーザーの詳細画面へのリンクを貼付する。  
アイコンをクリックすると、現在ログインしているユーザーの詳細画面にアクセスさせる。  

```slim
/ shared/_header.html.slim
/ 該当部分のみ抜粋

li.nav-item
  = link_to user_path(current_user), class: 'nav-link' do
    = icon 'far', 'user', class: 'fa-lg'
```

## ユーザーの詳細画面のプロフィール部分を拡大する

なお、Bootstrapに関する部分であるため、特に解説はしないが、  
サイズが小さかったため、以下のとおり、サイズを変更した。  

```slim
/ Before

.col-md-6.offset-md-3

/ After
.col-md-10.offset-md-1
```
