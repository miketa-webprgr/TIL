# mock_practice

## この課題について

だいそんさん作成。

- mockの課題。題材はGitのリポジトリ取得処理。
- [DaichiSaito/mock\-practice](https://github.com/DaichiSaito/mock-practice)

みけたのGitHub。

- [miketa\-webprgr/mock\_practice: モックやスタブの練習](https://github.com/miketa-webprgr/mock_practice)

HackMD

- https://hackmd.io/dexaGI29Tc2jh1ZtO9NNRg?both

## 環境構築について

私の環境限定ということかもしれないですが、これをしたら動きました。

```
bundle install
# もしくは bundle install --path vendor/bundle
yarn install
```

そして、起動したのがこの画面。

<a href="https://gyazo.com/0a0ad12476c885283f1beebc35c6ed46"><img src="https://i.gyazo.com/0a0ad12476c885283f1beebc35c6ed46.png" alt="Image from Gyazo" width="800"/></a>

最初はよく分からなかったですが、要はGitHubのアカウント名を入力したら、
レポジトリが取得できる簡単なアプリのようです。(このあたりで実践してみる)

なお、コントローラですが、このようになっています。

```rb
class HomesController < ApplicationController
  def show
    @repos = if params[:username]
               Octokit::Client.new.repositories(params[:username])
             else
               []
             end
  end
end
```

実は、OctokitがこのURLを叩いています。

- https://api.github.com/users/miketa-webprgr/repos

viewはこのようになっています。

```erb
<div style="margin: 0 auto; width: 800px;">

  <%= form_with method: :get do |f| %>
    <%= f.search_field :username, placeholder: 'ユーザー名' %>
    <%= f.submit '検索' %>
  <% end %>

  <table>
    <tr>
      <td>id</td>
      <td>name</td>
      <td>html_url</td>
    </tr>
    <% @repos.each do |repo| %>
      <tr>
        <td><%= repo.id %></td>
        <td><%= repo.name %></td>
        <td><%= link_to repo.html_url, repo.html_url %></td>
      </tr>
    <% end %>
  </table>
</div>
```

## どのようなテストを書くのか

このアプリについて、検索結果ページに以下が表示されるかテストを書いていきます。

- リポジトリのid
- リポジトリのname
- リポジトリのhtml_url（リンク）

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

## stubについて

stubについては、以下のようなメソッドが例として挙げられていました。

```rb
class RandomGenerator
  def random
    'A'*rand(1..10)
  end
end
```

このメソッドの場合、'A'と返されることもあれば、'AAAAA'と返されることもあります。
その場合、期待する返り値の値に揺らぎがあるので、テストを書くのが困難です。

この場合、このように対処していました。

```rb
it "generates 'A' random number of times" do
  generator = RandomNumberGenarator.new
  allow(generator).to receive(:rand).and_return(5)
  expect(generator.random).to eq('AAAAA')
end
```

ここでは、randメソッドによる返り値を5に固定しています。
これをstubというらしいです。（表層的な理解かも。あくまで動画内の話です）

randメソッドは、偽物に置き換えられてしまったわけですが、
このテストで本当にテストしたいのは、以下のはずです。

- 'A'*rand(1..10) が実行されるか
- 返り値として、'AAAAA'のような形式で値が得られるか

以上については、randメソッドによる返り値を5に固定しても問題ありません。

randメソッドについてはテストをすることはできませんが、randメソッドは、
rubyにそもそも備わっているメソッドなので、問題ありません。

**置き換える場合、このテストの本質は何なのか見極めることが重要です。**
返り値を置き換えることで、テストを書く上で障害となるrandメソッド特有の面倒くささを回避できます。

ちなみに、その次にモックの説明がありました。
ただ、必ずしも分かりやすいわけではなかったので、もう本題に移ろうと思います。
（気になる方は、この記事の最後にボツにしたセクションがあるので、よければどうぞ）

## mockをやる前に、実際にAPIを叩いちゃうテストを書いてみる

次に、モックの説明がありました。
が、動画の事例が分かりやすいわけではなかったので、もう本題に移ろうと思います。

もう一度、こちらの画像を確認して、アプリのイメージを思い浮かべましょう。

<a href="https://gyazo.com/0a0ad12476c885283f1beebc35c6ed46"><img src="https://i.gyazo.com/0a0ad12476c885283f1beebc35c6ed46.png" alt="Image from Gyazo" width="800"/></a>

コントローラテストを書く場合、愚直にテストを書くとすると、このような形になるはずです。

（なお、なんでレクエストスペックやシステムスペックじゃないんだという話は後ほどします。コントローラスペックじゃないと、説明する上で色々と面倒なんです）


```rb
require 'rails_helper'

RSpec.describe "homes", type: :controller do
  describe HomesController do
    describe '#GET' do
      it 'miketa-webprgrで検索すると検索結果が返ってくること' do
        get :show, params: { username: 'miketa-webprgr'}
        expect(response).to have_http_status(200)
      end
    end
  end
end
```

これは何をやっているかというと、usernameとして`miketa-webprgr`を入力すると、HTTPのステータスコードとして200が返ってくること（つまり、きちんとレスポンスが返ってくること）を試しています。

この状態で、テストとして成立しています。

ただ、以下のような課題があります。

- サーバーに負荷をかけるべきでない。
- APIを叩ける回数に制限がある。（制限を超えると有料）
- その外部APIを叩けない事情がある
    - tweetしてしまう、ユーザーを作成してしまうなど(post系)
- テストのためだけにアカウントを作るのは手間

そこで、外部APIを叩くことなく、このコードのテストを実行する必要が出てきます。
その時に活躍をするのが、モックというやつです。

モックを使って、`Octokit::Client.new.repositories(params[:username])`の部分を置き換えましょう。そして、外部APIを叩かずして、このrepositoriesメソッドがきちんと呼び出されるかまでを確認しましょう。

その場合、以下については確認することができません。

- repositoriesメソッドの返り値
- 外部APIがレスポンスとして何を返すか

ただ、repositoriesメソッドの返り値については、Octokitのgemの話です。ごく稀にgemが間違っていることもあるので気にした方がいいかもしれないですが、こちらが実装するものではないので、多くの場合は、テストの範囲から除外してもよいでしょう。

また、外部APIがレスポンスとして何を返すかについては、そもそもこちらが実装に関われる話ではないです。信用できないなら、外部APIなど使うべきではありません。もちろん、こちらが外部APIの仕様を誤解している可能性などもあるので、一定の確認は必要だと思いますが、テストの範囲から除外してもよいでしょう。

## mockについて

では、mockを使う場合、どうすればよいのか。
結論、こんな感じです。


```rb
require 'rails_helper'

RSpec.describe "homes", type: :controller do
  let(:miketa_params) { { username: 'miketa-webprgr'} }

  describe HomesController do
    describe '#GET' do
      it 'repositoriesメソッドが、somethingを引数として実行される' do
        mock = double('octokit')
        allow(Octokit::Client).to receive(:new).and_return(mock)
        expect(mock).to receive(:repositories).with('something')

        get :show, params: { username: 'something'}
        expect(response).to have_http_status(200)
      end
    end
  end
end
```

まず、`double`というメソッドを使って、ダミーインスタンスを作ります。

```rb
mock = double('octokit')
```

伊藤淳一さんはモックのことを「影武者」と表現していますが、今回の事例においては`Octokit::Client.new`を置き換える「替え玉」のようなものだと思ってください。


次に、このモックを`Otcokit::Client.new`に置き換えます。

```rb
allow(Octokit::Client).to receive(:new).and_return(mock)
```

ここで、`Otcokit::Client`の`new`メソッドが呼ばれた時に、mockを返すと設定します。


そして、mockの`repositories`メソッドが`something`という引数で実行されるかチェックします。ここがテストの核になります。（ちなみに、`and_return`を書いていない場合、どうやらnilを返すようです。）

```rb
expect(mock).to receive(:repositories).with('something')
```

そして、showアクションに対して、somethingというparamsを添えてアクセスします。

```rb
get :show, params: { username: 'something' }
# 開発環境の場合、以下のようなURLにアクセスすることになる
# http://localhost:3000/?username=something&commit=検索
```

すると、showアクションが実行されるので、showアクションを再度確認しましょう。

```rb
class HomesController < ApplicationController
  def show
    @repos = if params[:username]
               Octokit::Client.new.repositories(params[:username])
             else
               []
             end
  end
end
```

通常どおりであれば、`Octokit::Client.new.repositories(params[:username])`が実行されるはずですが、`Octokit::Client.new`の返り値は替え玉であるモックになっています。

すると、`mock.repositories('something')`となるはずです。また、`and_return`を書かなかったので、この返り値は`nil`となり、`@repos = nil`となるはずです。

このようなプロセスを経れば、`expect(mock).to receive(:repositories).with('something')
`というテストはパスします。

また、コントローラ内でエラーとならなかったので、サーバー側から200コードが返るはずです。

## Systemスペックの罠

ここではもう細かく説明は書きません。読んで察してください。
口頭で説明します。

なお、リクエストスペックも同じところで失敗するので、eachで回した時にエラーにならない何かを用意してあげる必要があります。

```erb
<% @repos.each do |repo| %>
  <tr>
    <td><%= repo.id %></td>
    <td><%= repo.name %></td>
    <td><%= link_to repo.html_url, repo.html_url %></td>
  </tr>
<% end %>
```

コントローラスペックだと、viewをrenderしないようなので、それでエラーにならないため都合が良かったのです。


```rb
require 'rails_helper'

RSpec.describe 'Homes', type: :system do
  context 'モックに置き換える場合' do
    # 苦肉の策として、わざわざusersテーブルとuserモデルを作りました
    # system specなので、view側で何かしらeachで回した時にエラーにならないものを作る必要がある
    let!(:users) { User.create(name: 'testuser', html_url: 'www.yahoo.co.jp') }

    it '検索するとOctokitのrepositoriesメソッドが走ること' do
      mock = double('octokit')
      allow(Octokit::Client).to receive(:new).and_return(mock)
      allow(mock).to receive(:repositories).with('something').and_return(User.all)

      visit root_path
      fill_in 'ユーザー名', with: 'something'
      click_button '検索'
      expect(page).to have_content 'testuser'
      expect(page).to have_content 'www.yahoo.co.jp'
    end
  end
end
```

とはいえ、これはあまりにも力技過ぎます。
どうすればよいのでしょうか。

## 力技を使わずしてどうすればいいのか

まず、byebugを使って`@repos`を調べてみました。
すると、ハッシュの配列が返ってきていることがわかりました。

例えば、このURLをGETメソッドで叩くとイメージが湧きやすいかと思います。
  - https://api.github.com/users/miketa-webprgr/repos

このレスポンスの形式を真似して、以下のような仮のレスポンスを用意しました。
具体的にはこんな感じでした。

これを`User.all`に代わって、`and_return`してみたいと思います。

```rb
  let!(:users) { [
    {
      id: 313595631,
      name: "aws-study",
      html_url: "https://github.com/miketa-webprgr/aws-study",
    }
  ] }
```

ただ、調べていて分かりましたが、{}で囲まれているものはハッシュかと思いきや、
`Sawyer::Resource`というクラスのインスタンスでした。

`Sawyer::Rerouce`クラスのインスタンスを置き換える方向で検討をしましたが、
色々と調べていく中で、だいぶ難しそうだとわかりました。

どうりで、`@repos.each` したものが`id`メソッドを使えるんですね。。。

なお、APIのドキュメントにも詳細な説明が書いてあります。
  - https://docs.github.com/en/rest/reference/repos#list-repositories-for-a-user

## webmockとvcrを使ってみる

### webmockとは

webmockを使うと、外部APIを叩いときのレスポンスをstubに差し替えることができます。

```rb
stub_request(:get, "www.example.com").
  to_return({body: "abc"})

# 以上のようにstubすると、responseがabcになる
Net::HTTP.get('www.example.com', '/')    # ===> "abc\n"
```

- [bblimke/webmock: Library for stubbing and setting expectations on HTTP requests in Ruby\.](https://github.com/bblimke/webmock)

### vcrとは

Vcrとは、Video Casette Recorderの略。
要は、日本でいうところのVTR（Video Tape Recorder）のことである。

Webmockを使うと、偽物のレスポンスに置き換えることができるので便利だが、自前で偽物のレスポンスを作る必要がある。これは、非常に手間がかかる。

そこでvcrを使うとよい。
外部APIを１回叩くだけで、そのレスポンス結果を録画（保存）することができる。
vcrにはレスポンス結果が保存されるので、２回目以降はその偽物に差し替えることができる。

- [vcr/vcr: Record your test suite's HTTP interactions and replay them during future test runs for fast, deterministic, accurate tests\.](https://github.com/vcr/vcr)

### 設定方法

コピペ設定の粋を出ていませんが、このyoutubeを参考にしたら大体うまくいきました。

- [Ruby Snack \#75: Testing with VCR](https://www.youtube.com/watch?v=kBKuHPdE5Kg)

ただし、日本語があるとvcrのデータがうまく表示されませんでした。そこで、このブログを参考にして一部設定を追加したところ、うまくいきました。
- [VCRでWeb APIのモックを楽しよう！ \- アクトインディ開発者ブログ](https://tech.actindi.net/2019/06/07/093705)

また、今回の設定では使いませんでしたが、この動画も詳細に説明してくれていて分かりやすかったです。

- [Stubbing External API Calls in Rails \(Improved Audio\)](https://www.youtube.com/watch?v=Okck4Fc557o&t=0s)

設定方法については、GitHubの方を参照ください。
- [vcr設定方法](https://github.com/miketa-webprgr/mock_practice/commit/c90ab5ea8a5854d0cb6fa0ea062cbdb30e65b525#diff-053aa3ea46cb7c6a649e0d0fc592ce729b66e1d44be475516fd58857ba695862)
- [webmock設定方法](https://github.com/miketa-webprgr/mock_practice/commit/98c805c0cf09657254f931ac2ea7add610baf06f#diff-053aa3ea46cb7c6a649e0d0fc592ce729b66e1d44be475516fd58857ba695862)

## Webmock・VCRを使ったテスト

とりあえず、コントローラスペックのテストを貼っておきます。

モックを使う場合、VCRを使う場合（実際にAPIを叩く場合）、Webmockを使う場合の３パターンを掲載しておきます。

```rb
describe HomesController do
  describe '#GET' do
    it 'mockに置き換える' do
      mock = double('octokit')
      allow(Octokit::Client).to receive(:new).and_return(mock)
      expect(mock).to receive(:repositories).with('something')

      get :show, params: { username: 'something'}
      expect(response).to have_http_status(200)
    end

    it 'miketa-webprgrで検索すると検索結果が返ってくること', :vcr do
      get :show, params: miketa_params
      expect(response).to have_http_status(200)
    end

    it 'webmockに置き換える' do
      username = 'something2'
      stub_request(:get, "https://api.github.com/users/#{username}/repos").to_return(
        status: 200)

      get :show, params: { username: 'something2' }
      expect(response).to have_http_status(200)
    end
  end
end
```

なお、vcrを使う場合、このような感じでymlファイルでresponse内容が保存されます。

- [ymlファイル](https://github.com/miketa-webprgr/mock_practice/commit/c90ab5ea8a5854d0cb6fa0ea062cbdb30e65b525#diff-c09297b0709d2d8bfb06589acb659ababd2ab82a2860f863429f67047a038927)


## 最後に

- スタブやモックを使う場合、テストをすべき範囲とする必要がない範囲を切り分けること
- stubで返り値を置き換えられる、mockでインスタンスを置き換えられる
- 外部APIの返り値はwebmockで置き換えられる
- 外部APIの返り値を作るのが面倒な場合、vcrを活用するといい

<br>

---

<br>

### mockついて(ボツ)

次に、モックの説明がありました。

このような事例を想定してください。

かなり無茶苦茶ですが、ImageFlipperクラスがあるとします。
そして、インスタンスを作成し、MiniMagickのような画像処理ライブラリを引数とします。

次に、`image_flipper.change_image('ruby.jpg')`という感じで実行すると、
画像処理ライブラリのflipメソッドにより、画像が反転されるとします。（そう考えてください！）

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

エラーが起きないか確認するだけに留めるにしろ、画像を実際に反転していては、
テストの実行時間が長くなってしまいます。

また、外部APIのテストの場合、叩ける回数に制限がかかってきます。
毎回叩くのは現実的ではありません。

そこで、妥協をしましょう。
change_imageというメソッドを実行した際に、"mini_magick"のflipメソッドが
"ruby.jpg"に対して実行されていればよしとしましょう。

```text
# 補足
 - "mini_magick"を"mini_moni"にしても"ruby.jpg"を"diamond!!!"にしても全く問題ないです
 - 実際のところはnewの引数がchange_imageというメソッドを実行した際に、flipメソッドが呼ばれるかという関係性のテストをしています
 - ここではmini_magickというライブラリのことや、ruby.jpgという具体的な画像については一切関与していません
```

この際、doubleというメソッドを使って「影武者」を作ります。
そして、image_processorをモックに置き換えます。

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

今回の場合、mini_magickが壊れていないかどうか、画像が反転しているかどうかまでは、適度に実際に試して確認すれば問題がないので、このようなmockで問題がないわけです。

テストの範疇を適切に狭めることで、高速にテストをガンガン走らせて、安全にコーディングができるようになるのです！
