# 20200614 ポートフォリオレビュー会

## レビュー対象

みけた
部費管理マネジャー
https://buhimanager.herokuapp.com/

## ポートフォリオ概要

サッカー部員が会計係に部費の申請ができるアプリ

### アプリの流れ

- サッカー部員がサッカーボールをポケットマネーで購入
- 部で使うボールなので、部費で立て替えて欲しい → 精算申請
- CRUD機能は搭載しているので、申請を修正・削除することができる  
 （ユーザー管理機能は作ってないので、誰でも申請可能・誰でも修正可能）  
 （5000円で申請したにもかかわらず、イタズラで500円にされる危険性あり）  
- 会計係は管理者画面でログイン
- 会計係が返金を終えた後、精算済とする
- 未精算だった申請が精算済扱いとなり、会計係以外は修正できなくなる  
 （イタズラがあっても、お金のやり取りの前に確認がなされるので問題がない・・・ということにしてください）

### 管理者ログイン

- email admin@example.com  
- PASS pasuwado  

### 受けた指摘など

-  Migrationとschema.rbの不一致はバグの温床になる

- フラッシュメッセージとエラーメッセージは別物である  
  - フラッシュメッセージは成功時だけでなく、失敗時も出した方がよいのでは  
  - バリデーションエラーの出力場所についても配慮した方がよいのでは（こだわるのは手間がかかるかも）  

- Strong paramsはきちんと設定しよう（セキュリティ上、危うい）

- 未精算・精算の切り替えでは、クライアントサイドから隠しパラメータを要求するような設定はやめるべき  
  - 不要なのであれば、サーバーサイドだけで処理を完結すべき

- パーシャルではインスタンス変数は使わず、ローカル変数を使うべき  
  - controller → view → partial という順で降りてくるため、controllerから受け渡されていることを認識しづらい  

- ModelでのScopeの活用について  
  - モデル側でカスタムのクエリ用メソッドを定義することができる  
  - 現場Railsのp178に記載あり  
  - [Railsのモデルのscopeを理解しよう \- Qiita](https://qiita.com/ozin/items/24d1b220a002004a6351)  

- destroyアクションは、失敗するケースがありえるのか  
  - 派生して、destroy!の方がよいのではないかという議論が盛り上がった  
    - 今回のように失敗が想定されないケースの場合、destroyが失敗した場合は例外処理（エラー画面の表示）とした方がよいというだいそんさんの意見  
    - 議論に追いつけず理解が及ばなかったが、ユーザーにエラー画面をあまり出すのはよろしくない（しかも実務上意外とそういったことはよく起こる）ので、  
      害となるような事態が想定しえないのであれば、検証エラーに引っかかった場合はfalseとした方がよいというブシトラさんの意見  
    - 現場Railsではp135-136に記載あり  
    - 伊藤さんがdestroy!について記事を書いていた  
      - [ActiveRecordにおけるdestroyとdestroy\!の違い \- Qiita](https://qiita.com/jnchito/items/3393c5c1a744199e128a)  
  - 今回のケースはおそらく違うが、「！」が最後につくメソッドの多くは破壊的メソッド  
    - 破壊的メソッドの意味がよく分からなかったので調べてみた  
    - 破壊的メソッド：オブジェクト自体を書き換えるメソッド  
      - [Ruby 破壊的メソッド 非破壊的メソッド \- Qiita](https://qiita.com/ktpnobu/items/8c78b7d020a03e18ee98)  

- Enumを使ってもよいかもという指摘があった
  - Enum：モデルの数値カラムに対して文字列による名前定義をマップすることができる
    - ３通りのステータスが想定できる場合、それぞれに数字を割り当てる
    - ただ、数字だと分かりづらいので、Enumを活用すると分かりやすい名前をつけて、その名前で呼び出すことができる
    - [【Rails】Enumってどんな子？使えるの？ \- Qiita](https://qiita.com/ozackiee/items/17b91e26fad58e147f2e)

- roleって言っていたけど・・・何？  
  - 役割を示す際によく使われるカラム名だと思われる  
  - こんな感じで使われることが多いらしい  

    ```
    enum role: { admin: 1, manage: 2, other: 3 } enum role: { admin: 1, manage: 2, other: 3 }
    ```  

- renderのcollectionオブションを使ってリファクタリングするとよい  
  - 参考の記事を発見した。明日、試してみたい。  
  - [rails部分テンプレートrenderのメモ \- Qiita](https://qiita.com/takeru56/items/299850d0f054ce107e21#collection%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3)  

- Admin::BaseControllerを作るとよい
  - 現場Railsのp415に書いてあった  
  - 管理者権限がないとredirectする処理があるが、これを全てのコントローラに書くのは冗長  
    - Admin::BaseControllerを作り、共通コントローラに書くとよい  
  - なお、共通のアクションを追加することは避けた方がよいとのこと  
    - 一部のコントローラだけ外すことができず、バグの温床となる  
    - モジュールのMixーinを使う方がよいとのこと  

### レビューを受けて感じた雑感

- gitのコミットログを真面目に書いたつもりだが、意外とミスリードするような書き方が多かった
- チームで仕事をする前提に立つと、コメントの書き方も考えないと分かりづらい
  - というか、１週間ぐらい放置していると、自分がそもそも何でそう書いたか意外と覚えていない。。。
- 現場Railsを頑張ってやったつもりだが、意外とスルーした場所が多いことに気づかされた
  - 特に、最終章は難しそうだと思い手をつけていないが、為になりそうな内容が多い
  - 今なら理解できそうなので、JSが一段落したら挑戦したい