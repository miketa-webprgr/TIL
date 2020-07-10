# 質問内容

## 質問リスト

- carrierwaveで複数画像の投稿機能を実装する際に気になったこと
  - JSON形式である意味について
  - `serializeメソッドとは`
  - `images: []`という書き方について
  - 参考：複数画像を投稿した場合のparamsを書き出してみた
- 自己参照とか自己結合について
  - JUNさんとの会話から
  - 勉強方法について

## 注意点

例の如く、また長くなってしまいました。  
質問したいのは、以下の２点です。  

1. Rails特訓コースのcarrierwaveで複数画像実装の際に書いたコードの意味
2. DB関係（自己参照・自己結合）

１番について、私が調べたことをぐだぐだと書いたので、議論していく上で活用できそうな
ものがあれば使ってください。微妙なものはスルーしてください。  

## 1. carrierwaveで複数画像の投稿機能を実装する際に気になったこと

### serializeメソッド

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

> [【Rails5】carrierwaveとfogでAWS S３に画像を複数アップロードする方法 \#rails \#画像アップロード \- mimemo](https://mimemo.io/m/3A2wRoN6Evo1zM6)  

ただ、言葉として分かっても、並列データを直列化というのが具体的にどういうことなのかピンと来ません。  
（並列データとは何を指しているのか、直列化するとどのような形に変化するのか）  

また、英語の情報になりますが、以下のような情報もありました。  
オブジェクトとして保存したい属性があるなら〜と書いてありました。  

> [ActiveRecord::AttributeMethods::Serialization::ClassMethods](https://api.rubyonrails.org/classes/ActiveRecord/AttributeMethods/Serialization/ClassMethods.html)  
>
> "If you have an attribute that needs to be saved to the database as an object, and retrieved as the same object, then specify the name of that attribute using this method and it will be handled automatically. The serialization is done through YAML...."
>
> "Keep in mind that database adapters handle certain serialization tasks for you. For instance: json and jsonb types in PostgreSQL will be converted between JSON object/array syntax and Ruby Hash or Array objects transparently. There is no need to use serialize in this case."

### JSON形式で保存するとは（以下で調べたことは関係ないかも）

serializeのJSONオプションを外してデータを保存した場合、データベース上では以下のとおり保存されていました。  
`id:144`がJSONオプションを外してデータを保存したものになります。（それ以前のデータはJSONオプション付きで保存）  

```mysql
+-----+---------------------------------------------+---------------------------------------+---------+---------------------+---------------------+
| id  | images                                      | body                                  | user_id | created_at          | updated_at          |
+-----+---------------------------------------------+---------------------------------------+---------+---------------------+---------------------+
| 131 | ["52-350x350.jpg","702-350x350.jpg"]        | Juventus is the best team!!!          |      40 | 2020-07-07 17:10:01 | 2020-07-07 17:10:01 |
| 132 | ["430-350x350.jpg","39-350x350.jpg"]        | Juventus is the best team!!!          |      41 | 2020-07-07 17:10:07 | 2020-07-07 17:10:07 |
| 133 | ["130-350x350.jpg","7-350x350.jpg"]         | Manchester United is the best team!!! |      42 | 2020-07-07 17:10:13 | 2020-07-07 17:10:13 |
| 134 | ["978-350x350.jpg","395-350x350.jpg"]       | Inter Milan is the best team!!!       |      43 | 2020-07-07 17:10:18 | 2020-07-07 17:10:18 |
| 135 | ["282-350x350.jpg","329-350x350.jpg"]       | Arsenal is the best team!!!           |      44 | 2020-07-07 17:10:25 | 2020-07-07 17:10:25 |
| 136 | ["608-350x350.jpg","447-350x350.jpg"]       | Benfica is the best team!!!           |      45 | 2020-07-07 17:10:32 | 2020-07-07 17:10:32 |
| 137 | ["499-350x350.jpg","944-350x350.jpg"]       | Schalke 04 is the best team!!!        |      46 | 2020-07-07 17:10:40 | 2020-07-07 17:10:40 |
| 138 | ["606-350x350.jpg","210-350x350.jpg"]       | Real Madrid is the best team!!!       |      47 | 2020-07-07 17:10:45 | 2020-07-07 17:10:45 |
| 139 | ["8-350x350.jpg","222-350x350.jpg"]         | FC Barcelona is the best team!!!      |      48 | 2020-07-07 17:10:51 | 2020-07-07 17:10:51 |
| 140 | ["743-350x350.jpg","964-350x350.jpg"]       | Galatasaray is the best team!!!       |      49 | 2020-07-07 17:10:56 | 2020-07-07 17:10:56 |
| 143 | ["おつ_かれ.png","すまん.png"]                | aaaaaa                                |      40 | 2020-07-10 12:41:26 | 2020-07-10 12:41:26 |
| 144 | ---
- おつ_かれ.png
- すまん.png
                                                    | aaaaaa                                |      40 | 2020-07-10 12:51:50 | 2020-07-10 12:51:50 |
```

また、railsコンソールで見てみると、以下のような違いがあることも分かりました。  

```rb
<Post:0x00007fb0501c31a0 id: 143, images: "[\"おつ_かれ.png\",\"すまん.png\"]", 省略
<Post:0x00007fb0501c3010 id: 144, images: ["おつ_かれ.png", "すまん.png"], 省略
```

この場合なんですが、serializeのJSONオプションを外したままであれば、  
JSONオプションを外して保存した画像に限り表示することができました。  

逆に、JSONオプションを戻した場合、画像を表示することができず、エラーとなってしまいました。  

この場合、`id:144`のデータがJSON形式で保存されているということなのでしょうか。  

### images: [ ] とは

```rb:posts_controller.rb
# posts_controller.rb
def post_params
  params.require(:post).permit(:body, :images #=> images: []から変更)
end
```

ここにある`images: [ ]`ですが、どのような意味なのでしょうか。  
配列形式で受け入れるということなのでしょうか。


### 参考：複数画像を投稿した場合のparamsを書き出してみた

参考まで、画像を複数送った時、paramsがどうなっているか確認してみました。  
頑張って、構造化してみました。  

何か分かることがあるかもと思ってやってみましたが、今回の質問について分かったことは特になかったですが、
画像をHTTPリクエストで送った場合、色々な情報が送られていること・carrierwaveのアップローダが
頑張っていることは分かりました。

クライアントとサーバーサイドでどのようなやり取りが聞けたら面白いなと思ったので、一応送っておきます。

```rb
# paramas

<ActionController::Parameters {
  "utf8"=>"✓",
  "authenticity_token"=>"aXi7QDiVvADlysLy6q9766syY4/ZRqOFA7zobjSo0drxJJQTdhp0De7aPGw1imT9E7kX2n0geao3XGsU2sQ8Lw==",
  "post"=> {
    "images"=>[
      #<ActionDispatch::Http::UploadedFile:0x00007fb85589d888
        @tempfile=#<Tempfile:/var/folders/79/zgjvw1r57z71ktv5q1s5c6j40000gn/T/RackMultipart20200710-24739-1ulvjec.png>,
        @original_filename="すまん.png",
        @content_type="image/png",
        @headers="Content-Disposition: form-data; name=\"post[images][]\"; filename=\"\xE3\x81\x99\xE3\x81\xBE\xE3\x82\x93.png\"\r\nContent-Type: image/png\r\n"
      >,
      #<ActionDispatch::Http::UploadedFile:0x00007fb871af3580
        @tempfile=#<Tempfile:/var/folders/79/zgjvw1r57z71ktv5q1s5c6j40000gn/T/RackMultipart20200710-24739-xt0tqa.png>,
        @original_filename="どん_まい.png",
        @content_type="image/png",
        @headers="Content-Disposition: form-data; name=\"post[images][]\"; filename=\"\xE3\x81\xA8\xE3\x82\x99\xE3\x82\x93_\xE3\x81\xBE\xE3\x81\x84.png\"\r\nContent-Type: image/png\r\n"
      >
    ],
    " body"=>" スタンプ2つ"
  },
  "commit"=>"登録する",
  "controller"=>"posts",
  "action"=>"create"}
permitted: false>
```

なお、current_user.posts.build(post_params)は以下のとおりとなった。

```rb
# current_user.posts.build(post_params)
<Post id: nil, images: nil, body: "aaaaaa", user_id: 40, created_at: nil, updated_at: nil>
```

current_user.posts.imagesがnilなのか不安なので確認をしたら、以下のとおりとなった。  

```rb
# current_user.posts.build(post_params).images

<ImageUploader:0x00007ff073a5cf38
  @model=#<Post id: nil, images: nil, body: "aaaaaa", user_id: 40, created_at: nil, updated_at: nil>,
  @mounted_as=:images,
  @staged=true,
  @file=
    #<CarrierWave::SanitizedFile:0x00007ff0906acdf8
      @file="/Users/miketa/Desktop/Github/RailsIntensiveTraining/InstaClone/public/uploads/tmp/1594351918-15561065205033-0003-2985/おつ_かれ.png",
      @original_filename="おつ_かれ.png",
      @content_type="image/png",
      @content=nil
    >,
  @filename="おつ_かれ.png",
  @cache_id="1594351918-15561065205033-0003-2985",
  @identifier=nil,
  @versions={},
  @original_filename="おつ_かれ.png",
  @cache_storage=#<CarrierWave::Storage::File:0x00007ff073127da0
    @uploader=#<ImageUploader:0x00007ff073a5cf38 ...>,
    @cache_called=nil
  >
>,
<ImageUploader:0x00007ff07420da70
  @model=#<Post id: nil, images: nil, body: "aaaaaa", user_id: 40, created_at: nil, updated_at: nil>,
  @mounted_as=:images,
  @staged=true,
  @file=#<CarrierWave::SanitizedFile:0x00007ff082f73490 
    @file="/Users/miketa/Desktop/Github/RailsIntensiveTraining/InstaClone/public/uploads/tmp/1594351918-772061412216386-0004-3089/すまん.png",
    @original_filename="すまん.png",
    @content_type="image/png",
    @content=nil
  >,
  @filename="すまん.png",
  @cache_id="1594351918-772061412216386-0004-3089",
  @identifier=nil,
  @versions={},
  @original_filename="すまん.png",
  @cache_storage=#<CarrierWave::Storage::File:0x00007ff073035938
    @uploader=#<ImageUploader:0x00007ff07420da70 ...>
    @cache_called=nil
  >
>
```

saveの処理が終わった後、@postは以下のとおりとなった。  

```rb
<Post id: 143, images: ["おつ_かれ.png", "すまん.png"], body: "aaaaaa", user_id: 40, created_at: "2020-07-10 03:41:26", updated_at: "2020-07-10 03:41:26">
```

## 2.DB設計について

JUNさんとの会話の中で話に出たので、自己参照とか自己結合について話を訊きたい。

### 自己参照や自己結合を使う場合（一般ユーザーと管理者（親御）を同じテーブルにする）

- テーブル設計がちょっと分かりづらくなる
- けれども、それぞれのユーザーにおいて同じ画面・メニューを使えるようになる

### 別テーブルにする場合

- 別テーブルにするとテーブル設計は比較的わかりやすくなる
- それぞれのユーザーにおいて同じ画面・メニューが使えなくなる

### 勉強方法について

- 楽々ERDレッスン
- その他の本でおすすめのもの
- 実際のアプリや業務システムなどを想像して、ER図を書くのが練習になるのか、など
