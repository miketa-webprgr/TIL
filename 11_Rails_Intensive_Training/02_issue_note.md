# Issue02 投稿のCRUD機能実装

## 求められている機能実装について

1. 投稿のCRUD機能実装
2. その他設定を行ってください
    - ユーザーとポストのシードファイルを作る
    - その際fakerを使ってダミーテキストを生成する
    - 画像のアップロードにはcarrierwaveを使用する
    - image_magickを使用して、画像は横幅or縦幅が最大1000pxとなるようにリサイズする
    - 画像は複数枚アップロードできるようにする
    - Swiper使って画像をスワイプできるようにする
    - 諸々のアイコンにはfontawesomeを使用する（おそらく導入済）

### 分からない単語・概念等の一覧

- carrierwave
- image_magick
- faker
- swiper
- fontawesome

## Carrierwave

今回の核となる実装。  
別ファイルにまとめた。  

複数枚のアップロード方法についてもまとめた。  

[CarrierWaveについてのノート](02_issue_note_carrierwave.md)  

## image_magick

`MiniMagick`と`RMagick`がある。  
公式では、"MiniMagick is recommended."と書いてある。  

今回は、あくまで`CarrierWave`の中でアップローダの中で設定するだけに留まる。  

[CarrierWave\+MiniMagickで使う、画像リサイズのメソッド \- Qiita](https://qiita.com/wann/items/c6d4c3f17b97bb33936f)  

## faker

ダミーデータを自動生成してくれるもの。  

使い方は非常にシンプル。  
GitHubのReadmeを読むと、異常なほどのダミーが作れることにビビる。  

わかりやすいので、英語でもこれは公式を読んでもいける！  

> [Github: faker\-ruby/faker](https://github.com/faker-ruby/faker)  
> [Fakerを使ってみました！（使い方と実行例） \- Qiita](https://qiita.com/ginokinh/items/3f825326841dad6bd11e)

## swiper

すごいサイトがあった。  
めちゃくちゃ分かりやすい。  

> [サンプル付き！簡単にスライドを作れるライブラリSwiper\.js超解説（基礎編） \| ガリガリコード](https://garigaricode.com/swiper/)

今回の場合、この仕組みさえ分かれば、解読できる。  

```html
<!-- Swiper START -->
<div class="swiper-container">
	<!-- メイン表示部分 -->
	<div class="swiper-wrapper">
		<!-- 各スライド -->
		<div class="swiper-slide">Slide 1</div>
		<div class="swiper-slide">Slide 2</div>
		<div class="swiper-slide">Slide 3</div>
		<div class="swiper-slide">Slide 4</div>
	</div>
	<div class="swiper-pagination"></div>
</div>
<!-- Swiper END -->
```

## fontawesome

まだ調べられていない。  

## 投稿のCRUD機能実装

これからコントローラとモデル関係の実装を追っていく。  

## View関係の実装

コードを見ても、何がどうなっているのか解読が難しかったので、  
愚直に画面に対応関係を記してみた。  

editのコードはシンプルだったので、画像だけ貼り付けた。  
なお、newのコードはeditと変わらない。  

【index】<br>
<img src="02_posts_index.png" width=300 border="1"><br>

【show】<br>
<img src="02_posts_show.png" width=300 border="1"><br>

【edit】（new)<br>
<img src="02_posts_edit.png" width=300 border="1"><br>


## 動作確認方法

1. git clone https://github.com/miketa-webprgr/instagram_clone.git
2. git checkout git checkout -b feature/01_login_logout origin/feature/01_login_logout
3. bundle install
4. yarn install
5. MySQL と Redis を立ち上げる
