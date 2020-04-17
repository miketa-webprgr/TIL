# Chapter 4 - Webのさまざまなデータ形式

### ○ はじめに

Webで使われるさまざまなデータ形式についてまとめました。<br>
特に、DOM・JSON・Atom・マイクロフォーマットなどは馴染みがないので、<br>
今後は具体的なレベルでの理解を深めていきたい。

### ○ HTMLとは

- タグで構成される。属性も追加可能。
  - タグとは<>のこと。
  - 属性とは、<~ class="">で表現されるもの。classに限らない。
- 最新のバージョンは5.1
- HTML内にDOCTYPE宣言があり、それによってバージョンを指定する

### ○ 画像形式とは

##### ● JPEG

- JPEG = Joint Photographic Experts Group の略　（作った団体の名称）
- イラストよりも、写真に使われる。
- 1677万色を表現できる。
- 不可逆性があり、画像を圧縮すると粗くなってしまう。

##### ● GIF

- GIF = Graphic Interchange Format の略
- イラストに使われる。
- 256色しか表現できないが、透明色も利用できる。
- 可逆性があり、画像を圧縮しても粗くならない。
- パラパラ漫画のようなアニメーションが表現できる。
- UNISYS社の特許の問題があったが、2004年に失効。

##### ● PNG

- PNG = Portable Network Graphics の略
- イラストにも写真にも使える。
- 1677万色を表現できる。透明色も使える。透明具合も指定できる。
- 可逆性があり、画像を圧縮しても粗くならない。
- GIFよりファイルサイズが小さくなる。
- パラパラ漫画のようなアニメーションは、表現できない。
- W3Cが開発。GIFの特許の問題を受け、開発。

### ○ XMLとは

- XML = Extensible Markup Language の略
  - Extensible = 拡張的な
  - 汎用的に使えるマークアップ言語という意味
- タグを自分で定義でき、個別の目的に応じて使うことが可能
  - 例えば、<目次>のようなタグを自分で定義できる。
  - 実例を見るのが一番わかりやすい。
- XHTMLは、 XMLの文法で書かれたHTML
  - brなどのタグなどは、XMLの文法上、「/br」などで閉じる必要あり
  - XHTML化することで、XMLを読み込むことができる。
 
### ○ CSSとは

- CSS = Cascading　Stylesheets の略
- HTMLの体裁を記述する。
- 同一のCSSファイルを複数のHTMLに適用できる。
  - Webサイトの統一感を確保できる。
  - HTML内に組み込めるが、別ファイルとして保存することが多い。

### ○ スクリプト言語とは

- フロントサイド言語とサーバーサイド言語がある。
- フロントサイド言語　
  - Javascript
    - 厳密には、ECMAscript
    - 以前には、規格が2つあり、統一されていなかった。
    - Mozilla社のJavascriptとMicrosoft社のJScriptがあった。
  - サーバーサイド言語
    - Perl
    - 古くからあり、記述の自由度が高い
  - Python
    - 可読性が高く、記述の厳密性が求められる
    - 近年だと機械学習やデータ分析ではPythonという流れがある
  - PHP
    - 元々Webページで使うことを想定されて開発された言語
  - Ruby
    - Object指向の言語。まつもとさんという日本人が開発した。
   
### ○ DOMとは

- DOM = Document Object Model の略
- Documentについて、タグで囲まれた各Objectをツリー構造にモデル化する仕組みのこと
 - つまり、HTMLの文書をツリー構造化してくれるソフトウェア
 - HTMLだけではなく、 XMLなども扱うAPI
- ツリー構造化された際の各要素はノードと呼ばれる。<br>

><a href="https://ja.wikipedia.org/wiki/Document_Object_Model">
  Document Object Model</a><br><br>
フリー百科事典 『ウィキペディア（Wikipedia）』<br>

><a href="https://eng-entrance.com/what-is-dom">
  JavaScript初心者でもすぐわかる！DOMとは何か？</a><br><br>
エンジニアの入り口（Linuxアカデミー）<br>

### ○ JSONとは

- JSON = Javascript Object Notation の略
- Javascriptの書式にて、XMLを書いてようなもの
  - 実際の事例を見ると分かりやすい
  - XMLはタグで囲んでいるが、JSONはコロンなどで表記している
- Javascriptの文法で書かれているので、以下の特徴がある
  - 可読性が低い
  - Javascriptのでの操作が直接できる
    - XMLの場合、Javascriptで操作する場合はDOMを挟む必要がある

### ○ JSONとは

- JSON = Javascript Object Notation の略
- Javascriptの書式にて、XMLを書いてようなもの
  - 実際の事例を見ると分かりやすい
  - XMLはタグで囲んでいるが、JSONはコロンなどで表記している
- Javascriptの文法で書かれているので、以下の特徴がある
  - 可読性が低い
  - Javascriptのでの操作が直接できる
    - XMLの場合、Javascriptで操作する場合はDOMを挟む必要がある

### ○ フィードとは

- Webサイトの更新情報を配信するファイル
- RSSとAtom
  - Netscape社の RDF Site Summaryからスタート
    - RDF = Resource Description Framework
  - そこから分派し、RSS１.0と２.０系列に分かれた
    - RSS1.0は表現力が豊かだが、構文が複雑
    - RSS2.0は表現力に制限があるが、構文がシンプル
  - 分派したことに嫌気がさした人々がXMLベースのAtomを開発
- フィードリーダー（RSSリーダー）
  - RSSファイルを収集し、更新情報を効率的に閲覧できるソフト
- Podcast
  - RSSを用いて、Web上に音楽ファイルをネット配信する手法
  
### ○ マイクロフォーマットとは

- HTMLやXMLに意味を与える
- Semantic Webの実現
  - classタグなどで意味を与えていき、その情報を活用していく取り組み
- 統一的な規格はなく、各団体が色々な規格を作成している
  - hCard
  - hCalender
  - hAtomなど

### ○ 音声動画とは

- 圧縮にはコーデックというソフトが用いられる
  - 圧縮 = encording
  - 伸長 = decoriding （再生する際にはデコードされる）
- 音声ファイルの形式
  - mp３
  - acc（iPodなど）
  - wmaなど
- 動画ファイルの形式
  - mpeg2(DVDや地デジ）
  - mpeg4(スマホなど）
  - wmvなど
- 配信形式
  - ダウンロード形式
  - プログレッシブダウンロード形式
    - Youtubeやニコ動はこの形式
  - ストリーミング形式
    - abemaなどはこの形式

><a href="https://qiita.com/you8/items/e903fd463cf770688e1e">
  RSS、atomの仕様をまとめる</a><br><br>
Qiita記事 @you8<br>

### ○ メディアタイプとは

- 機器の種類
- HTML上で <~ media="">とすることで、それぞれの機器に合わせた表現が可能
- css上で @media とすることで、同様の対応が可能
  - tv（テレビ）
  - tty（文字幅固定の機器）
  - handheld（スマホなど）
