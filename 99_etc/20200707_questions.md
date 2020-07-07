# 質問内容

1. JSON形式である意味について

Carrierwaveで複数の画像を保存する実装を行っている際に生じた疑問です。  
画像を保存するPostモデルを設計している際、`serialize`というメソッドが出てきました。  

```rb:Post.rb
class Post < ApplicationRecord
  mount_uploaders :images, ImageUploader
  serialize :images, JSON
  validates :body, presence: true, length: { maximum: 1000 }
  validates :images, presence: true  
  belongs_to :user
end
```

調べると、serialize「シリアル化」とは複数の並列データを直列化して送信することというのが分かりました。
[【Rails5】carrierwaveとfogでAWS S３に画像を複数アップロードする方法 \#rails \#画像アップロード \- mimemo](https://mimemo.io/m/3A2wRoN6Evo1zM6)  

rails consoleを使うと、imagesの中身が以下のとおりとなっており、  
なるほど、このような形式になっているのか！ということもわかりました。

images: ["161-350x350.jpg", "384-350x350.jpg"]

また、先ほどの記事を読むことで、そもそもJSONで受け取れるように`posts_controller.rb`内で  
プライベートメソッドを以下のように定義していることに改めて気付きました。  

```rb:posts_controller.rb
def post_params
  params.require(:post).permit(:body, images: [])
end
```  

そこで、どうでもよい話なのかもしれないですが、あえてJSON形式でなくてもできるはずだと考えて、  
Array形式？（というんですかね？一般的なRubyで使える配列）でもできないか試してみました。  

試してみると、このようにしたら書き換えても動くことが確認できました。

```rb:Post.rb
class Post < ApplicationRecord
  mount_uploaders :images, ImageUploader
  serialize :images #=> JSONを削除、Arrayと指定しなくとも自動で配列になる
  validates :body, presence: true, length: { maximum: 1000 }
  validates :images, presence: true  
  belongs_to :user
end
```

```rb:posts_controller.rb
def post_params
  params.require(:post).permit(:body, :images #=> images: []から変更)
end
```

おそらく、どちらでも構わないのでしょうけど、JSON形式の方が何かと便利なのでしょうか。
これは、WebAPIを扱いだすと分かってくることなんですかね。。。
