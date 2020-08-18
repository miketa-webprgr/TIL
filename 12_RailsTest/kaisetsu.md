# Railsテストに対する解説

## 問題の掲載先

以下を参照すること。  
なお、TechEssentialsにログインする必要がある。  

https://tech-essentials.work/courses/7/take_quiz/new

なお、問題の作成にあたっては、Railsガイドを参考にしたとのことなので、  
基本的にはそちらを参照すれば詳しい情報を得ることができる。  

## 問題1

ActiveRecordについて正しい記述はどれか問うもの。  

### 問題1: 解説ActiveRecordの機能

Railsガイドに書いてある。  
ORM(Object-Relational Mapping)というものらしい。  

難しいことは分からないが、RubyとSQLをつなぐための仕組みと捉えればよいと思われる。  
RailsではActiveRecordだが、他のプログラミング言語にも似たようなものがあるらしい。  

ActiveRecordの機能は、以下に書いてあるとおり。  
詳しくは、Railsガイドを参照しよう。  

- モデルおよびモデル内のデータを表現する
- モデル同士の関連付け(アソシエーション)を表現する
- 関連付けられているモデル間の継承階層を表現する
- **データをデータベースで永続化する前にバリデーション(検証)を行なう**
- **データベースをオブジェクト指向スタイルで操作する**

- [ORMフレームワークとしてのActive Record](https://railsguides.jp/active_record_basics.html#orm%E3%83%95%E3%83%AC%E3%83%BC%E3%83%A0%E3%83%AF%E3%83%BC%E3%82%AF%E3%81%A8%E3%81%97%E3%81%A6%E3%81%AEactive-record)  

### 問題1: RailsのモデルはActiveRecordを継承している

開発しているRailsのアプリを開き、適当なモデルのファイルを開いてみよう。  
こうなっているはず。継承してい流ことが分かる。  

```rb
class Product < ApplicationRecord
end
```

### 問題1: Railsにおける命名ルール

- モデルのクラス - 単数形、語頭を大文字にする (例: BookClub)
- データベースのテーブル - 複数形、語はアンダースコアで区切られる (例: book_clubs)

面白いのは、Mouseクラスを作成した場合、きちんとデータベースはmiceにしなければならないところ。  
こういった不規則なものについても対応している。  

- [命名ルール](https://railsguides.jp/active_record_basics.html#%E5%91%BD%E5%90%8D%E3%83%AB%E3%83%BC%E3%83%AB)  

## 問題2

モデルについて正しい記述はどれか問うもの。  
基本的には、問題１で解説した内容でカバーしているため、そちらを参照するとよい。  

### 問題2: ケバブケース・キャメルケース・スネークケース

| 名称         | 用例       | 主な言語                |                名称の由来                |
| ----------- | ---------- | --------------------- | --------------------------------------- |
| キャメルケース | camelCase  | Java, JavaScript      |   ラクダのコブみたいだから（大文字のがコブ）   |
| スネークケース | snake_case | PHP, Python, Ruby     |   ヘビみたいに下を這っているから             |
| ケバブケース  | kebab-case | HTML, CSS, Lisp       |    ケバブみたいに串刺しだから（焼き鳥的な）    |

気合で覚えると大変。以下のように覚えよう。  
多分、RubyやRailsでケバブケースって使わない。  

モデルはキャメルケースで単数系。  
例えば、`QuizProblem`のようにすること。  

テーブルはスネークケースで複数形。  
例えば、`quiz_problems`のようにすること。  

## 問題3

基本的には、問題１や問題２で解説したとおり。  
以下、補足を記載する。  

### データベースのテーブル名の探索方法

Railsでデータベースのテーブル名を探索する場合、どうなっているか。  
例えば、usersテーブルのデータを全て取得したい場合、Railsでどう書くか考えてみよう。  

```rb
# すべてのユーザーのコレクションを返す
users = User.all
```

このように、Rails上ではモデルのクラス名（単数系）をそのまま書くが、  
裏では、モデルのクラス名を複数形にしたusersテーブルを探している。  

### 外部キーはテーブル名の複数形_idで定義する？

Railsガイドに書いてあるとおり。  
テーブル名の単数系_idで定義する必要がある。  

Railsでテーブル間の紐付けをする場合、「`belongs_to`と`has_many`を書いて終わり」  
というようなことが多いので意識しづらいが、覚えておくようにしよう。  

例えば、usersとpostsというテーブルが紐づいている場合、  
postsテーブルにおいて指定されている外部キーは`user_id`になる。  
間違っても、`users_id`ではない。  

- [2.2 スキーマのルール](https://railsguides.jp/active_record_basics.html#%E3%82%B9%E3%82%AD%E3%83%BC%E3%83%9E%E3%81%AE%E3%83%AB%E3%83%BC%E3%83%AB)  

### オートインクリメント

そもそも聞くことがあまりない言葉かと思う。  
idの自動採番のことを言うらしい。  

主キーは、railsの方で自動的に付与するidになるので、  
**主キーはオートインクリメントされる。**

調べていないが、こうするとidを1000から始めることができるらしい。  

- [auto\_incrementの初期値を設定したい \- Qiita](https://qiita.com/w-tdon/items/fd147120eebf3cc22edd)  

## 問題 4

CRUDに関するメソッドについて問うもの。  
具体的にはcreate, newなどのメソッド。  

### 解説 4

Railsガイドを見れば、全てが書いてある。  

- [5 CRUD: データの読み書き](https://railsguides.jp/active_record_basics.html#crud-%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AE%E8%AA%AD%E3%81%BF%E6%9B%B8%E3%81%8D)  

注意点だけ、以下に記す。  

- createメソッドは、newインスタンスを作り、saveするメソッド。  
- Rubyの話になるが、インスタンスの属性（attributes）を書き換える場合は4番目の選択肢のような書き方はしない。  
  - シンボルである`:name`に文字列である`"David"`を代入している
  - 厳密には違うようだが、ハッシュのようなものなので、本来的？な書き方はこうなる

```rb
user = User.new(:name => "David", :occupation => "Code Artist")
```

## 問題 5

ActiveRecordのメソッドについて問うもの。  

### 解説 5

ActiveRecordについて、Railsガイドに解説が書いてある。  

- [Active Record クエリインターフェイス](https://railsguides.jp/active_record_querying.html)  

また、各メソッドについて調べる場合、Railsドキュメントを活用するとよい。  

- [where \| Railsドキュメント](https://railsdoc.com/page/model_where)  
- [first \| Railsドキュメント](https://railsdoc.com/page/model_first)  
- [find \| Railsドキュメント](https://railsdoc.com/page/find)  

なお、以上の選択肢の内、2つ目の選択肢以外の全ての選択肢がエラーになる。  

各メソッドのはRailsドキュメントのとおりだが、誤っている部分について以下に記す。  

- 第１の選択肢
  - users.nameの時点でエラーとなる（なお、取得したuser数が1つの場合に限りエラーとならない）
  - users.pluck(:name)とすればエラーとならない
- 第２の選択肢
  - 最初のユーザーを取得できる
- 第３の選択肢
  - findメソッドはidしか引数として受け付けず、id以外のものである場合はエラーとなる。
- 第４の選択肢
  - イコールではなく、シンボルで書かないとエラーになる。
  - なお、`User.where(name: 'David').where(occupation: 'Code Artist')`と書くこともできる

各メソッドの概要については、以下にまとめておく。  

### whereメソッド

- 条件を指定し、当てはまるレコードを全て取得するメソッド
- 条件については文字列で指定したり、ハッシュで指定することができる
- その他、配列で指定することができるので、ドキュメントで確認してみるとよい

### firstメソッド

- モデルの先頭のレコードを取得するメソッド
- 先頭のレコードを３つ取得することや、並び替えてから先頭のレコードを取得することもできる

### findメソッド

- IDを指定してレコードを取得
- 複数してしたり、配列で指定して取得することもできる

## 問題 6

問題5の類似問題。  
update関連のメソッドについて問うもの。  

### 解説 6

Railsガイドに書いてある。  

- [5 CRUD: データの読み書き(Update)](https://railsguides.jp/active_record_basics.html#crud-%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AE%E8%AA%AD%E3%81%BF%E6%9B%B8%E3%81%8D)  

また、各メソッドについて調べる場合、Railsドキュメントを活用するとよい。  

- [find\_by \| Railsドキュメント](https://railsdoc.com/page/find_by)  
- [find \| Railsドキュメント](https://railsdoc.com/page/find)  

なお、以上の選択肢の内、３つ目の選択肢がエラーになる。  

詳細はRailsドキュメントのとおりだが、findメソッドはidしか引数として受け付けず、  
id以外のものである場合は例外処理となる。（例外処理にしたくない場合、find_byを使うとよい）  
（解説5でも説明したとおりである）  

## 問題 7

問題5の類似問題。  
destroy系のメソッドについて問うもの。  

### 解説 7

Railsガイドに書いてある。  

- [5 CRUD: データの読み書き(Delete)](https://railsguides.jp/active_record_basics.html#delete)  

また、各メソッドについて調べる場合、Railsドキュメントを活用するとよい。  

- [destroy \| Railsドキュメント](https://railsdoc.com/page/model_destroy)  
- [destroy\_all \| Railsドキュメント](https://railsdoc.com/page/model_destroy_all)  
- [delete \| Railsドキュメント](https://railsdoc.com/page/model_delete)
- [delete\_all \| Railsドキュメント](https://railsdoc.com/page/model_delete_all)  

ただ、Railsドキュメントだと情報量が少ない部分があるので、英語になるが、RailsAPIを参照する方がよいかもしれない。  
（例えば、destroyメソッドなど）

なお、destroyとdeleteの違いは以下のとおりである。  

- destroy系はモデルを通すため、depedentしているモデルのデータについても併せて削除する  
- delete系はモデルを通さず、直接データベース上のデータを削除するため、dependentの影響を受けない  

また、それぞれのメソッドは、以下のとおりとなっている。  
なお、以下ではdestroyについて書いたが、deleteの場合も同様である。  

- destroy_allは、データを全て削除する場合に使う。
- destroy_byは、条件に合致するデータを全て削除する場合に使う。  
- destroyは、データを一つ削除する場合に使う
  - `User.first.destroy`といった書き方が一般的
  - `User.destroy(1)`といった書き方も可能である
  - `User.destroy([5,6])`とすれば、複数のデータを削除することもできる。  

よって、今回の場合、以下のとおりとなる。

- `User.all.destroy`
  - User.allのデータは一つではないのでエラーとなる
- `User.destroy.all`
  - クラスに対してdestroyする場合、引数がないのでエラーとなる。その後に`all`と書くのもおかしい。
- `User.destroy_all`
  - 正しい
- `User.delete!`
  - クラスに対するdeleteも、destroyと同様に引数にidがないとエラーとなる。

## 問題 8

validationに関する問題。  

### 解説 8

Railsガイドに書いてある。  

- [6 バリデーション（検証）- Railsガイド](https://railsguides.jp/active_record_basics.html#%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%EF%BC%88%E6%A4%9C%E8%A8%BC%EF%BC%89)  

> save、updateメソッドは、バリデーションに失敗するとfalseを返します。  
> このとき実際のデータベース操作は行われません。  
>
> 上のメソッドにはそれぞれ破壊的なバージョン (save!、update!) があり、  
> こちらは検証に失敗した場合にさらに厳しい対応、つまりActiveRecord::RecordInvalid例外を発生します。  

最初の２つの選択肢については、validationに抵触するため、  
非破壊的なメソッドの場合はfalseとなり、破壊的メソッドの場合は例外処理となるはずである。  
だが、いずれのメソッドも逆の結果を示しているので、挙動として正しいものではない。  

最後の２つの選択肢については、問題なくsaveやcreateができるケースであり、挙動として正しいものである。  

## 問題 9

マイグレーションファイルに関する問題。  
マイグレーションコマンドについて問うものとなっている。  

### 解説 9

Railsガイドにおいて該当する部分は以下のとおりである。  

- [Active Record マイグレーション \- Railsガイド](https://railsguides.jp/active_record_migrations.html)  

ここで書いてある概要は分かりづらい。（基本的な部分は分かっている前提で書かれている）  
そこで、自分なりに分かりやすい解説を書いてみる。  

なお、この問題に関する情報としては、Railsガイドの先ほどの解説の中から、  
「5 既存のマイグレーションを変更する」を参照するとよい。  

- [5 既存のマイグレーションを変更する --- Active Record マイグレーション \- Railsガイド](https://railsguides.jp/active_record_migrations.html#%E6%97%A2%E5%AD%98%E3%81%AE%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E5%A4%89%E6%9B%B4%E3%81%99%E3%82%8B)

### そもそもデータベースとは

そもそも、データベースとRailsというのは全くの別物である。  
Railsの中にデータベースが保存されていると考えたりすると混乱の元となるので、ここをまず抑える必要がある。  
（sqliteについては、アプリケーションに組み込んで利用されるデータベースであるため、Rails内にデータベースがあると考えてもよいかも）  

データベースを管理するソフトウェアとして代表的なものは、MySQLやPostgreSQLなどがある。  
（また、変わり種としてRedisというものなどもあるが、ここではその概要に踏み込まない）  

これらを利用する場合、Railsとはまた別のサーバーを立ち上げる必要があり、  
そちらのサーバーにおいて、データベースが保存されている。  

### そもそもマイグレーションファイルとは

では、マイグレーションファイルとは何なのか。  

詳しくは以下を参照してもらいたいが、マイグレーションファイルとはデータベースの操作について命令を書いたコードであり、  
このマイグレーションファイルを実行することで、Railsを介する形で、データベースを作成したり、カラムを追加したり、  
保存できる情報について制約を加えたりすることができる。  

- [Rails初心者がつまずきやすい「マイグレーション」](https://www.transnet.ne.jp/2015/12/29/rails%E5%88%9D%E5%BF%83%E8%80%85%E3%81%8C%E3%81%A4%E3%81%BE%E3%81%9A%E3%81%8D%E3%82%84%E3%81%99%E3%81%84%E3%80%8C%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%80%8Dcolnr/)
- [DIVE INTO CODE \| Railsにおけるマイグレーションとは](https://diveintocode.jp/blogs/Technology/migration#:~:text=%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%A8%E3%81%AF%E3%80%81Ruby,%E3%81%AE%E3%81%93%E3%81%A8%E3%82%92%E6%8C%87%E3%81%97%E3%81%BE%E3%81%99%E3%80%82)
- [【Rails】マイグレーションファイルを徹底解説！ \| Pikawaka \- ピカ1わかりやすいプログラミング用語サイト](https://pikawaka.com/rails/migration)

アプリケーションを開発する場合、まず扱うデータベースの「型」を決める必要がある。  
例えば、usersテーブルを作成し、usernameやemailアドレスというカラムを用意する必要があるだろう。  

こうした操作は、マイグレーションファイルの実行によって、Railsから操作することができる。  

もちろん、MySQLなどのデータベースの管理ソフトを開き、そちらのソフトの方から直接操作を行うことも可能である。  
ただ、わざわざMySQLを開き、Railsで書いているコードと整合性を取るのは面倒であるし、エラーを生む原因の温床となってしまう。  

また、マイグレーションファイルという形で命令の履歴を残すことができるので、チーム開発を行う上で色々と便利になる。  

マイグレーションファイルの具体的な操作の方法については、pikawakaやQiitaの記事などに分かりやすい記事が掲載されている。  
以下では参考までに、pikawakaの記事を掲載する。  

- [【Rails】マイグレーションファイルを徹底解説！ \| Pikawaka \- ピカ1わかりやすいプログラミング用語サイト](https://pikawaka.com/rails/migration)

もちろん、Railsガイドを参照すると詳しく書いてあるので、そちらも併せて参照するとよい。  

### さて、問題を解いてみる

順を追って、各選択肢に誤りがないか検討していく。  

最初の選択肢であるが、マイグレーションファイルに誤りがあった場合、まずrollbackしなければいけない。  
そのまま一部修正し再度db:migrateを行なっても、修正は反映されないので、誤りである。  

このことは、Railsガイドのこの部分においてはっきりと書いてある。  

- [5 既存のマイグレーションを変更する --- Active Record マイグレーション \- Railsガイド](https://railsguides.jp/active_record_migrations.html#%E6%97%A2%E5%AD%98%E3%81%AE%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E5%A4%89%E6%9B%B4%E3%81%99%E3%82%8B)

次に２つ目の選択肢であるが、マイグレーションファイルを再度実行するコマンドはdb:migrate:resetである。  
よって、誤りであり、３つ目の選択肢が誤っていること、4つ目の選択肢が正しいことも同時に分かる。  

なお、db: resetのコマンドは、マイグレーションコマンドを再度実行するものでなく、  
schema.rbを活用して再構築するものなので、注意すること。  

- [4.3 データベースをリセットする --- Active Record マイグレーション \- Railsガイド](https://railsguides.jp/active_record_migrations.html#%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E3%82%92%E3%83%AA%E3%82%BB%E3%83%83%E3%83%88%E3%81%99%E3%82%8B)
- [rails db:migrate:resetできなかったのでrails db:resetした \- Qiita](https://qiita.com/mom0tomo/items/a252ff8a42eea00f81b1)

## 問題 10

以下のマイグレーションファイルについての説明で正しいものはどれか尋ねる問題。  
具体的なマイグレーションファイルのコードがあるので、その中身の理解度を問うている。  

### 解説 10

Railsガイドにおいて該当する部分は以下のとおりである。  
問題9と同じ範囲を扱っているが、今回は具体的なマイグレーションファイルの内容について尋ねられている。  

- [Active Record マイグレーション \- Railsガイド](https://railsguides.jp/active_record_migrations.html)  

また、今回のケースにおいては、pikawakaのサイトが初学者向けに分かりやすく説明しているため、参考にするとよい。  

- [【Rails】マイグレーションファイルを徹底解説！ \| Pikawaka \- ピカ1わかりやすいプログラミング用語サイト](https://pikawaka.com/rails/migration)

### マイグレーションファイルの自動作成

RailsガイドやPikawakaに掲載されている。  

- [2.1 単独のマイグレーションを作成する --- Active Record マイグレーション \- Railsガイド](https://railsguides.jp/active_record_migrations.html#%E5%8D%98%E7%8B%AC%E3%81%AE%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B)
- [2.2 モデルを生成する --- Active Record マイグレーション \- Railsガイド](https://railsguides.jp/active_record_migrations.html#%E3%83%A2%E3%83%87%E3%83%AB%E3%82%92%E7%94%9F%E6%88%90%E3%81%99%E3%82%8B)
- [マイグレーションファイルの作成方法 --- 【Rails】マイグレーションファイルを徹底解説！ \| Pikawaka](https://pikawaka.com/rails/migration#%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AE%E4%BD%9C%E6%88%90%E6%96%B9%E6%B3%95)

以上に掲載のとおり、`rails g model`を使うとmodelと併せてmigrationファイルを作成できる。  
また、`rails g migration`を使うとmigrationファイルのみを作成できる。  

### t.referencesについて

マイグレーションファイルにおいては、`t.string`といった形式にて、カラムの型を指定している。  
このカラムの型には様々な種類があり、使っているデータベース管理ソフトの種類によって、扱えるデータ型の数も異なる。  
（例えば、sqliteは使えるデータ型の種類にかなり限りがあるとのこと）  

この中で特徴的なのが、`t.references`というものだ。  

一見すると、`t.references :user, foreign_key: true`と書いてあると、  
references型のカラムにuserという名前を付けようとしているように思えるが、  
実際のところ、カラム名はuserではなく、user_idとなる。  

referencesは、多くの場合、外部キーを作成する時に使われる。  
（indexを貼るためやpolymorphicとするために使われることもあるもよう）
この型のカラムがあることで、テーブルのjoinが可能になる。  

なお、外部キーとして指定するためには、`foreign_key: true`とする必要がある。  

### add_indexについて

以下に記載のとおり、add_indexとは、indexを貼るためのものであり、  
引数としては、まずtable名、次にカラム名、最後に`unique: true`などのオプションがつく。  

```rb
add_index(table_name, column_name, options = {})
```

- [ActiveRecord::ConnectionAdapters::SchemaStatements](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_index)

### 各選択肢の検討

- userというカラムが作られ訳ではなく、user_idというカラムが作られ、外部キーとして指定される。
- `add_index :products, :sku, unique: true`により、skuは一意でなければならない。
- `rails g model`を使うとmodelと併せてmigrationファイルを作成できる。  
- `t.references :user, foreign_key: true`により、user_idには外部キー制約がついている。

以上のことから、１番目の選択肢を除き、全ての選択肢が正しいと判断できる。  

## 問題11

以下のマイグレーションファイルを生成するために打つべきコマンドはどれか問う問題。  
ターミナルに具体的に何を入力するか問うている。  

### 解説 11

- [2.1 単独のマイグレーションを作成する --- Active Record マイグレーション \- Railsガイド](https://railsguides.jp/active_record_migrations.html#%E5%8D%98%E7%8B%AC%E3%81%AE%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B)
- [マイグレーションファイルの作成方法 --- 【Rails】マイグレーションファイルを徹底解説！ \| Pikawaka](https://pikawaka.com/rails/migration#%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AE%E4%BD%9C%E6%88%90%E6%96%B9%E6%B3%95)

以上のサイトなどに解説があるが、とにかく覚えているか覚えていないかを問う問題。  
今回の場合、最初の二つの選択肢が正しく、「カラム名:データ型」となることを覚えること。  

## 問題12

すでにmasterブランチにマージされて本番リリースがされているマイグレーションファイルに誤りを見つけました。  
どう対処するのが良さそう？（マイグレーションファイルをどうすればよいか問う問題）  

### 解説 12

gitの知識も必要とする問題が、基本的にはマイグレーションファイルに関係する知識を引き続き問う問題。  
以下を改めて確認するとよい。  

- [5 既存のマイグレーションを変更する --- Active Record マイグレーション \- Railsガイド](https://railsguides.jp/active_record_migrations.html#%E6%97%A2%E5%AD%98%E3%81%AE%E3%83%9E%E3%82%A4%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E5%A4%89%E6%9B%B4%E3%81%99%E3%82%8B)

> 既存のマイグレーションを直接修正するのではなく、そのためのマイグレーションを新たに作成して  
> それを実行するのが正しい方法です。これまでコミットされてない (より一般的に言えば、これまで  
> development環境以外に展開されたことのない) マイグレーションを新たに生成し、それを編集する  
> のが害の少ない方法であると言えます。

マイグレーションファイルはデータベースの構造の操作に関する履歴なので、その履歴を書き換えてしまえば、  
その履歴間のつながりが追えなくなり、障害の温床を産んでしまう。  

直接編集するのはもちろんのこと、削除をするのはもってのほかである。  

以上のことから、最後の選択肢が対処法としてよさそうであり、他の方法はあまりよろしくないことが分かる。  
