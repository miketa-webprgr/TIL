## 現場Rails Chapter04
## 「現実の複雑さに対応する」
---

<BR><BR>

### Chapter04-1 「さまざまなマイグレーション操作を使いこなす」
---

マイグレーションは２ステップ  

1. スキーマを変更するマイグレーションファイルの作成
2. マイグレーションファイルを「rails db:migrate」で走らせ、データベースに適応

マイグレーションファイルは、gitファイルのようにバージョンとして扱われており、  
マイグレーションを適用したり、取り消したりすることでバージョン管理ができる。  

特に指定をしない場合、マイグレーションは開発用DBに適用される。  
なお、本番用DBやテスト用DBに適用する場合は、下記のとおり記載。  

```rb
# 本番用
$ bin/rails db:migrate RAILS_ENV=production
```

```rb
# テスト用
$ bin/rails db:migrate RAILS_ENV=test
```

また、マイグレーションの名前の付け方だが、アプリケーション内で一意でなければならない。  
ファイル名が重複してはならず、「AddDeadlinetoTasks」のように具体的に書くのが一般的。  

<br>

#### Schema.rb
---

Railsでは、「schema.rb」に現在のデータベースの構造を出力する。  
先ほどのtaskleafアプリの場合、schema.rbは下記のとおりとなっていた。  

```rb
ActiveRecord::Schema.define(version: 2020_05_06_094134) do

  # These are extensions that must be enabled in order to support this database
  enable_extension "plpgsql"

  create_table "tasks", force: :cascade do |t|
    t.string "name"
    t.text "description"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

end
```

<br>

#### Railsのマイグレーションコマンド
---

以下に詳細な案内がある。

[Qiita: RailsのMigrationに関する基本まとめ](https://qiita.com/pon-san/items/c54aee04e56e6bb26eff)  
[Qiita: rails generate migrationコマンドまとめ](https://qiita.com/zaru/items/cde2c46b6126867a1a64)  

なお、「bin/rails db:migrate」で最新のマイグレーションを実行できるが、  
version を指定してマイグレーションをすることもできる。  

また、マイグレーションを取り消す「bin/rails db:rollback」というものがある。

適宜、データベースに操作を加える場合、操作を確認し、マイグレーションに関するコマンドを入力するようにする。  

マイグレーションの適用にエラーが発生した場合、ロールバックされる。 
その適用は取り消され流ため、コードを修正し、またコマンドの適用を試みる。  

<BR><BR>

### Chapter04-2 「データの内容を制限する」
---

データベースのカラムには型を指定する。  
指定できる型には色々な種類がある。  

- string : 文字列
- text : 長い文字列
- integer : 整数
- float : 浮動小数
- decimal : 精度の高い小数
- datetime : 日時
- timestamp : タイムスタンプ
- time : 時間
- date : 日付
- binary : バイナリデータ
- boolean : 真偽値

NOT NULL 制約というものもある。  
語義のとおり、NULL（空であること）を受け付けない制約である。  

<br>

#### NULL制約を加えてみる  
---

まず、bin/rails g migration ファイル名にて、マイグレーションのひな形ファイルを作成する。

```rb
bin/rails g migration ChangeTasksNameNotNull
```

次にひな形に変更内容を加える。  

```rb
# change_column_nullの行を追記

class ChangeTasksNameNotNull < ActiveRecord::Migration[5.2]
  def change
    change_column_null :tasks, :name, false
  end
end
```

そして、マイグレートコマンドを入力

```rb
bin/rails db:migrate
```

なお、最初から分かっている場合、最初に作ったマイグレーションファイルに追記しても良い。

<br>

#### limitで文字列カラムの長さを制限する
---

limitを使うことで、カラムの文字数を制限できる。

```rb
# t.stringに「limit: 30」を追記

class CreateTasks < ActiveRecord::Migration[5.2]
  def change
    create_table :tasks do |t|
      t.string :name, limit: 30
      t.text :description

      t.timestamps
    end
  end
end
```

テーブル作成後に制限を加える場合、rollbackできるようにdownメソッドを定義する。  

```rb
# 「limit: 30」を追記し、マイグレート用のupとロールバック用のdownのメソッドを作成

class ChangeTasksNameNotNull < ActiveRecord::Migration[5.2]
  def up
    change_column :tasks, :name, :string, limit: 30
  end
  def down
    change_column :tasks, :name, :string
  end
end
```


<br>

#### ユニークインデックスの作成
---

カラムのデータが全レコードで一意である場合、データベースにユニークインデックスを作成することで、  
一意性が壊されるのを防ぐことができる・・・？  

[DBOnline: UNIQUEインデックスを作成する](https://www.dbonline.jp/sqlite/index/index3.html)  

なるほど。

> インデックスの対象となるテーブルのカラムに格納されている値には重複した値が  
> 含まれていても構いませんが、重複した値を許可しないように設定することもできます。  
> このようなインデックスをユニークインデックスと呼びます。  

ユニークインデックスを作成する場合、以下のとおり追記する。

```rb
# ユニークインデックスの作成

class ChangeTasksNameNotNull < ActiveRecord::Migration[5.2]
  def change
    change_column_null :tasks, :name, unique: true
  end
end
```

