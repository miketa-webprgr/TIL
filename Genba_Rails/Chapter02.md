## 現場Rails Chapter02 「Railsアプリケーションをのぞいてみよう」
---

<BR><BR>

### rbenvとrvm
---

現場railsを読むとrbenvでバージョン管理をしましょうと指示が出てくる。

そういえば、sinatraで環境構築で試行錯誤していた時に、  
「rvm」という他にもバージョン管理ソフトがあるのを耳にしていた。

違いを調べてみた。

[Qiita： Ruby覚書 - Ruby / RubyGem / rvm / rbenv / Bundler](https://qiita.com/anfangd/items/3f35fb98b0f9e023475c)

> rbenv は RVM より軽量コンパクトな Ruby バージョン切り替えツール。  
RVM は機能が豊富だけど使いこなせていないという人は、乗り換えを検討するとよい。

以下のような文言があったため、脳死でrbenvをとりあえず使っていくことにした。  
また、以下も改めて確認した。bash_profileというのもよく分からなかったので、ざっと調べてみた。

[Qiita：【2018年版】macにrbenvを入れてrubyを管理できるようにしちゃう](https://qiita.com/Alex_mht_code/items/d2db2eba17830e36a5f1)  
[Qiita：「.bash_profile」 と 「.bashrc」 ってなに？](https://qiita.com/Kaze-01/items/ffe4486c06a2334b490f)

rbenv -h とコマンドラインで打つと、親切にコマンドの解説が出てきたので、  
その指示に従って、作業フォルダのrubyのバージョンを「2.5.1」にしてみた。

```
# 該当フォルダに移動後
$ rbenv local 2.5.1
=> 2.5.1
# 該当フォルダにてバージョンを確認
$ ruby -v
=> ruby 2.5.1p57 (2018-03-29 revision 63029)
# 非該当フォルダにてバージョンを確認
$ ruby -v
ruby 2.6.3p62 (2019-04-16 revision 67580) 
```

うまくいった！ 

<br>

### その他の環境構築
---

rubygemsとbundleの設定も終え、railsもインストール終了。
と思いきや、

```
$ rails -v
=> Rails is not currently installed on this system. To get the latest version, simply type:
    $ sudo gem install rails
    You can then rerun your "rails" command.
````

という指示が出てきた。  
sudo = 「管理者権限で」というのは何となく把握しているが、コマンドを打つ前に改めて調べる。  

[Qiita：無限 sudo gem install rails にハマりました](http://tomoprog.hatenablog.com/entry/2017/02/03/015936)

何やら、無限にハマるという恐ろしいことが書いてある。  
とはいえ、似たようなPATHを設定するコマンドは既に打ち込んでいる。。。  

とりあえず、コマンドラインを再起動。  
無事にバージョンが確認できた。PATHの設定が反映されていなかったもよう。  

Node.jsのインストールも無事終え、postgresqlのインストールも終了。  
Postgresqlの起動も確認できた。  

Rails::PG ConnectionBadというエラーに今後遭遇した場合、  
おそらくPostgreSQLの立ち上げ忘れが原因であることが多いらしい。  

エラーに遭遇した場合は、このことを思い出していきたい。

<br>

### Railsを動かしてみる
---

さて、「rails new」コマンドにて、早速 Railsアプリケーションを新規作成する。  

```
$ Rails kentasuedomi$ rails new scaffold_app -d postgresql
```

すごい勢いでファイルの作成が行われていく。。。  
さて、次にデータベースの構築である。  

```
$ bin/rake db:create
```

すると、データベースは一応作成できたが、以下の症状になったので、Qiita の記事に従って対応した。    
[Qiita：rails sした時にalready initialized constant FileUtils::VERSIONとwarningが出てくる](https://qiita.com/zQmjRAb73seN5RM/items/ecb9e19ee8f3e9af6018)  

解決後、 bin/rails s のコマンドを打つと、無事 rails の例の画面が出てきた。 

<img src="https://i.gyazo.com/a48b64c0b91264ac5388185a27a53c41.png" width=40% border=1>  

<br><br>

### Railsを動かしてみる
---

以下のコマンドを入力し、指示に従って「``http://localhost:3000/users``」を確認する。

```
$ bin/rails generate scaffold user name:string address:string age:integer
$ bin/rails db:migrate
$ bin/rails s
```
  
当然とはいえ、ここまでの画面が出てくる。驚きだ。  
CRUD機能までついているらしい。（Create, Read, Update, Delete の４機能）  

<img src="https://i.gyazo.com/a3e8856f30bbffd69b273a18c7e1f36c.png" height=200px border=1>  
<img src="https://i.gyazo.com/9efe6fff3c99f3fccb618e2b211d7f5b.png" height=200px border=1>  

<br>

<img src="https://i.gyazo.com/cbd361ec3ff15a6bb59eba69997653b2.png" height=200px border=1>  
<img src="https://i.gyazo.com/c8d6d49126f9a978dbdd102ae86d3f01.png" height=200px border=1>  

<br>

Rails は、MVCモデルのフレームワークである。  
以下の手順に従って、HTMLが動的に生成されている。  

1. まず Controller が呼ばれる
2. indexアクションにより、インスタンス変数 @users に 新しく User というオブジェクトが代入される。
3. User というオブジェクトは、ApplicationRecordという親クラスが与えられている。
    - おそらく ActiveRecord という gem により）
    - User は、データベース関係のモデルである。
4. @users には配列が格納されており、index.html.erb 内でテーブル関係のコードとして出力される。
5. index.html.erbは、レイアウトファイルである application.html.erb に流し込まれる。

また、railsのディレクトリ構造を把握することが重要になるらしく、  
特に以下のディレクトリはどんなrailsアプリケーションでも頻繁に利用するらしい。  

  - app/controllers
  - app/models
  - app/views
  - config
    - アプリケーションの基本的な設定ファイルが保存されている
    - database.yml と routes.rb は特に重要なファイル
  - db
    - テーブル関係のファイル
    - migrationファイル、schema.rb や seeds.rb が保存されている

なお、各ディレクトリについては以下のような解説がある。  
[Qiita：デフォルトのRailsフォルダ構造](https://qiita.com/RMEL/items/d2fc650396be67e52ef1)
