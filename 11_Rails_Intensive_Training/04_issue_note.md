# Issue04 投稿に対するコメントのCRUD機能実装

## どんな感じ？

<a href="https://gyazo.com/9b35f2fdd2a162deb985a4f85bdf7854"><img src="https://i.gyazo.com/9b35f2fdd2a162deb985a4f85bdf7854.gif" alt="Image from Gyazo" width="500" border=1/></a><br>  

<a href="https://gyazo.com/cf4e1a130fbdd0ee89ec6ed43f7b1eb1"><img src="https://i.gyazo.com/cf4e1a130fbdd0ee89ec6ed43f7b1eb1.gif" alt="Image from Gyazo" width="500" border=1/></a><br>  

## 求められている機能実装について

1. 投稿に対するコメントのCRUD機能実装
2. 編集・更新はモーダルを表示させ非同期で行う（form_with利用）
3. 文字列長など適切なバリデーションを付与する
4. shallowルーティングを使用する

## 具体的な実装手順について

長くなったので別ファイルとした。  
コードリーディングを頑張ってやってみた。  

> - [コメント機能（非同期）の実装手順について](04_issue_note_comments-crud-ajax.md)

## 分からない単語・概念等の一覧

- １対多のAssociation（投稿に対するコメントの紐付け）
- モーダル
- 非同期・Ajax通信
- 文字列長のバリデーション
- shallowルーティングとは

## １対多のAssociation

以下のQiita記事が参考になった。  

> - [【初心者向け】丁寧すぎるRails『アソシエーション』チュートリアル【幾ら何でも】【完璧にわかる】🎸 \- Qiita](https://qiita.com/kazukimatsumoto/items/14bdff681ec5ddac26d1#user%E3%81%A8user%E3%81%AE%E5%A4%9A%E5%AF%BE%E5%A4%9Amn%E3%82%92%E8%A8%AD%E8%A8%88%E3%81%97%E3%82%88%E3%81%86%E8%87%AA%E5%B7%B1%E7%B5%90%E5%90%88)  

## モーダルとは

イメージとしては以下が分かりやすい。  
ポップアップ的な感じ。

> -[\[Rails\]モーダルウィンドウを作成する\(データを削除前に再度確認する\) \- Qiita](https://qiita.com/takachan_coding/items/9179cf361d0e92ae0bad)  

また、以下の記事ではこのように説明されていた。

> ウィンドウ内で指定された操作を完了、またはキャンセルするまで他のウィンドウを開くことができないウィンドウ  
>
> - [モーダルウィンドウとは？モーダル表示の役割とデメリットについて](https://www.seohacks.net/basic/terms/modal-window/)

## 非同期・Ajax通信とは

ぬるっと早い。  

同期処理だと、そのページを表示するのに必要なHTML, CSS, JS などを順を追って、改めて全て取得するが、  
非同期処理だと、それぞれの処理を同時多発的に行うことができる。（HTML・CSS・JSを一斉に取得するようなイメージ？）  

非同期処理の応用（？）として、該当のHTMLの一部分のみを更新する、ということもできる。  
その場合、DOM（Document Object Model）構造を把握して、該当部分のみを更新する。  

HTMLだと`<html>`というタグに始まり、`<body>`というタグがあり、その中に`<div>`タグがある  
という構造になっているが、非同期処理を活用すると、その`<div>`タグの〜クラスだけ更新できる。  

これは、Ajaxという技術によって実現される。  
Ajaxとは、Asynchronized JS + XML(JSON)の略。  
「JSとXMLを使って非同期通信行うよ」ということを意味する。  
ただ、実際はXMLではなくて、JSONを使うことが多い。  

裏側で行われている処理を分解すると、以下のとおり。  

- JSが指定の条件に合致した場合に発火する
- クライアント側にてAjaxエンジンがxmlHTTPリクエストをサーバーを送る
- サーバーがXMLもしくはJSONを返す
- AjaxエンジンがHTMLの該当部分のみ書き換える

> - [初心者目線でAjaxの説明 \- Qiita](https://qiita.com/hisamura333/items/e3ea6ae549eb09b7efb9)  
> - [非同期通信Ajaxをできるだけ分かりやすく説明してみた](https://applingo.tokyo/article/654)  

## 文字列長のバリデーション

以下を参照するとよい。  

> - [Active Record バリデーション \- Railsガイド](https://railsguides.jp/active_record_validations.html#length)

具体的には、モデルを以下のとおり修正する。  

```rb:commnet.rb
class Comment < ApplicationRecord
  belongs_to :user
  belongs_to :post

  # length~を追加した
  validates :body, presence: true, length: { maximum: 1000 }
end
```

## shallowオプション

以下に説明がある。  

> コレクション (index/new/createのような、idを持たないアクション) だけを  
> 親のスコープの下で生成するという手法があります。  
>
> このとき、メンバー (show/edit/update/destroyのような、idを必要とするアクション)  
> をネストに含めないのがポイントです。  
> [Rails のルーティング \- Railsガイド](https://railsguides.jp/routing.html#%E3%80%8C%E6%B5%85%E3%81%84%E3%80%8D%E3%83%8D%E3%82%B9%E3%83%88)  

なお、コードは以下のとおりとなる。  
これにより、コレクションの場合は親のスコープの下で生成する。  

```rb:routes.rb
resources :posts do
  resources :comments, shallow: true
end
```

つまり、ルーティングテーブルは下記のとおりとなる。  

```text
   post_comments GET    /posts/:post_id/comments(.:format)         comments#index
                 POST   /posts/:post_id/comments(.:format)         comments#create
new_post_comment GET    /posts/:post_id/comments/new(.:format)     comments#new
    edit_comment GET    /comments/:id/edit(.:format)               comments#edit
         comment GET    /comments/:id(.:format)                    comments#show
                 PATCH  /comments/:id(.:format)                    comments#update
                 PUT    /comments/:id(.:format)                    comments#update
                 DELETE /comments/:id(.:format)                    comments#destroy
```

shallowオプションのメリットは、リソースが一意であることを保ちながら、  
URLを極力短くすることができる点にある。  

例えば、indexアクションについては、どのpostに紐づくか示さないと一意に保つことができない。  
`/posts/:post_id`の部分は削ると、Aさんの〜投稿に対してのコメントであると示すことはできない。  
これは、createアクションやnewアクションについても同様である。  

他方、editアクションについては、`/posts/:post_id`がなくとも一意であることを示すことができる。  
`/comments/:id`と`/:id`でどのコメントか明らかにできるため、どの投稿に属するか示す必要がない。  
これは、showアクション、updateアクション、destroyアクションについても同様である。  

## 参考

> - [【初心者向け】丁寧すぎるRails『アソシエーション』チュートリアル【幾ら何でも】【完璧にわかる】🎸 \- Qiita](https://qiita.com/kazukimatsumoto/items/14bdff681ec5ddac26d1#user%E3%81%A8user%E3%81%AE%E5%A4%9A%E5%AF%BE%E5%A4%9Amn%E3%82%92%E8%A8%AD%E8%A8%88%E3%81%97%E3%82%88%E3%81%86%E8%87%AA%E5%B7%B1%E7%B5%90%E5%90%88)  
> - [\[Rails\]モーダルウィンドウを作成する\(データを削除前に再度確認する\) \- Qiita](https://qiita.com/takachan_coding/items/9179cf361d0e92ae0bad)  
> - [モーダルウィンドウとは？モーダル表示の役割とデメリットについて](https://www.seohacks.net/basic/terms/modal-window/)  
> - [初心者目線でAjaxの説明 \- Qiita](https://qiita.com/hisamura333/items/e3ea6ae549eb09b7efb9)  
> - [非同期通信Ajaxをできるだけ分かりやすく説明してみた](https://applingo.tokyo/article/654)  
> - [Active Record バリデーション \- Railsガイド](https://railsguides.jp/active_record_validations.html#length)
> - [Rails のルーティング \- Railsガイド](https://railsguides.jp/routing.html#%E3%80%8C%E6%B5%85%E3%81%84%E3%80%8D%E3%83%8D%E3%82%B9%E3%83%88)  
> - [Rails5と BootstrapでAjax\-modalform \- Qiita](https://qiita.com/niwaken/items/ffbce52fb024fd369f24)  
> - [Ajaxを用いた動的なコメント投稿・削除機能の実装で学ぶRuby on Rails \- 銀行員からのRailsエンジニア](https://ysk-pro.hatenablog.com/entry/2018/02/10/101739)  
> - [Rails Bootstrap with Modal Form \- Qiita](https://qiita.com/tsunemiso/items/edbc58becf55875c4fdb)  
> - [Ajax\(非同期通信\)についてわかりやすさ重視でまとめてみた\(Rails使用のデモ付\) \- Qiita](https://qiita.com/__tambo__/items/409ccf256e84017ea307)  
> - [Railsで remote: true と js\.erbを使って簡単にAjax\(非同期通信\)を実装しよう！\(いいね機能のデモ付\) \- Qiita](https://qiita.com/__tambo__/items/45211df065e0c037d032)  
> - [【Rails基礎】ややこしい部分テンプレートの省略形について簡単にまとめてみた｜TechTechMedia](https://techtechmedia.com/partial-template-rails/)  
> - [\[Rails\] 部分テンプレート使用時のrenderの書き方 \- Qiita](https://qiita.com/kojiro3/items/4ce52494a4e69bc443a2)  
> - [hamlでの部分テンプレートの呼び出し方の種類（省略形含む） \- Qiita](https://qiita.com/mgmmy/items/4de27770338ac3194e33)  
> - [【Rails】form\_with/form\_forについて【入門】 \- Qiita](https://qiita.com/snskOgata/items/44d32a06045e6a52d11c#23-form_with-model-modela-modelb)  
> - [KozakuraRyuuichiさんとだいそんさんのやり取り](https://github.com/KozakuraRyuuichi/trn_insta_clone/pull/6#discussion_r433299695)  
> - [prepend\(content\) \- jQuery 日本語リファレンス](http://semooh.jp/jquery/api/manipulation/prepend/content/)  
> - [【jQuery入門】prepend\(\)の使い方と要素の先頭に追加する方法まとめ！ \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/46746)  
> - [RailsでAjax基本形（Scaffoldで学ぶ） \- Qiita](https://qiita.com/mm36/items/684f36f22e79d0a27ae9)  
> - [ActionView::Helpers::JavaScriptHelper(escape_javascript)](https://api.rubyonrails.org/classes/ActionView/Helpers/JavaScriptHelper.html#method-i-escape_javascript)  
> - [Action View の概要 \- Railsガイド](https://railsguides.jp/action_view_overview.html#javascripthelper)  
