# Issue14 SEO対策 メタタグの設定

## どんな感じ？

頑張って写真を撮りました。  

|slack（トップ）|slack（投稿詳細）|
|:----|:----|
| <a href="https://gyazo.com/a9a9277ff8b0b5077acb82e575444e19"><img src="https://i.gyazo.com/e408db6f78527cf6cafbb9bdd9137b4b.png" alt="Image from Gyazo" width="400"/></a> | <a href="https://gyazo.com/a9a9277ff8b0b5077acb82e575444e19"><img src="https://i.gyazo.com/b85a09465d577daabef1c30528c586b1.png" alt="Image from Gyazo" width="400"/></a> |

| twitter（トップ）| android（トップ）|
|:----|:----|
| <a href="https://gyazo.com/a9a9277ff8b0b5077acb82e575444e19"><img src="https://i.gyazo.com/b99b7904ca5fa47e7af921861f480ae1.png" alt="Image from Gyazo" width="400"/></a> | <a href="https://gyazo.com/a9a9277ff8b0b5077acb82e575444e19"><img src="https://i.gyazo.com/a9a9277ff8b0b5077acb82e575444e19.png" alt="Image from Gyazo" width="400"/></a> |

## 求められている機能実装・実装条件について

- title, description, keywordが適切に設定する
- ngrokを使って一時的にインターネット上にサイトを公開する
- Slackに投稿した時にメタタグが本当に反映されているか確認する

| |トップページ|ユーザー一覧ページ|ユーザー詳細|投稿詳細|
|:----|:----|:----|:----|:----|
|title|InstaClone - Railsの実践的アプリケーション|ユーザー一覧ページ | InstaClone|ユーザー詳細ページ | InstaClone|投稿詳細ページ | InstaClone|
|description|Ruby on Railsの実践的な課題です。<br>Sidekiqを使った非同期処理やトランザクションを利用した課金処理など実践的な内容が学べます。|← 同じ|← 同じ|投稿のbody|
|keyword|rails, instaclone, rails特訓コース|← 同じ|← 同じ|← 同じ|
|OGP画像|デフォの|デフォの|デフォの|1枚目の画像|

## そもそもSEOとは

SEO = Search Engine Optimization  
日本語だと、検索エンジン最適化。  

つまり、Googleが検索エンジン市場を寡占している状況においては、SEO = Google対策。  
Google検索をした際に、どうしたら上位に表示させることができるか問題のこと。  

どのサイトがよいか分からないが、以下のサイトがヒットした。  
また、Googleが公式に出している情報があるので、そちらを参照するのがよさそう。  

- [SEOとは？ 押さえておくべき24のSEO対策方法とポイント](https://www.gyro-n.com/seo/hack/seo-point/)
- [検索エンジン最適化（SEO）スターター ガイド \- Search Console ヘルプ](https://support.google.com/webmasters/answer/7451184?hl=ja)
- [Googleが作成したSEOに関するスターターガイド（PDF）](https://static.googleusercontent.com/media/www.google.co.jp/ja/jp/intl/ja/webmasters/docs/search-engine-optimization-starter-guide-ja.pdf)

## `gem 'meta-tags'`について

こちらを参照するとよい。  

- [【公式GitHub】kpumuk/meta\-tags: Search Engine Optimization \(SEO\) for Ruby on Rails applications\.](https://github.com/kpumuk/meta-tags)
- [【Rails】『meta\-tags』gemを使ってSEO対策をおこなう方法 \| vdeep](http://vdeep.net/rubyonrails-meta-tags-seo)

SEO対策としてやるべきことはたくさんあるが、ここでは内部的対策として、  
`title, description, keyword`の設定を行う。

設定することでどのような意味があるかについては、`gem 'meta-tags'`の公式GitHubにて解説されている。
また、公式ではMozというサイトが紹介されている。

- [【公式GitHub】kpumuk/meta\-tags（SEO Basics and MetaTags）](https://github.com/kpumuk/meta-tags#seo-basics-and-metatags)
- [\(Meta\) Title Tags \+ Title Length Checker 2020 \- Moz](https://moz.com/learn/seo/title-tag)

### Title

イメージしやすいように、Mozというサイトから拝借した。  
以下のとおり、Titleは採用される。  

<a href="https://gyazo.com/261ee0d895ea5e63520cce803a3ce8d3"><img src="https://i.gyazo.com/261ee0d895ea5e63520cce803a3ce8d3.png" alt="Image from Gyazo" width="500"/></a><br>

- 以下で採用され、サイトの「顔」となる
  - 検索エンジンの検索結果
  - ブラウザのタブ
  - SNSなど
- 文字数に注意すること
  - English: 50-60characters
  - 日本語：35字以内程度
- 流入検索ワードが自然と含まれていること
  - 読みづらい、不自然なタイトルはNG

### Description

- 文字数に注意すること
  - English: 2-3 lines, up to 300 characters
  - 日本語：130字以内程度
  - デバイスや仕様変更に左右されるので、あくまで目安として
- 引きのある文章にすることで、被クリック率をあげることができる
- Mozによると、2009年時点ではDescriptionは検索エンジンのアルゴリズムに影響しないとのこと

### Keyword

- 検索エンジンで流入してもらいたいキーワードを指定できる
- 文字数、キーワード数に注意すること
  - English: up to 255 characters, 20 words.
  - 日本語： 不明、5個ぐらいまでにした方がいいとの情報が多かった
  - 不要なキーワードをつけ過ぎるとペナルティを受ける
- GoogleもBingも、keywordは検索エンジンのアルゴリズムに影響しないと公言したらしい（10年前くらい）
  - 業者のSEO対策があまりにもひどかったためらしい
  - アルゴリズムに影響しないため、検索ワードを設定しないところも多いらしい

### canonical

例えば、PC用とスマホ用のサイトでURLが違う場合、どちらかを元サイトとして指定してあげることで、  
検索エンジンが正しく同一の内容のものだと認識してくれる。  

このような設定をしないと検索エンジンは別サイトとして認識するため、評価が2分してしまうなど、  
場合によっては不利益を受けることがある。  

### Open Graph Protocol

FacebookやTwitterなど主にSNS上でうまく情報をshareするために必要となる設定である。  

### その他

公式で一通り説明されているので、そこから必要な情報を適宜深掘りするのが良さそう。  

## ngrokについて

公式に書いてあるとおり、設定すればよい。  
15分もあれば、簡単にネット上で公開できる。  

1. 必要なファイルをインストール
2. ネット上のサイトでログインし、authtokenを取得
     - `./ngrok authtoken <your_auth_token>`
3. コマンドを入力して、ネット上に公開
    - `./ngrok http 80`

### 参考サイト

- [ngrok \- download](https://ngrok.com/download)
- [ngrokが便利すぎる \- Qiita](https://qiita.com/mininobu/items/b45dbc70faedf30f484e)

## 実装内容

以下にて、`gem 'meta-tags'`を使ったSEO対策の実装を行っていく。  

まず、様々なメタタグの種類については、以下のQiita記事などで確認するとよい。  

- [色々あるHTMLのmetaタグなど一覧 \- Qiita](https://qiita.com/tsuka-rinorino/items/3b4fb69d980cecddf512#%E3%83%95%E3%82%A1%E3%83%93%E3%82%B3%E3%83%B3%E3%81%AE%E6%8C%87%E5%AE%9A)

繰り返しになるが、基本的にはこちらを参照するとよい。  

- [【公式GitHub】kpumuk/meta\-tags: Search Engine Optimization \(SEO\) for Ruby on Rails applications\.](https://github.com/kpumuk/meta-tags)
- [【Rails】『meta\-tags』gemを使ってSEO対策をおこなう方法 \| vdeep](http://vdeep.net/rubyonrails-meta-tags-seo)

### `gem 'meta-tags'`の導入

まず、`gem 'meta-tags'`をインストールする。  
続いて、`bundle exec rails generate meta_tags:install`にて`config/initializers/meta_tags.rb`を作成する。  

文字数制限などを設けることができるが、特に不要であればそのままでも問題がない。  

```rb
# onfig/initializers/meta_tags.rb

# frozen_string_literal: true

# Use this setup block to configure all options available in MetaTags.
MetaTags.configure do |config|
  # How many characters should the title meta tag have at most. Default is 70.
  # Set to nil or 0 to remove limits.
  # config.title_limit = 70

  # When true, site title will be truncated instead of title. Default is false.
  # config.truncate_site_title_first = false

  # Maximum length of the page description. Default is 300.
  # Set to nil or 0 to remove limits.
  # config.description_limit = 300

  # Maximum length of the keywords meta tag. Default is 255.
  # config.keywords_limit = 255

  # Default separator for keywords meta tag (used when an Array passed with
  # the list of keywords). Default is ", ".
  # config.keywords_separator = ', '

  # When true, keywords will be converted to lowercase, otherwise they will
  # appear on the page as is. Default is true.
  # config.keywords_lowercase = true

  # When true, the output will not include new line characters between meta tags.
  # Default is false.
  # config.minify_output = false

  # When false, generated meta tags will be self-closing (<meta ... />) instead
  # of open (`<meta ...>`). Default is true.
  # config.open_meta_tags = true

  # List of additional meta tags that should use "property" attribute instead
  # of "name" attribute in <meta> tags.
  # config.property_tags.push(
  #   'x-hearthstone:deck',
  # )
end
```

### `gem 'meta_tags'`の使い方

公式に書かれているとおり、以下のとおり書く。  

`display_meta_tags`メソッドを使って、メタタグを表示する。  
`title`や`description`などのメソッドを使って、メタタグをセットする。  

```html
<head>
  <%= display_meta_tags site: 'My website' %>
</head>
<body>
  <h1><%= title 'My page title' %></h1>
</body>
```

すると、このようになる。  

```html
<head>
  <title>My website | My page title</title>
</head>
<body>
  <h1>My page title</h1>
</body>
```

なお、`set_meta_tags`を使って一度に複数のメタタグをセットすることもできる。  

```rb
<% set_meta_tags title: 'Member Login',
                 description: 'Member login page.',
                 keywords: 'Site, Login, Members' %>
```

`to_meta_tags`メソッドを使うと、このような書き方もできる。  
ActiveRecordクラスのオブジェクトを引数とすることができる。  

```rb
class Document < ApplicationRecord
  def to_meta_tags
    {
      title: title,
      description: summary,
    }
  end
end

@document = Document.first
set_meta_tags @document
```

・・・まあ、全ては公式の丸写しです。  

### 実装手順

以下のブログなどで記載されているとおり、全般に共通する内容を`layout/application.html.slim`に記載し、  
個別具体的な内容については、個別のページで上書きする形がシンプルな形となる。  

なお、application.html.slimに全て書くと見辛くなるので、`display_meta_tags(default_meta_tags)`として、  
helperファイルにて、`default_meta_tags`の内容を記述するのがスマートである。  

よって、`gem 'config'`と組み合わせるような形で以下のとおりにするとよい。  

```rb
# application_helper.rb

module ApplicationHelper
  def default_meta_tags
    {
      # サイト名
      site: Settings.meta.site,
      # trueの場合、ページのタイトル名 → サイト名 という表記の仕方になる
      reverse: true,
      # ページのタイトル名
      title: Settings.meta.title,
      # ページの概要（検索サイトなどに表示される）
      description: Settings.meta.description,
      # 検索ワード（ただ、SEO的に意味がないらしい）
      keywords: Settings.meta.keywords,
      # 似たようなサイトがある場合、これを使ってまとめる
      canonical: request.original_url,
      # FacebookやTwitterでいい感じにリンクを表示させるための設定
      og: {
        title: :full_title,
        type: Settings.meta.og.type,
        url: request.original_url,
        image: image_url(Settings.meta.og.image_path),
        site_name: :site,
        description: :description,
        locale: 'ja_JP'
      },
      # Twitterでリンクを表示させる場合のカードの種類を設定
      twitter: {
        card: 'summary_large_image'
      },
      # アイコンを設定する（みけた独自）
      # 以下のサイトを利用して、勝手にやってみました。スマホの場合でもいい感じのアイコンを作ってくれるらしいです。
      # https://www.favicon-generator.org/
      icon: [
        { href: '/favicon/apple-icon-57x57.png', sizes: '57x57', type: 'image/png' },
        { href: '/favicon/apple-icon-60x60.png', sizes: '60x60', type: 'image/png' },
        { href: '/favicon/apple-icon-72x72.png', sizes: '72x72', type: 'image/png' },
        { href: '/favicon/apple-icon-76x76.png', sizes: '76x76', type: 'image/png' },
        { href: '/favicon/apple-icon-114x114.png', sizes: '114x114', type: 'image/png' },
        { href: '/favicon/apple-icon-120x120.png', sizes: '120x120', type: 'image/png' },
        { href: '/favicon/apple-icon-144x144.png', sizes: '144x144', type: 'image/png' },
        { href: '/favicon/apple-icon-152x152.png', sizes: '152x152', type: 'image/png' },
        { href: '/favicon/apple-icon-180x180.png', sizes: '180x180', type: 'image/png' },
        { href: '/favicon/android-icon-192x192.png', sizes: '192x192', type: 'image/png' },
        { href: '/favicon/favicon-32x32.png', sizes: '32x32', type: 'image/png' },
        { href: '/favicon/favicon-96x96.png', sizes: '96x96', type: 'image/png' },
        { href: '/favicon/favicon-16x16.png', sizes: '16x16', type: 'image/png' },
      ]
      # その他、favicon-generatorに勧められた内容を設定した（主にWindows用）
      name: {
        msapplication-TileColor: "#ffffff",
        application-TileImage: "/favicon/ms-icon-144x144.png",
        theme-color: "#ffffff"
      }
    }
  end
end
```

```yml
# config/settings.yml

meta:
  site: InstaClone
  title: InstaClone - Railsの実践的アプリケーション
  description: Ruby on Railsの実践的な課題です。Sidekiqを使った非同期処理やトランザクションを利用した課金処理など実践的な内容が学べます。
  keywords:
    - Rails
    - InstaClone
    - Rails特訓コース
  og:
    type: website
    # 画像をGitHubにpushしたくなかったので、URLで指定しています
    image_path: https://i.gyazo.com/9e198c10b972906d645516e86b89c66b.png
```

すると、`application.html.slim`に1行だけ書いたコードが、このようなHTMLに変換される。  
変換前がこちら。  

```slim
doctype html
html
  head
    meta content=("text/html; charset=UTF-8") http-equiv="Content-Type" /
    meta[name="viewport" content="width=device-width, initial-scale=1.0"]
    = display_meta_tags(default_meta_tags)
    = csrf_meta_tags
    = csp_meta_tag
```

変換後がこちら。  

```html
<!DOCTYPE html>
  <html>
  <head>
  <meta charset="UTF-8" />
  <meta content="width=device-width, initial-scale=1.0" name="viewport" />
  <title>InstaClone - Railsの実践的アプリケーション | InstaClone</title>
  <link rel="icon" type="image/png" href="/favicon/apple-icon-57x57.png" sizes="57x57">
  <link rel="icon" type="image/png" href="/favicon/apple-icon-60x60.png" sizes="60x60">
  <link rel="icon" type="image/png" href="/favicon/apple-icon-72x72.png" sizes="72x72">
  <link rel="icon" type="image/png" href="/favicon/apple-icon-76x76.png" sizes="76x76">
  <link rel="icon" type="image/png" href="/favicon/apple-icon-114x114.png" sizes="114x114">
  <link rel="icon" type="image/png" href="/favicon/apple-icon-120x120.png" sizes="120x120">
  <link rel="icon" type="image/png" href="/favicon/apple-icon-144x144.png" sizes="144x144">
  <link rel="icon" type="image/png" href="/favicon/apple-icon-152x152.png" sizes="152x152">
  <link rel="icon" type="image/png" href="/favicon/apple-icon-180x180.png" sizes="180x180">
  <link rel="icon" type="image/png" href="/favicon/android-icon-192x192.png" sizes="192x192">
  <link rel="icon" type="image/png" href="/favicon/favicon-32x32.png" sizes="32x32">
  <link rel="icon" type="image/png" href="/favicon/favicon-96x96.png" sizes="96x96">
  <link rel="icon" type="image/png" href="/favicon/favicon-16x16.png" sizes="16x16">
  <meta name="description" content="Ruby on Railsの実践的な課題です。Sidekiqを使った非同期処理やトランザクションを利用した課金処理など実践的な内容が学べます。">
  <meta name="keywords" content="rails, instaclone, rails特訓コース">
  <link rel="canonical" href="http://28320f91b759.ngrok.io/">
  <meta property="og:title" content="InstaClone - Railsの実践的アプリケーション | InstaClone">
  <meta property="og:type" content="website">
  <meta property="og:url" content="http://28320f91b759.ngrok.io/">
  <meta property="og:image" content="https://i.gyazo.com/9e198c10b972906d645516e86b89c66b.png">
  <meta property="og:site_name" content="InstaClone">
  <meta property="og:description" content="Ruby on Railsの実践的な課題です。Sidekiqを使った非同期処理やトランザクションを利用した課金処理など実践的な内容が学べます。">
  <meta property="og:locale" content="ja_JP">
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:image" content="https://i.gyazo.com/9e198c10b972906d645516e86b89c66b.png">
  <meta name="msapplication-TileColor" content="#ffffff">
  <meta name="application-TileImage" content="/favicon/ms-icon-144x144.png">
  <meta name="theme-color" content="#ffffff">
  <meta name="csrf-param" content="authenticity_token" />
  <meta name="csrf-token" content="fl6TurOVL/L1AowYRdgZbHl6gSu0SBNV8IlwE4o4NuHC4SsgP/FEzJMX90fzgwDZHoqF4AjZe+hADnqsc5VD8A==" />
```

あとは、各サイトを必要に応じて個別に`meta_tags`を設定していく。  
今回は、だいそんさんの例にならって、下記のとおりとした。  

```html.slim
app/views/posts/show.html.slim

- set_meta_tags title: '投稿詳細ページ', description: @post.body,
        og: { image: "#{request.base_url}#{@post.images.first.url}"} # 本来carrierwaveのasset_pathで設定すべき
```

```html.slim
/ app/views/users/index.html.slim

- set_meta_tags title: 'ユーザー一覧ページ'
```

```html.slim
/ app/views/user_sessions/new.html.slim

- set_meta_tags title: 'ログインページ'
```

```html.slim
/ app/views/users/new.html.slim

- set_meta_tags title: 'ユーザー登録ページ'
```

```html.slim
/ app/views/users/show.html.slim

- set_meta_tags title: 'ユーザー詳細ページ'
```
