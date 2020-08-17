# Issue08 プロフィール編集機能の実装

## どんな感じ？

アバターとユーザー名を変更できるよう、編集用画面を作成する。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/11bf3e6404cd8410244acb8bc235efb8.gif" alt="Image from Gyazo" width="500"/></a></a><br>  

アバター選択時（ファイル選択時）にプレビューを表示する。  

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/3b02085c742829845e5694c087b03428.gif" alt="Image from Gyazo" width="500"/></a></a><br>  

更新すると、アバターとユーザー名が変更される。

<a href="https://gyazo.com/0ecf1fca07bdcf7707a312b7f312b53e"><img src="https://i.gyazo.com/fad7b84ab6fcbd7e432ab60b7c31ccc3.gif" alt="Image from Gyazo" width="500"/></a></a><br>  

## 求められている機能実装・実装条件について

既にgifにて表示したとおり、以下を条件として実装する。  

- 編集画面は/mypage/account/editというパスとする
- アバターとユーザー名を変更できるようにする
- アバター選択時（ファイル選択時）にプレビューを表示する
- image_magickを使用して、画像は横幅or縦幅が最大400pxとなるようにリサイズする
  - image_magickとmini_magickの関係性について不明だったが、どうやらmini_magickはimage_magickを使うためのgemであるらしい
  - ざっと見た限り、だいそんさんのコードではmini_magickについて設定するのを忘れているようだったが、難しくはなさそうなので挑戦したい
- 以降の課題でもマイページに諸々追加するのでそれを考慮した設計とする（ルーティングやコントローラやレイアウトファイルなど）

## コードリーディング

以下では、コードリーディングを行っていく。  

## Avatar画像を取り扱うための実装

### データベース上での設定

まず、アバター画像を取り扱えるようにするため、usersテーブルにavatarというカラムを追加する。  

Issue02の復習になるが、保存するのはあくまでファイル名になるので、  
データ型はStringとなる。このファイル名を活用して、サーバーの保存されている画像のリンクをCarrierwaveで生成させる。  

マイグレーションファイルを実行後、データベース上にカラムが追加され、スキーマファイルが更新させる。  

```rb
# usersテーブルにavatarカラムを追加するマイグレーションファイル

class AddAvatarToUsers < ActiveRecord::Migration[5.2]
  def change
    add_column :users, :avatar, :string
  end
end
```

### Carrierwave上での設定

続いて、Carrierwaveを使うため、設定を行なっていく。  
基本的には公式のGitHubページを参照すればよい。  

- [carrierwaveuploader/carrierwave: Classier solution for file uploads for Rails, Sinatra and other Ruby web frameworks](https://github.com/carrierwaveuploader/carrierwave#getting-started)

また、以前に作成したノートがあるので、そちらを改めて参照するとよい。  

- [TIL/02\_issue\_note\_carrierwave\.md at master · miketa\-webprgr/TIL](https://github.com/miketa-webprgr/TIL/blob/master/11_Rails_Intensive_Training/02_issue_note_carrierwave.md)

設定手順は以下のとおりである。  
なお、MiniMagickを使う場合、HomebrewなどでImageMagickをインストールする必要がある。  

1. `rails generate uploader Avatar`を実行する
2. app/uploaders/avatar_uploader.rbが作成される
3. avatar_uploader.rbの設定を行う（基本的には、コメントアウトを戻す・修正する形で行える）
4. Carrierwaveが使えるように、User.rbファイルに追記を行う

```rb
class AvatarUploader < CarrierWave::Uploader::Base
  # Include RMagick or MiniMagick support:
  # include CarrierWave::RMagick
  # リサイズなどを行うにあたって、公式も推奨しているMiniMagickを採用
  # 使う場合はImageMagickをHomebrewなどでインストールする必要がある
  include CarrierWave::MiniMagick

  # Choose what kind of storage to use for this uploader:
  storage :file
  # fogはS3などのクラウドに画像を保存する場合に必要
  # storage :fog

  # 画像の保存先の設定
  # Override the directory where uploaded files will be stored.
  # This is a sensible default for uploaders that are meant to be mounted:
  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  # ファイルがアップロードされていない場合、デフォルトの画像を使用するよう設定できる。
  # Provide a default URL as a default if there hasn't been a file uploaded:
  def default_url
    'profile-placeholder.png'
  end

  # アップロードする過程でファイルをリサイズしてくれる
  # Process files as they are uploaded:
  # process scale: [200, 300]
  # こちらを参考にした
  # [CarrierWave\+MiniMagickで使う、画像リサイズのメソッド \- Qiita](https://qiita.com/wann/items/c6d4c3f17b97bb33936f)
  # 縦と横の幅を最大400pxとする。なお、他にもresize_to_sizeなどがある。
  process resize_to_limit: [400, 400]

  # サムネ用のファイルもアップロードする場合、設定を行う
  # Create different versions of your uploaded files:
  # version :thumb do
  #   process resize_to_fit: [50, 50]
  # end

  # アップロードできるファイルの拡張子を制限する
  # Add a white list of extensions which are allowed to be uploaded.
  # For images you might use something like this:
  def extension_whitelist
    %w(jpg jpeg gif png)
  end

  # Override the filename of the uploaded files:
  # Avoid using model.id or version_name here, see uploader/store.rb for details.
  # def filename
  #   "something.jpg" if original_filename
  # end
end
```

```rb
# User.rb
# 追記部分のみ記載

class User < ApplicationRecord
  mount_uploader :avatar, AvatarUploader
  〜 その他は省略 〜
end
```

## ルーティングの設定を行う

以下のとおり、プロフィール編集用のルーティングを設定している。  
ここでは、namespaceを使っている。  

```rb
namespace :mypage do
  resource :account, only: %i[edit update]
end
```

namespaceについては、以前にだいそんさんから解説があった。  
一般的には、管理者画面などを実装する際に使うことが多い。  

- [TIL/20200628\_dyson\_answers\.md at master · miketa\-webprgr/TIL](https://github.com/miketa-webprgr/TIL/blob/master/99_etc/20200628_dyson_answers.md)
- [動画講義一覧 \| TechEssentials](https://tech-essentials.work/movies)
  - Rails全般の知識・勉強方法等の24分30秒頃を参照する

`namespace :mypage do`とすると、mypageディレクトリ以下にコントローラファイルや  
ビューファイルが保存されるようになる。

|Controller |View  |
|-----|------|
|<a href="https://gyazo.com/56e6696a01895f5d46a8ffe588c29212"><img src="https://i.gyazo.com/56e6696a01895f5d46a8ffe588c29212.png" alt="Image from Gyazo" height="300"/></a>|<a href="https://gyazo.com/1a9824bd4b10558b501c27ff8ffacf88"><img src="https://i.gyazo.com/1a9824bd4b10558b501c27ff8ffacf88.png" alt="Image from Gyazo" height="300"/></a>|  

これにより、ロジックなどを簡単に書くことができるようになる。  

例えば、管理者画面を実装する場合、「管理者であれば〜を表示したい」ということや、  
「管理者であれば〜という機能を実装したい」などということが多いかと思う。  

そのような場合、ディレクトリを分けることなく、if文などを使って対応することもできるが、  
namespaceを使って、ディレクトリを分ける方がスマートである。  

今回も、そのようなケースに該当するので、そのような対応が取られている。  

## ビューファイルの実装

```slim
# mypage.html.slim
# application.html.slimのマイページ版と捉えてよい

doctype html
html
  head
    meta content=("text/html; charset=UTF-8") http-equiv="Content-Type" /
    meta[name="viewport" content="width=device-width, initial-scale=1.0"]
    title マイページ | InstaCloneApp
    = csrf_meta_tags
    = csp_meta_tag
    = stylesheet_link_tag 'mypage', media: 'all'
    = javascript_include_tag 'mypage'
  body
    = render 'shared/header'
    = render 'shared/flash_messages'
    main
      .container
        .row
          .col-md-8.offset-md-2
            .card
              .card-body
                .row
                  .col-md-3
                    = render 'mypage/shared/sidebar'
                  .col-md-9
                    .mypage_content
                      = yield
```

```scss
@import 'bootstrap-material-design/dist/css/bootstrap-material-design.css';

main {
  padding-top: 50px;
}
```
