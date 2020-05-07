# Chapter ２ - Webとネットワーク

### ○ Webの仕組みついてのおさらい

改めて、前回のまとめで貼った画像を貼っておきます。
<img width="700" alt="WebArchitecture" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/429155/3c364d67-b219-b041-1d49-1f4ff6edf35f.jpeg">  

以上の図のとおり、自宅のPCのChromeやIEなどにWebページが表示されるのは、
ChromeなどのWebブラウザからWebサーバーに対して「〜ページを表示して！」というリクエストが送られ、
それを受けたWebサーバーが該当のファイルを送り返しているからである。

そのやり取りを行うにあたって使われているのが、HTTP（HyperText Transfer Protocol)という
規格や、より安全なHTTPSという規格である。

「〜ページを表示して！」というリクエストの行き先は、統一資源位置指定子！！！（URLのことです。。。）に
よって特定され、IPアドレスという番号の羅列に変換されて、初めてPCが分かる行き先になる。

Webサーバーから送られるファイルは、サーバー内のサーバーサイド言語（Ruby・PHP・Python）などによって、
その都度生成されることがあり、これを「動的ページ」という。

ちなみに、FacebookやInstagramがユーザーごとにカスタマイズしたページを表示できるのは、
Webブラウザから受け取った情報を元にして、動的にWebページを生成しているからである。

### ○ DNSについて

先ほどの説明だと、単純にWebページの表示は自宅のPCとWebサーバーのみで完結しているように思われる。
だが、実際には、IPアドレスを見つける為に、その間にはDNSサーバーが介在している。

単純に自宅のPCがダイレクトにWebサーバーに行けるわけではなく、IPアドレス取得のため、
DNSサーバーへのスタンプラリーを終えなければいけない。

<img width="700" alt="WebArchitecture" src="https://www.nic.ad.jp/ja/dom/img/basics_b.gif"> 

これがそのスタンプラリーの様子だ。
会社の稟議のように、DNSルートサーバーを始めとして、多くのDNSサーバーを媒介しなければならない。

ところで、DNSとはDomain Network Systemの略称である。
要は、場所を特定するためのネットワークシステムということである。

文字どおり、場所特定のためのスーパーコンピュータがあるわけではなく、
複数のコンピュータが繋がった形で、それぞれが協力してIPアドレスを特定しているのである。

><a href="https://www.nic.ad.jp/ja/dom/system.html">ドメイン名のしくみ</a><br>
一般社団法人日本ネットワークインフォメーションセンターHP

### ○ TCP/IPについて

TCP/IPとは、インターネット上のプロトコルの総称を指す。
例えば、以下のようなプロトコルがある。
- HTTP（HTMLのプロトコル）
- FTP（Fileのプロトコル）
- SMTP（メール送信のプロトコル）
- POP（メール受信のプロトコル）

ちなみに、TCP/IPはいくつものレイヤー（層）に分かれている。
HTTPやFTPはアプリケーションというレイヤーに属している。

><a href="https://tomslifestylelab.com/tcp-ip/">TCP/IPとは？初心者がTCP/IPを理解するための完全ガイド</a><br>
ブログ 「LIFESTYLE LAB（ライフスタイルラボ）」

><a href="https://qiita.com/naoki_mochizuki/items/7ee0e01db61e1e7abd62">[インターネット通信の流れ]</a><br>
Qiita記事 @naoki_mochizuki
