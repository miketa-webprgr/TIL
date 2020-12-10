# 【第１回】 Railsガイドの「Action View フォームヘルパー」をひたすらまとめていく 【Formの概要編】

## この連載の概要

- フォーム部分の理解が不十分だと感じたので、Railsガイドで勉強していく
  - [Action View フォームヘルパー \- Railsガイド](https://railsguides.jp/form_helpers.html)
  - ざっくりと言うと、form_with関係とも言えますが、それ以外も対象となってます
- クリスマスまでを目処に、ちょこちょことまとめて記事をあげていく
  - ひとりアドベントカレンダーなる斬新なアイデアをかろりーなさんから頂きました笑
- Crieitにも近々掲載していきたい
- 小難しくしないのをコンセプトとしたいので、不正確な表現も増えそう
  - そもそも初学者なのでふつーに間違いもありそうですが、そこはご愛敬ということで

## 第１回の概要

- 初回なので、Railsガイドの解説に突入する前に、フォーム自体についてまとめる

## フォームとは

- フォームとは、Webアプリを使う人が**サーバーに何かを送るときに使う**
- 例えば、ユーザー登録のときにメールアドレスやパスワード登録をするときなど
- ここなんかが分かりやすそうだった
  - [formタグとは｜コーディングのプロが作るHTMLタグ辞典](https://html-coding.co.jp/annex/dictionary/html/form/)

初学者でも、何かを送るものって意識はあるだろうけど、以下の意識は薄いのではないか。  
フォームでは、以下が定義されている。  

- どこのURLに
- どのようなメソッドで
- どのようなパラメータを送るか
  - keyとvalueは何か
- フォームの種類は何か
  - HTMLのinputで指定する
  - text_fieldなどがある
  - 実は20種類以上もある。。

What, Where, How,（ + Type of input)と覚えるとよいかも。（自己流です）  
2W1HT・・・なんかいい語呂でもないかな。。。  

まず、フォームの概要について押さえておく。  
ここはRailsガイドに掲載されていないが、把握すべき基礎的な内容だ。  

## formタグの概要について

```html
<form accept-charset="UTF-8" action="/users" method="post">
  <input type="email" name="user[email]" value="abc@example.com" />
  <input type="text" name="user[name]" value="alice" />
  <input type="text" name="user[age]" value="20" />
</form>
```

action: アクセス先のURL
method: HTTPメソッド
type: フォームの入力値の種類（text, email, radio, checkboxなど）
name: フォームの名前を示している
value: 値

アクションは、あくまでアクセス先のURLである。  
間違いやすいので注意すること。  

paramsハッシュの中身は、以下のとおりとなる。  

`{'user' => {'email' => 'abc@example.com'}, {'name' => 'alice'}, {'age' => 20} }`

このような形で書くことで、params[:user]と書くことで、Userモデルに紐づく全てのハッシュを取得できる。  
また、以下のように取得するハッシュを制限することができる。  

```rb
params.require(:user).permit(:email)
```

## Labelタグについて

フォームでは、よくラベルタグを使う。  
フォームを使うメリットは、２つある。  

- ラベルをクリックすると、そのチェックボックスをクリックしたことになる。  
- 音声読み上げソフトが、いい感じに対応してくれる。  

ラベルタグは、インプットタグと紐付ける。  
紐付けにあたっては、labelのforとinputのidを同一にする。  

以下の事例においては、cheeseで紐付けされている。  
繰り返しになるが、nameはパラメータのkeyなので、勘違いしないように。  
（keyというのは用語が適切でないかも。Hashのkey-valueのkey）  

```html
<div class="preference">
    <label for="cheese">Do you like cheese?</label>
    <input type="checkbox" name="cheese" id="cheese">
</div>
```

Form_withでは使われないが、labelタグにinputタグをネストすることでも紐付け可能。  

```html
<label>Do you like peas?
  <input type="checkbox" name="peas">
</label>
```

- [<label> \- HTML: HyperText Markup Language \| MDN](https://developer.mozilla.org/ja/docs/Web/HTML/Element/label)

## Formの隠し要素

Railsでフォームを作ると、文字エンコードとauthenticity_tokenが自動で含まれる。  
authenticity_tokenはCSRF対策のため。GETメソッドの場合、このトークンは作成されない。  

Railsサーバーに対して、このトークン抜きでリクエストを送っても拒否される。要注意。  
（ただし、Railsの設定次第ではトークン抜きでも受け付けることができたはず）  

```erb
<%= form_tag do %>
  Form contents
<% end %>
```

```html
<form accept-charset="UTF-8" action="/home/index" method="post">
  <div style="margin:0;padding:0">
    <input name="utf8" type="hidden" value="&#x2713;" />
    <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c432144748a6d4b7a7ddf2b71" />
  </div>
  Form contents
</form>
```

## CSRFとは

CSRFとは、クロスサイトリクエストフォージェリーの略。  
まず、リクエストフォージェリーに着目するとよさそう。  

リクエストとは、サーバーに送るリクエストのこと。  
例えば、ユーザー登録をするのは、クライアントからサーバーへのリクエスト。  

フォージェリとは、偽物のこと。  

で、リクエストフォージェリー。  
これは、悪意のある人がJavaScriptなんかでこっそりコードを仕込んでおくことで、  
その人の意志とは関係なく、サーバー側にリクエストを送ってしまうこと。  

え、そんなことできるのかって。できる。  
JavaScriptを使うと、例えば何かの要素をクリックした際に、  
特定のURLに対してリクエストを送るよう指示を出すことができる。  

この機能を使って、例えばtwitterにログインしているユーザーを対象にして、  
クリックしたと同時に、そのユーザーに知らず知らずの内に、サーバー側にリクエストを投げることができる。  

例えば、殺害予告ツイートの投稿リクエストを投げることができる。  
ログインした状態で、まさにそのユーザーからリクエストが送られてしまうので、  
このリクエストは受け入れられてしまい、意図していないのに殺害予告ツイートをしてしまうハメになる。  

たぶん、概念的にはこんな感じの理解で大枠あっていると思う。  

## authenticity_tokenでCSRFがなぜ対策できるのか

試してみると分かるが、このtokenはサーバー側から毎回ランダムに生成される。  
同じページであっても、リロードすると変わるはずだ。  

このトークンは、投稿時にサーバー側に合わせて送られる。  
そして、このトークンがサーバー側で事前に送ったものと一致するか検証がなされる。  

つまり、イメージとしては、こんなかんじ。  
まず、OKパターン。  

1. サーバー「ユーザー登録するときは、開けゴマって言うこと」
2. クライアント「開けゴマ！ ユーザー登録します。」
3. サーバー「OK！ 登録する！」

NGパターン。  

1. クライアントがCSRFにひかかってしまった
2. 意図していないのに「〇〇を17時に爆破する」というツイート登録がサーバーに飛ぶ
3. サーバー「開けゴマ！がないから受け付けません」

この開けゴマがtoken。  

これがリロードする度に変わるので、サイトをまたいで、つまりクロスサイトで  
フォージェリーなリクエストを送ることを防げる。  

## 最後にテスト

まず、これが自分の言葉で説明できるか考えてみよう。  

```html
<form accept-charset="UTF-8" action="/users" method="post">
  <input type="email" name="user[email]" value="abc@example.com" />
  <input type="text" name="user[name]" value="alice" />
  <input type="text" name="user[age]" value="20" />
</form>
```

次に、ラベルって何か自分の言葉説明できるか考えてみよう。  
そして、空欄に何が入るか振り返ってみよう。  

```html
<div class="preference">
    <label for="cheese">Do you like cheese?</label>
    <input type="checkbox" name="cheese" id="cheese">
</div>
```

最後にauthenticity_tokenって何だったか自分の言葉で説明できるか考えてみよう。  

## 補足

自分で書いておきながら説明できないかも。。。  
あと、これは書きダメベースなので、次回から尻すぼみます。  