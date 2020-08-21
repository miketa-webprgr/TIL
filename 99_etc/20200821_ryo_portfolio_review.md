# Ryoさんポートフォリオレビューのメモ

みけたが調子に乗って、色々とメモを書いてみました。  
怖いって言われそうですが、できれば頑張って作ったので、Ryoさんと話す時間が欲しい笑

## 全般

- え、これ初めて作ったアプリですか？
  - 以前に結構勉強していたんですか？
  - todosやcategoriesといったテーブルを作って紐付けしているのはすごい
    - 自分が最初作ったアプリではそもそもテーブルの紐付けをしなかった
  - editor_user, viewer_user, 未登録ユーザーを想定して設計したのもすごい
    - ここまで自分にはできなかった

- だいそんさんのチェックポイント
  - schema.rb, routes.rb, Gemfile
  - あと、master.keyがないか、DBサーバーのパスワードが直書きじゃないかなど
  - 書き方のお作法が悪いと印象が悪いと言われる
    - インデントや改行がずれているところは指摘されるかも
    - scssやhelperのファイルは削除して方がよいと指摘されるかも
    - これは好みの問題だけれど、rails使いはslimも２スペースでインデントする方が一般的？

- namespaceで区切ると、分かりやすくなりそう
  - だいそんさんの解説に期待！

## データベース関係

- DBの関係について
  - 文字数制限がきちんとあるのは素晴らしい！
  - マイグレーションファイルで試行錯誤の後が見れたので、頑張った跡が分かりました笑
    - 単純にカラムを追加した後に、そのカラムを削除してindexを貼るマイグレーションファイルを作っていた

- データベースの構造は以下のとおり
  - usersテーブルがeditor_idを持っている
    - editor_idは管理者権限的なもの？
  - categoriesテーブルがuser_idを外部キーとして持っている
  - todosテーブルがcategory_idとuser_idを外部キーとして持っている
  
- データベースに関係する指摘
  - 外部キーとして指定する場合、`foreign_key: true`を追加した方がよい（ケアレス？）
    - これがないと、宙に浮いたtodos（userに紐づかないもの）がデータベースに残ってしまう可能性が出てきてしまう
    - [カラム追加時（Qiita記事）](https://qiita.com/publichtml/items/1fba15d8071fab66d043)
    - ちなみに、modelでは`dependent: :destroy`のオプションがあった！
  - category_idを元にしてtodosを検索する機能を実装する場合にはindexを貼った方がよい
    - ただし、今回はそのような機能を実装していない
  - 単純な漏れだと思うが、todosのdescriptionの文字数制約が漏れている
    - modelでの制約も漏れているようでした！
    - ただ、text型の場合、65536bytesがpostgresql上の制約のようなので大した問題ではないかも
      - あえて制約をつけていないのかも

## コントローラ関係

- コントローラ全般に言える指摘
  - updateメソッドは破壊的メソッドでない方がよいかも
    - validationエラーの場合、アプリが停止してしまう
    - [![Image from Gyazo](https://i.gyazo.com/6d1507b6338340c8dfbff9c260a6fb64.gif)](https://gyazo.com/6d1507b6338340c8dfbff9c260a6fb64)
  - destroyメソッドは破壊的メソッドにするとよいかも
    - 失敗が想定しえないものを破壊的メソッドにする

- application_controller.rbに関する指摘
  - 個人的に読みづらかったが、みけたが悪いのか聞いてみたい
    - `@editor_user ||= @current_user unless @current_user&.editor_id`
      - editor_idがなければ、@editor_userに@current_userを代入する
      - 「外部キーとしてeditor_idを持っていない@current_userは、editor_userである」と理解するのに時間がかかった
        - これぐらい読めるようになるべきか、それともコメントを入れておくべきか
        - そもそもryoさんからすれば、他人が読む前提で今回はアプリを作っていないだろうけど笑
  - `before_action :editor_required`して、必要なところだけskipするのは良い設計なのか
    - 漏れがなくなりそうだけど、若干わかりにくい気もする
    - この問題はだいそんさんにも聞いてみたい（そもそもnamespaceで区切るって話になる？）

- todos_controller.rbに関する指摘
  - リファクタの余地がありそう
    - set_todoメソッドを書き直して、もっと有効活用するのはどうか（みけたの方で具体的にコードを書いてみたい）
  - createアクションについて失敗した場合、flashでエラーメッセージも出すのが一般的かも
    - ただし、バリデーションエラーが出るので、これは好みの問題

## だいそんさんからの指摘

- DB上の制約は変更が大変
  - そのことを頭に入れて設計するとよい
  - 今回の例で言えば、モデルのみの制約に留め、DB上に制約をかける必要はないかも
- モデルにロジックを寄せるとよさそう
  - editorか判断するメソッドなどを作る
  - 親か子であるかを関係なくtodosを持ってくるメソッドを作る

```rb
#user.rb
  # editorか？
  def editor?
    editor_id.nil?
  end
  # viewerか？
  def viewer?
    editor_id.present?
  end
```

```rb
#user.rb
def my_todos
  if editor?
    todos
  else
    editor.todos
  end
end
```

```rb
def show
  @todo = current_user.my_todos.find(params[:id])
end  
```
