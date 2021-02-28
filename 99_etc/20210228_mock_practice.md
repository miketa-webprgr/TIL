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

このコードに対してテストを書く場合、実際にリクエストを投げて、レスポンスを待ってもよいのですが、
画像処理が終わるのを待つ必要があるため、テスト実行にかなりの時間がかかってしまいます。

ただ、このテストで本質的に確認をしたいのは、画像が正しく反転されるかではなく、
実装したメソッドを実行した際に、期待する返り値が返ってくるかということです。

もちろん、ユーザー視点に立てば、返り値云々なんて知ったこっちゃない話ではないので、
どこかできちんと画像が反転されるのか、どこかで検証は行うべきです。

ただ、単体テストを書く場合なんかを想定した場合、ライブラリがきちんと走るかどうかは
コード実装者の責務から外れてくるので、テストの範囲から除外したいわけです。

そこで、image_processorをモックに置き換えて（doubleメソッドを使って「影武者」を作る）、
期待した返り値が取得できるかだけテストします。　　

これなら動作が高速なので、ガンガンテストを走らせることができます。
具体的にはこんな感じになります。

```rb
RSpec.describe "ImageFlipper" do
  it "calls the flip method with the correct arguments" do
    # mini_magickがimage_processorです
    mock = double("mini_magick")

    # mini_magickのflipメソッドがruby.jpgに対して呼び出されるか確認
    # ここではmini_magickが動くかなんて知ったこっちゃない
    # きちんとmini_magickのflipメソッドがruby.jpgという引数に対して実行されるかだけ確認できればOK
    expect(mock).to receive(:flip).with("ruby.jpg")
    img = ImageFlipper.new(mock)
    img.change_image("ruby.jpg")
  end
end
```



