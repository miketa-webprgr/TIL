## 現場Rails Chapter03-01
## 「タスク管理アプリケーションを作ろう」 「アプリケーション作成の準備」

---

<BR><BR>

### アプリケーション作成の準備（概要）
---

アプリケーションの作成にあたっては、事前に以下の作業が必要になる。  

1. アプリの内容を考える
2. アプリの名前を考える
3. アプリのひな形を作成する
4. データベースを作成する

さらに現場Railsでは、以下の作業も併せて行っていく。  

5. Viewを効率的に書くためにSlimを使う
6. 見栄えを整えるためにBootstrapを使う
7. Railsのエラーメッセージを日本語で表示させる

<br>

#### アプリの内容
---

- タスク管理アプリ
  - CRUD機能を備える
  - 具体的には、一覧機能を表示、詳細表示、新規登録、編集、削除などの機能を搭載

<br>

#### データベースの環境ごとの使い分け
---

開発用・テスト用・本番用と使い分けられた方が都合がよいため、  
Railsでは以下の環境が用意されている。  

1. development
2. test
3. production

各環境に分けて、データベースのほか、アプリケーションの動作に係る設定を行うことができる。  
なお、デフォルトで「bin/rails db:create」すると、開発用とテスト用でデータベースが作成される。  

<br>

#### Slimの設定
---

Railsでは、ビューのファイルを作成するにあたってErbという形式がデフォルトで採用されている。  
ただ、開発の現場ではHamlやSlimが採用されることが多い。  

Erb形式はタグで囲む形式であり、HTMLに書き方が非常に似ているが、  
HamlやSlimではツリー構造で下記のように記載される。  

```
# Haml
# タグ =「%」、Rubyのコード =「@」

%html
  %body
    %h1 = @title # ここでrubyを使って動的にタイトルを出力
    %p # ここが文章です
```

```
# Slim
# タグ = 構造で表現、Rubyのコード =「@」

html
  body
    h1 = @title # ここでrubyを使って動的にタイトルを出力
    p # ここが文章です
```

HamlやSlimを使うには、gemをインストールする必要がある。  
Slimを使うにあたっては、以下をインストールする。
1. slim-rails（Slimジェネレーター）
2. html2slim（Erb形式をSlim形式に変換するerb2slimコマンドを提供）

Gemfileに書き込み、bundleを実行。  
その後、既に作成されているerbファイルを変換するためにerb2slimコマンドを早速実行。

```
$ bundle exec erb2slim app/views/layouts/ --delete
```

無事、ファイル形式が「html.slim」の形式に変更されている！！！  

<br>

#### Bootstrapの導入
---

効率的にアプリケーションの見栄えをよくするため、Bootstrapというフロントエンドのフレームワークを導入する。  
先ほどと同様にgemの導入を行い、その後にapplication.scssファイルを作成し、Bootstrapを読み込んでいく。  

```:Gemfile
gem 'bootstrap'
```

```application.scss
# application.cssを削除し、本ファイルを作成する
# application.css及び.scssは、全てのビューファイルに適用されるCSSテンプレートのようなもの

@import "bootstrap";
```

そして、application.html.slimにBootstrapで用意されたスタイルが適用されるようにコードを追記する。

<br>

#### 日本語でエラーメッセージを表示させる
---

Githubの「rails-|18n」レポジトリにある日本語翻訳ファイルを活用する。  
具体的にはymlファイルをダウンロードし、config/initializers/locale.rbに追記する。  

ただ、ここでは「gem 'rails-i18n'」があるという情報も入手していたので、その方法で試してみる。  
その際、下記のQiitaを参考にした。  

[Qiita:[初学者]Railsのi18nによる日本語化対応
](https://qiita.com/shimadama/items/7e5c3d75c9a9f51abdd5)  

> このgemを導入することによって、  
> Railsを日本語で使う場合のデフォルトのロケールファイルを「svenfuchs/rails-i18n」  
> をダウンロードしなくても使えるようになります。  

ありがたい！！  
とはいえ、設定が大変そう。。。

なので、ここは端的に「ymlをダウンロードする」　→　「gemを導入する」というアレンジだけを加えて、  
あとは現場railsに従ってやってみることとした。

うまくいきますように。。。