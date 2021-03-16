# mock_practice

## この課題について

だいそんさん作成。

- mockの課題。題材はGitのリポジトリ取得処理。
- [DaichiSaito/mock\-practice](https://github.com/DaichiSaito/mock-practice

## 環境構築について

私の環境限定ということかもしれないですが、これをしたら動きました。

```
bundle install
yarn install
```

そして、起動したのがこの画面。

<a href="https://gyazo.com/0a0ad12476c885283f1beebc35c6ed46"><img src="https://i.gyazo.com/0a0ad12476c885283f1beebc35c6ed46.png" alt="Image from Gyazo" width="800"/></a>

最初はよく分からなかったですが、要はGitHubのアカウント名を入力したら、
レポジトリが取得できる簡単なアプリのようです。

このレポジトリについて、以下のテストを書いていきます。

> 検索結果ページにリポジトリのidとnameとhtml_urlが表示されているのか

## モックやスタブとは（ざっくり概要編）

テストといえば伊藤淳一さんということで、以下の記事から抜粋しました。

> モックとはざっくりいうと 「本物のふりをするニセモノのプログラム」 のことです。
> 何らかの理由で本物のプログラムが使えない、もしくは使わない方がよいケースでモックが使われます。

> たとえば、外部のAPIを利用しなければならない場合です。
> [使えるRSpec入門・その3「ゼロからわかるモック（mock）を使ったテストの書き方」 \- Qiita](https://qiita.com/jnchito/items/640f17e124ab263a54dd)

## 動画教材を活用して、mockやstubを学ぶ

教材としては以下を学習しました。

英語ですが、個人的にはシンプルな例を出して説明してくれたので、
非常に分かりやすかったです。

伊藤さんのQiita記事を読む前に試聴をしてもよいと思います。
（スペイン語なまりの強い英語ですが）

- [YouTube動画 - How to Use RSpec Mocks & Stubs](https://www.youtube.com/watch?v=oyMPzA-ZWkE&ab_channel=JesusCastello)
- [テキスト版 - How to Use RSpec Mocks](https://www.rubyguides.com/2018/10/rspec-mocks/)

モックとスタブを簡単なコードを使って解説してくれています。

### stubについて

stubについては、以下のようなメソッドが例として挙げられていました。

```rb
class RandomNumberGenerator
  def random
    'A'*rand(1..10)
  end
end
```

このメソッドの場合、'A'と返されることもあれば、'AAAAA'と返されることもあります。
その場合、期待する返り値の値に揺らぎがあるので、テストを書くのが困難です。

この場合、このように対処していました。

```rb
it "generates a random number" do
  generator = RandomNumberGenarator.new
  allow(generator).to receive(:rand).and_return(5)
  expect(generator.random).to eq('AAAAA')
end
```

ここでは、randメソッドによる返り値を5に固定しています。
これをstubというらしいです。（表層的な理解かも。あくまで動画内の話です）

randメソッドは、偽物に置き換えられてしまったわけですが、
このテストで本当にテストしたいのは、randomメソッドであり、
rubyにそもそも備わっているrandメソッドではないので問題がないわけです。

置き換える場合には、このテストの本質は何なのか見極めることが重要です。

### mockついて（動画教材活用編）

次に、モックの説明がありました。
このようなクラスがあると事例として挙げられていました。

かなり無茶苦茶ですが、ImageFlipperオブジェクトが
`image_flipper.change_image('ruby.jpg')`という感じで実行すると、
何らかの画像処理ライブラリ（例えばMiniMagick）が走り、
flipメソッドにより、画像が反転されるとします。（そう考えてください！）

```rb
class ImageFlipper
  def initialize(image_processor)
    @image_processor = image_processor
  end

  def change_image(file_name)
    @image_processor.flip(file_name)
  end
end
```

このコードに対してテストを書く場合、どうすればよいでしょうか？
まず、愚直に書く場合を考えてみましょう。

```rb
RSpec.describe "ImageFlipper" do
  it "calls the flip method with the correct arguments" do
    # ImageFlipperでmini_magickというライブラリを使う
    img = ImageFlipper.new("mini_magick")
    # change_imageメソッドで、頑張って画像を反転させる。この処理が重い。。。
    # あと、マッチャを何にすればいいかが分からない。。。
    # とりあえず、エラーにならないということを検証してみました
    expect(img.change_image("ruby.jpg")).not_to raise_error
  end
end
```

コメントアウトで書きましたが、このような課題があります。

- 画像処理が終わるのを待つ必要があるため、テスト実行にかなりの時間がかかってしまう。
- テストのために画像を用意するのは大変。
- マッチャが微妙。画像が反転しているかどうかは、分からない。エラーが起きないか確認しているだけ。

じゃあ、ここでは一体何をどこまで確認するのがよいのでしょうか。

もちろん、ベストなのは毎回手動で画像が反転しているか確かめることです。
ですが、毎回そこまでテストをするのは相当の負担です。

エラーが起きないか確認するだけに留めるにしろ、画像を実際に反転していては、テストの実行時間が長くなってしまいます。
（ちなみに、外部APIのテストの場合、叩ける回数に制限がかかっているため、毎回叩くのは現実的ではない）

そこで、妥協をしましょう。
change_imageというメソッドを実行した際に、"mini_magick"のflipメソッドが"ruby.jpg"に対して実行されていればよしとしましょう。

（ちなみに、"mini_magick"を"mini_moni"にしても"ruby.jpg"を"diamond!!!"にしても全く問題ないです)
（実際のところはnewの引数がchange_imageというメソッドを実行した際に、flipメソッドが呼ばれるかという関係性のテストをしています）
（ここではmini_magickというライブラリのことや、ruby.jpgという具体的な画像については一切関与していません）

この際、doubleというメソッドを使って「影武者」を作り、image_processorをモックに置き換えます。
抽象的すぎるので、コードを見てみましょう。

```rb
RSpec.describe "ImageFlipper" do
  it "calls the flip method with the correct arguments" do
    # 影武者mockに置き換えましょう
    mock = double("mini_magick")
    # 影武者mockのflipメソッドがpictureという引数に対して実行されるかテストする
    expect(mock).to receive(:flip).with(picture)

    # 影武者mockを引数とする
    img = ImageFlipper.new(mock)
    # imgのchange_imageメソッドを実行する
    # flipメソッドがpictureに対して呼ばれていれば、テストはパスする
    img.change_image(picture)
  end
end
```

このテストでは、既に括弧書きする形で書きましたが、あくまで関係性のテストをしています。
なので、mini_magickが壊れていてもテストは通りますし、画像じゃないテキストファイルを入れてもテストは通ります。

というのも、ここで確認しているのは、flipメソッドがpictureという引数に対して実行されるかという１点のみだからです。

mockを使う際には、どこをテストの範疇から除外しているのか、どこを除外していいのか意識する必要があります。

今回の場合、mini_magickが壊れていないかどうか、画像が反転しているかどうかまでは、適度に実際に試して確認すれば問題がないので、
このようなmockで問題がないわけです。

テストの範疇を適切に狭めることで、高速にテストをガンガン走らせて、安全にコーディングができるようになるのです！
