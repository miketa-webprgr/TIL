## 現場Rails Chapter04-1 ~ Chapter04-4
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

<BR><BR>

### Chapter04-3 「モデルの『検証』を使う」
---

データベースで制約を設けることもあるが、あまり制約を設けるとデータが扱いづらくなる。  
そこで、データベースだけでなく、モデルでバリデーションという制約を設けることがある。  

モデルの検証は、以下のフローに沿って事前に行われる。  
1. 検証を行う
2. エラーがあれば登録・更新などを行わずに差し戻す
3. エラーがなければ登録・更新を行う

saveメソッドは、エラーがあればfalseを返す。  
エラーがなければ、更新をした後にtrueを返す。  

save「!」メソッドは、エラーがあればfalseではなく例外を発生させる。  
エラーが分かりやすいため、予期せぬ失敗を防ぐことができる。

また、以下のようにすることで、ひとまずvalidationをオフにすることもできる。

```rb
task.save(validate: false)
```

<br>

#### 検証の書き方
---

２種類ある。

1. Railsが用意しているヘルパー
2. 自分で書く

Railsのヘルパーには、以下のようなものがある。  

```rb
# データがあるか確認
validates :foo, presence: true

# 数値のデータか、期待した数値か（小数点の有無・マイナスであるかなど）
validates :foo, numericality: true

# 数値が想定した範囲内か
validates :foo, inclusion{ in 0..9}

# 文字数の長さは範囲内か
validates :foo, length {maximum: 30}

# 文字列のフォーマットや構成文字種が想定どおりか（例：半角英数字だけか）
validates :foo, format: { with: /\A[a-zA-Z]+\z/ }

# データが一意であり、他と重複がないか
validates :foo, uniqueness: true

# パスワードが一致しているか
validates :foo, confirmation: true
```

<br>

#### これまでのアプリに検証を追加する
---

task.rbに以下のとおり修正
```rb
class Task < ApplicationRecord
  # nameがnullではないか確認
  validates :name, presence: true
end
```

タイトルが空の状態で入力しようとすると、無事エラーになった。  
ターミナルでも、ロールバックしていることが確認できる。  

<a href="https://gyazo.com/7091355d427c22aa82a45f9091fc8239"><img src="https://i.gyazo.com/7091355d427c22aa82a45f9091fc8239.png" alt="Image from Gyazo" width="550" border=1/></a>  

なお、Railsコンソールにて、現在の動作環境でデータベースを操作した際にどのような挙動になるか確認ができる。  

エラーについては、「task.errors.full_messages」で完成されたエラーメッセージのみを確認できる。  
エラーに関する情報は、端的に「task.errors」で確認できる。  

<br>

#### コントローラとビューで検証エラーに対応する
---

これまでのアプリでは、validationがなかったため、データの保存が成功する前提でいて問題がなかった。  
なので、これから失敗も想定して、その場合にエラー画面が出力されるようコードを書き換える。


##### コントローラの修正
---

```rb
# tasks_controller.rb
# 変更前（該当部分のみ抜粋）
def create
  task = Task.new(task_params)
  task.save!
  redirect_to tasks_url, notice: "タスク「#{task.name}」を登録しました。"
end
```

```rb
# 変更後

def create
  # 変数taskをインスタンス変数化する（@をつける）
  # 検証エラーの際に、入力データを引き継いで使う必要があるため（メソッドをまたぐ）
    # 1. 入力データを引き継ぐと、エラーになってから再度入力する際には、修正するだけで済む
    # 2. 入力データを活用して、エラーを表示できる
  @task = Task.new(task_params)

  # if で分岐させる。そのため、「save!」を「save」に変更する
  if @task.save
    # tasks_urlというパスから、@taskに変更
    # ここについては、コード外で触れる
    redirect_to @task, notice: "タスク「#{@task.name}」を登録しました。"
  else
    # saveに失敗した場合、
    render: new
  end
end
```

なお、redirect_toの部分については、なぜ@taskに変わったのかよく分からなかったため調べてみた。  

[Qiita: redirect_to の引数とかのメモ](https://qiita.com/kanpe777/items/c5154b58c852855deefc#%E3%81%93%E3%82%8C%E3%81%8C%E7%96%91%E5%95%8F%E3%81%A0%E3%81%A3%E3%81%9F)  

つまり、以下のように変換されるらしい。  
むずい。。。  

```rb
# 変更前
redirect_to @task, notice: "タスク「#{task.name}」を登録しました。"
```

```rb
# 変更後
redirect_to task_url(id: @task.id), notice: "タスク「#{task.name}」を登録しました。"
```

当然だが、resourcesを利用していて、Railsの規約に沿っているからこそ、  
@taskが都合よくtask_urlとなるわけで、好き勝手に@todoと気分で命名してはいけない。  
（してもいいけど、Railsの理念に反するし、指定することが増えてくる）  

<br>

##### ビューの修正
---

パーシャルのファイルにエラーメッセージを表示させるためのコードを加える。

```rb
# _form.html.slimに以下を追加

# エラーの有無を確認し、該当がある場合にエラーメッセージを表示
- if task.errors.present?
  ul
  # エラーメッセージを全て格納し、リスト形式で逐一表示
  - task.errors.full_messages.each do |message|
      li= message
```

なお、「erros.~系」には、色々なメソッドがあり、railsguideに詳しい解説がある。  

[Railsguide: バリデーションエラーに対応する](https://railsguides.jp/active_record_validations.html#%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%AB%E5%AF%BE%E5%BF%9C%E3%81%99%E3%82%8B)  

また、現場railsに沿って、文字列長のvalidationも追加する。  

<br>

##### オリジナルの検証コードを書く
---

Railsの検証ヘルパーだけでは事足らない場合、自分で書くことができる。  
書き方は２つある。  

1. 検証を行うメソッドを追加して、そのメソッドを検証用のメソッドとして指定する  
2. 自前のvalidatorを作って利用する  

例えば、タスクの名前属性の値には、カンマが入ってはいけないvalidationを作成したいとする。  

その場合、以下のステップを行う。  
1. 検証用のメソッドをモデルクラスに登録する  
2. 検証用のメソッドを実装する  

```rb
# models/task.rb

# 1. モデルクラスへの登録
validate :validate_name_not_including_comma

private

# 2. メソッドの実装
# errors.addにより、検証エラーをerrorsに格納
# if nameがあるのは、nameがnilの時にエラーに格納されるのを避けるため
def validate_name_not_including_comma
  errors.add(:name, 'にカンマを含めることはできません') if name&.include?(',')
end
```

なお、saveメソッドのように自動で検証を行うメソッドだけでなく、  
指定をしないと検証しないメソッドもあるので注意すること。  

<BR><BR>

### Chapter04-4 「モデルの状態を自動的に制御する -- 『コールバック』 --」
---

コールバックとは、検証・登録・更新・削除などの然るべきタイミングで呼び出せる仕組みのことを指す。  
例えば、以下のような時にコールバックを使うことができる。  

- 削除前に入力の確認をしたい
- nameがnilであった場合、「名無しさん」としたい

<br>

##### コールバックの実装
---

さて、「nameがnilであった場合に『名無しさん』とする」コールバックの実装を行うとする。  
その場合、まずどこでコールバックをするか検討する。  

1. Taskオブジェクトが生成される
2. nameが代入される
3. nameが検証される
4. 登録される

既に備えているバリデーションにより、nilはエラーに格納されてしまうので、  
２と３の間にコールバックし、nilを回避するようにしたい。  

この場合、before_validationを使うとよい。  
そのほかのコールバックは、以下に詳細が説明されている。  

[Qiita: Railsのcallbackについて調べた](https://qiita.com/rtoya/items/29cef3e328299781a328)  

下記のとおり実装する。
```rb
# models/task.rb

class Task < ApplicationRecord
# コールバックメソッドの定義
before_validation :set_nameless_name

private

# コールバックメソッドの実装
# errors.addにより、検証エラーをerrorsに格納
# if nameがあるのは、nameがnilの時にエラーに格納されるのを避けるため
def set_nameless_name
  self.name = '名前なし' if name.blank?
end
```

