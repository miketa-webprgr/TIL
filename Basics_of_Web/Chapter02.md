# はじめに

みけた＠プログラマー転職です。<br>
元公務員、無職の31歳です！<br>

転職に向けて、まずWebの基礎について理解したいと思い、下記の書籍を読んでいくことにしました。<br>
間違いがあるかもしれませんが、この書籍を読まれる初学者の方にとって参考になれば幸いです。<br>

><a href="https://www.amazon.co.jp/%E3%82%A4%E3%83%A9%E3%82%B9%E3%83%88%E5%9B%B3%E8%A7%A3%E5%BC%8F-%E3%81%93%E3%81%AE%E4%B8%80%E5%86%8A%E3%81%A7%E5%85%A8%E9%83%A8%E3%82%8F%E3%81%8B%E3%82%8BWeb%E6%8A%80%E8%A1%93%E3%81%AE%E5%9F%BA%E6%9C%AC-%E5%B0%8F%E6%9E%97-%E6%81%AD%E5%B9%B3-ebook/dp/B06XNMMC9S">イラスト図解式 この一冊で全部わかるWeb技術の基本</a><br>
小林 恭平  (著), 坂本 陽  (著), 佐々木 拓郎  (監修)<br>
※ Amazonで、Kindle版が1000円弱で買えます

# Chapter ２ - Webとネットワーク

### ○ Webの仕組みついてのおさらい

改めて、前回のまとめで貼った画像を貼っておきます。
<img width="700" alt="WebArchitecture" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/429155/3c364d67-b219-b041-1d49-1f4ff6edf35f.jpeg">  

以上の図のとおり、自宅のPCのChromeやIEなどにWebページが表示されるのは、<br>
ChromeなどのWebブラウザからWebサーバーに対して「〜ページを表示して！」というリクエストが送られ、<br>
それを受けたWebサーバーが該当のファイルを送り返しているからである。<br>

そのやり取りを行うにあたって使われているのが、HTTP（HyperText Transfer Protocol)という<br>
規格や、より安全なHTTPSという規格である。<br>

「〜ページを表示して！」というリクエストの行き先は、統一資源位置指定子！！！（URLのことです。。。）に<br>
よって特定され、IPアドレスという番号の羅列に変換されて、初めてPCが分かる行き先になる。<br>

Webサーバーから送られるファイルは、サーバー内のサーバーサイド言語（Ruby・PHP・Python）などによって、<br>
その都度生成されることがあり、これを「動的ページ」という。<br>

ちなみに、FacebookやInstagramがユーザーごとにカスタマイズしたページを表示できるのは、<br>
Webブラウザから受け取った情報を元にして、動的にWebページを生成しているからである。<br>

### ○ DNSについて

先ほどの説明だと、単純にWebページの表示は自宅のPCとWebサーバーのみで完結しているように思われる。<br>
だが、実際には、IPアドレスを見つける為に、その間にはDNSサーバーが介在している。<br>

単純に自宅のPCがダイレクトにWebサーバーに行けるわけではなく、IPアドレス取得のため、<br>
DNSサーバーへのスタンプラリーを終えなければいけない。<br>

<img width="700" alt="WebArchitecture" src="https://www.nic.ad.jp/ja/dom/img/basics_b.gif"> <br>

これがそのスタンプラリーの様子だ。<br>
会社の稟議のように、DNSルートサーバーを始めとして、多くのDNSサーバーを媒介しなければならない。<br>

ところで、DNSとはDomain Network Systemの略称である。<br>
要は、場所を特定するためのネットワークシステムということである。<br>

文字どおり、場所特定のためのスーパーコンピュータがあるわけではなく、<br>
複数のコンピュータが繋がった形で、それぞれが協力してIPアドレスを特定しているのである。<br>

><a href="https://www.nic.ad.jp/ja/dom/system.html">ドメイン名のしくみ</a><br>
一般社団法人日本ネットワークインフォメーションセンターHP

### ○ TCP/IPについて

TCP/IPとは、インターネット上のプロトコルの総称を指す。<br>
例えば、以下のようなプロトコルがある。<br>
- HTTP（HTMLのプロトコル）
- FTP（Fileのプロトコル）
- SMTP（メール送信のプロトコル）
- POP（メール受信のプロトコル）<br>

ちなみに、TCP/IPはいくつものレイヤー（層）に分かれている。<br>
HTTPやFTPはアプリケーションというレイヤーに属している。<br>

><a href="https://tomslifestylelab.com/tcp-ip/">TCP/IPとは？初心者がTCP/IPを理解するための完全ガイド</a><br>
ブログ 「LIFESTYLE LAB（ライフスタイルラボ）」

><a href="https://qiita.com/naoki_mochizuki/items/7ee0e01db61e1e7abd62">[インターネット通信の流れ]</a><br>
Qiita記事 @naoki_mochizuki
