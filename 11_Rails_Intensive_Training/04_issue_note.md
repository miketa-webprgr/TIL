# Issue04 投稿に対するコメントのCRUD機能実装

## どんな感じ？

<a href="https://gyazo.com/9b35f2fdd2a162deb985a4f85bdf7854"><img src="https://i.gyazo.com/9b35f2fdd2a162deb985a4f85bdf7854.gif" alt="Image from Gyazo" width="500" border=1/></a><br>  

<a href="https://gyazo.com/cf4e1a130fbdd0ee89ec6ed43f7b1eb1"><img src="https://i.gyazo.com/cf4e1a130fbdd0ee89ec6ed43f7b1eb1.gif" alt="Image from Gyazo" width="500" border=1/></a><br>  

## 求められている機能実装について

1. 投稿に対するコメントのCRUD機能実装
2. 編集・更新はモーダルを表示させ非同期で行う（form_with利用）
3. 文字列長など適切なバリデーションを付与する
4. shallowルーティングを使用する

## 分からない単語・概念等の一覧

- １対多のAssociation（投稿に対するコメントの紐付け）
- モーダル
- 非同期・Ajax通信
- `form_with`を使った非同期通信の実装
- 文字列長のバリデーション
  - モデルでバリデーション足すだけのはず
- shallowルーティングとは

### １対多のAssociation

以下のQiita記事が参考になった。  

> - [【初心者向け】丁寧すぎるRails『アソシエーション』チュートリアル【幾ら何でも】【完璧にわかる】🎸 \- Qiita](https://qiita.com/kazukimatsumoto/items/14bdff681ec5ddac26d1#user%E3%81%A8user%E3%81%AE%E5%A4%9A%E5%AF%BE%E5%A4%9Amn%E3%82%92%E8%A8%AD%E8%A8%88%E3%81%97%E3%82%88%E3%81%86%E8%87%AA%E5%B7%B1%E7%B5%90%E5%90%88)  

### モーダルとは

イメージとしては以下が分かりやすい。  
ポップアップ的な感じ。

> -[\[Rails\]モーダルウィンドウを作成する\(データを削除前に再度確認する\) \- Qiita](https://qiita.com/takachan_coding/items/9179cf361d0e92ae0bad)  

また、以下の記事ではこのように説明されていた。

> ウィンドウ内で指定された操作を完了、またはキャンセルするまで他のウィンドウを開くことができないウィンドウ  
>
> - [モーダルウィンドウとは？モーダル表示の役割とデメリットについて](https://www.seohacks.net/basic/terms/modal-window/)

### 非同期・Ajax通信とは

ぬるっと早い。  

同期処理だと、そのページを表示するのに必要なHTML, CSS, JS などを順を追って、改めて全て取得するが、  
非同期処理だと、それぞれの処理を同時多発的に行うことができる。（HTML・CSS・JSを一斉に取得するようなイメージ？）  

非同期処理の応用（？）として、該当のHTMLの一部分のみを更新する、ということもできる。  
その場合、DOM（Document Object Model）構造を把握して、該当部分のみを更新する。  

HTMLだと`<html>`というタグに始まり、`<body>`というタグがあり、その中に`<div>`タグがある  
という構造になっているが、非同期処理を活用すると、その`<div>`タグの〜クラスだけ更新できる。  

これは、Ajaxという技術によって実現される。  
Ajaxとは、Asynchronized JS + XML(JSON)の略。  
「JSとXMLを使って非同期通信行うよ」を意味する。  
ただ、実際はXMLではなくて、JSONを使うことが多い。  

裏側で行われている処理を分解すると、以下のとおり。  

- JSが指定の条件に合致した場合に発火する
- クライアント側にてAjaxエンジンがxmlHTTPリクエストをサーバーを送る
- サーバーがXMLもしくはJSONを返す
- AjaxエンジンがHTMLの該当部分のみ書き換える

> - [初心者目線でAjaxの説明 \- Qiita](https://qiita.com/hisamura333/items/e3ea6ae549eb09b7efb9)  
> - [非同期通信Ajaxをできるだけ分かりやすく説明してみた](https://applingo.tokyo/article/654)  

## form_withを用いた非同期での実装

長くなったので別ファイルとした。

> - [form_withを用いた非同期での実装手順について](04_issue_note_comments-crud-ajax.md)

## 参考

作成中

## 動作確認方法
