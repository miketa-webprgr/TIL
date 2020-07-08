# CarrierWave

## 概要

- 画像投稿用のテーブルを作り、親となるモデルに紐付ける
- アップローダーを作成し、そのファイルに設定を記述する
- あとは、適切に画面に実装するだけ

設定方法については、Qiita記事を参照すれば問題なくできそう。  
公式のGitHubを紐解いていくと、細かいところも書いてある。  

## Active Storage との違い（適当に書いている）

画像投稿の`gem`であるが、今後は`Active Storage`の方が主流になるかも。（現場Railsなどに記載あり）  
Railsでは、`Active Storage`が標準搭載されている。  

まだよく分かっていないが、`Active Storage`では中間テーブルを使っており、  
クラウドストレージへのファイルアップロードなんかが簡単にできるらしい。  
（ただ、そのことにより、キャッシュが効かないといった話をだいそんがしていた。）  

一方、`CarrierWave`については、ざっと調べた限りでは、中間テーブルを使わず、  
画像投稿用のモデルを作り、そのモデルを紐づける形で実装するらしい。  

## アップローダについて

`CarrierWave`にはアップローダーというものが出てくるが、このアップローダーに設定を  
書き込むことで、保存先・ホワイトリスト・ファイル名などの設定ができる。  

アップローダは以下のとおり作成。  
Imageはあくまでアップローダファイル名を指定しているだけなので、自由に設定してよい。  

```text
rails generate uploader Image
```

今回の課題では以下のように設定されていた。  

```rb
class PostImageUploader < CarrierWave::Uploader::Base
  # Include RMagick or MiniMagick support:
  # 今回はMiniMagickが採用されている
  include CarrierWave::MiniMagick

  # resize_to_fit(): そのサイズに合うようにリサイズ。（大きくなることがある。）
  # resize_to_limit(): そのサイズより大きい画像のときに、そのサイズに合うようにリサイズ。（大きくなることはない。）
  process resize_to_limit: [1000, 1000]

  # Choose what kind of storage to use for this uploader:
  # なお、storage :fogは、本番環境でクラウドストレージを活用する場合に使用する
  storage :file

  # 保存先はここで指定する
  # Override the directory where uploaded files will be stored.
  # This is a sensible default for uploaders that are meant to be mounted:
  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  # ファイルがアップロードされていない場合、デフォルトのアップロード先URLを設定する
  # ・・・何に使うんだろう？
  # Provide a default URL as a default if there hasn't been a file uploaded:
  # def default_url(*args)
  #   # For Rails 3.1+ asset pipeline compatibility:
  #   # ActionController::Base.helpers.asset_path("fallback/" + [version_name, "default.png"].compact.join('_'))
  #
  #   "/images/fallback/" + [version_name, "default.png"].compact.join('_')
  # end

  # これを設定すると、アップロードする過程でファイルをリサイズしてくれるらしい
  # つまり、carrierwaveにおいては、ファイル自体はデフォルトのサイズでアップロードするが、表示する段階においてリサイズしている
  # それだとサイズの大きいファイルがサーバーにアップロードされて困る！っていう人はここで設定しろ、ということだと思う
  # https://github.com/carrierwaveuploader/carrierwave#adding-versions
  # 今回は不要なので、特に設定されていない
  # Process files as they are uploaded:
  # process scale: [200, 300]
  #
  # def scale(width, height)
  #   # do something
  # end


  # 設定すると、普通のファイル＋サムネ用のファイルの2つがアップロードされるらしい
  # Create different versions of your uploaded files:
  # version :thumb do
  #   process resize_to_fit: [50, 50]
  # end

  # ホワイトリスト。拡張子でアップロードできるファイルを制限している
  # モデルでバリデーションをかけるのも忘れないように！
  # Add a white list of extensions which are allowed to be uploaded.
  # For images you might use something like this:
  def extension_whitelist
    %w(jpg jpeg gif png)
  end

  # ファイル名の設定。モデルidを名前に使うなと書いてある。。。
  # 真面目なので、ちゃんとuploader/store.rbを読んでみた！！！
  # https://github.com/carrierwaveuploader/carrierwave/blob/master/lib/carrierwave/uploader/store.rb
  # コードの解読はスルーしたが、DBに保存する時に、record id will be nil と書いてあった

  # Override the filename of the uploaded files:
  # Avoid using model.id or version_name here, see uploader/store.rb for details.
  # def filename
  #   "something.jpg" if original_filename
  # end
end
```

## 複数画像を保存する場合

Postモデルは、以下のとおりとなる。  

```rb:Post.rb
class Post < ApplicationRecord
  mount_uploaders :images, ImageUploader
  # 複数の画像を取り扱う場合、serializeメソッドが必要
  # JSON形式でなくとも、複数の画像を受け取ることは可能
  # ただ、posts_controllerにてJSON形式でデータを受け取るよう指定しているので、整合性を取る必要あり
  serialize :images, JSON
  validates :body, presence: true, length: { maximum: 1000 }
  validates :images, presence: true

  belongs_to :user
end
```

重要な点は、以下のとおり。  

- `mount_uploaders`には、「images」と複数形で書く
- `serialize`メソッドを活用する
- JSONを扱う場合、paramsの受け取り方も注意する（以下のコードを参照）

```rb:posts_controller.rb
# posts_controller.rbから該当部分を抜粋
def post_params
  # images:[]とすることで、JSON形式でparamsを受け取る
  params.require(:post).permit(:body, images: [])
end
```

## 参考

以下の記事が参考になった。  

> - [Github: carrierwaveuploader/carrierwave](https://github.com/carrierwaveuploader/carrierwave)  
> - [Railsでcarrierwaveを使ってAWS S3に画像をアップロードする手順を画像付きで説明する](https://qiita.com/junara/items/1899f23c091bcee3b058)  
> - [画像周りの扱い方 \- Qiita](https://qiita.com/jiggaman0412/items/64cb44592923bde2e8ff)  
> - [Rails 超お手軽な画像アップローダー CarrierWave の使い方 \| Workabroad\.jp](https://workabroad.jp/tech/1118)  
> - [CarrierWave 複数の画像をコード三行で一つのカラムに保存する \- Qiita](https://qiita.com/ggtmtmgg/items/ba5f275c122c83013ea1)  
> - [Carrierwaveで複数の画像をSQLiteのDBに保存する \- Qiita](https://qiita.com/Kanahiro/items/20d50393a916cd41b8d8)
