# Issue06 フォロー機能を実装

## どんな感じ？

フォロー機能  
<a href="https://gyazo.com/2b0bf1d9a04a1de896bece3531317080"><img src="https://i.gyazo.com/2b0bf1d9a04a1de896bece3531317080.gif" alt="Image from Gyazo" width="500" border=1/></a><br>  

フォローして更新をした後  
（フォローしたユーザーの投稿 + 自身の投稿のみが表示される）  
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
- 投稿一覧画面の右にあるユーザーの一覧については登録日が新しい順に5件分表示してください
- ユーザーの一覧画面、詳細画面も実装すること

## 分からない単語・概念等の一覧

### associationの自己結合について

こちらの記事が最強なので、改めて熟読することから始めた。  
自己結合についてもまとめてある。  

- [【初心者向け】丁寧すぎるRails『アソシエーション』チュートリアル【幾ら何でも】【完璧にわかる】🎸 \- Qiita](https://qiita.com/kazukimatsumoto/items/14bdff681ec5ddac26d1#%E3%83%95%E3%82%A9%E3%83%AD%E3%83%BC%E3%83%95%E3%82%A9%E3%83%AD%E3%83%AF%E3%83%BC%E6%A9%9F%E8%83%BD%E3%82%92er%E5%9B%B3%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E8%A8%AD%E8%A8%88%E3%81%97%E3%82%88%E3%81%86)

前回のいいね機能実装と同様に、多対多の関係であることは変わらない。  

フォローするユーザーとフォローされるユーザーの関係を表すための中間テーブルを作成し、  
そのテーブルでのデータの有無にて、フォローの関係があるか判断できるようにする。  

ただ、前回と違うのは、UserテーブルのUserが、同じテーブル内の別のユーザーに関係するということである。  
２つのテーブル間での関係ではなく、１つのテーブル内での関係なので、自己結合というらしい。  

この自己結合の関係は、多対多だけではなく、一対多の関係（例：上司と部下の関係）もある。  

詳しくは、Qiita記事を参照するとよいが、自己結合であるため、  
参照する外部キーなどを改めて指定してあげる必要がある。  

例えば、中間テーブルにfollowするuser_idとfollowedされるuser_idが入るRelationshipという  
モデルがある場合、`has_many: Relationship`と書くだけではどちらを外部キーとするのか確定できない。  
そこで、followするユーザーを取得したい場合、`foreign_key: :follow_id`と指定する必要がある。  

## 具体的な実装手順について

長くなったので別ファイルとした。  
[フォロー機能の実装手順について（コードリーディング）](06_issue_note_follow-unfollow.md)  
