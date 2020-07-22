# Issue06 フォロー機能を実装

## どんな感じ？

フォロー機能  
<a href="https://gyazo.com/2b0bf1d9a04a1de896bece3531317080"><img src="https://i.gyazo.com/2b0bf1d9a04a1de896bece3531317080.gif" alt="Image from Gyazo" width="500" border=1/></a><br>  

フォローして更新をした後  
<a href="https://gyazo.com/d6d4ee67561c3d0e2adb9defdf580e26"><img src="https://i.gyazo.com/d6d4ee67561c3d0e2adb9defdf580e26.gif" alt="Image from Gyazo" width="500" border=1/></a><br>  

ユーザーの一覧 → ユーザー詳細  
<a href="https://gyazo.com/d21cdaf9b6d4ea202d7795464976a7a9"><img src="https://i.gyazo.com/d21cdaf9b6d4ea202d7795464976a7a9.gif" alt="Image from Gyazo" width="500" border=1/></a><br>  

投稿がない場合は「投稿がありません」を表示  
<a href="https://gyazo.com/4d2d25ca3cd29e9c9c063be5bd3cdc90"><img src="https://i.gyazo.com/4d2d25ca3cd29e9c9c063be5bd3cdc90.gif" alt="Image from Gyazo" width="500" border=1/></a><br>  

## 求められている機能実装・実装条件についてについて

- フォロー・アンフォローは非同期で行う。form_withを利用すること。
- 適切なバリデーションを付与する
- 投稿一覧画面について
  - ログインしている場合
    - フォローしているユーザーと自分の投稿だけ表示させること
  - ログインしていない場合
    - 全ての投稿を表示させること
- 一件もない場合は『投稿がありません』と画面に表示させること
- 投稿一覧画面右にあるユーザー一覧については登録日が新しい順に5件分表示してください
- ユーザー一覧画面、詳細画面も実装すること

## 分からない単語・概念等の一覧

- associationの自己結合について

## 具体的な実装手順について
