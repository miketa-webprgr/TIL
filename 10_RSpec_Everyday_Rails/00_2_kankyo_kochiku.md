# Rails6系で「everyday Rails」（RSpec）を勉強したいので、伊藤さんの記事を参考にして頑張ってみた（part2）

## はじめに

この記事は、以下のQiita記事の続編です。
> Rails6系で「everyday Rails」（RSpec）を勉強したいので、伊藤さんの記事を参考にして頑張ってみた（part2）

伊藤さんがアップロードされたyoutube動画を見ながら、私個人が
どのように対応していったか記しているだけなので、動画を見れば大体事足ります。  

> [【動画付き】Everyday RailsのサンプルアプリをRails 6で動かす際に必要なテストコードの変更点 \- give IT a try](https://blog.jnito.com/entry/2019/10/25/053521)
> [永久保存版！？伊藤さん式・Railsアプリのアップグレード手順 \- Qiita](https://qiita.com/jnchito/items/0ee47108972a0e302caf)  
> [【前編】永久保存版！？伊藤さん式・Railsアプリのアップグレード手順](https://youtube.com/watch?v=hT68fhuWbHU)
> [【後編】永久保存版！？伊藤さん式・Railsアプリのアップグレード手順](https://youtube.com/watch?v=SnwNFMauzjM)

なので、あまりこの記事の意義はないのですが、文字の方が探しやすい  
こともあるかと思うので、参考までに活用いただければと思います。

## 4-c. トラブルが起きやすそうなgemを1つずつアップデートする

伊藤さんによると、次は`bundle outdated | bof --format markdown`の実行によって得られた、
defaultと書いてある`gem`を見ていくべきとのことだが、そもそもdefaultと書かれているgemが存在しない。  

参考までに、伊藤さんの動画のスクショと、私が得られたdefaultと書かれていないgemの一覧を貼付しておく。  
勝手な推測だが、これも伊藤さんが動画を撮った時期と、私がアップデートしている時期が異なるためだと思われる。  

<a href="https://gyazo.com/bafa6a3ee68e7ed93ed0dcb3e0796f54"><img src="https://i.gyazo.com/bafa6a3ee68e7ed93ed0dcb3e0796f54.png" alt="Image from Gyazo" width="500" border=1/></a>  


| gem | newest | installed | requested | groups |
| --- | --- | --- | --- | --- |
| actioncable | 6.0.3.1 | 5.1.7 | | |
| actionmailer | 6.0.3.1 | 5.1.7 | | |
| actionpack | 6.0.3.1 | 5.1.7 | | |
| actionview | 6.0.3.1 | 5.1.7 | | |
| activejob | 6.0.3.1 | 5.1.7 | | |
| activemodel | 6.0.3.1 | 5.1.7 | | |
| activerecord | 6.0.3.1 | 5.1.7 | | |
| activesupport | 6.0.3.1 | 5.1.7 | | |
| arel | 9.0.0 | 8.0.0 | | |
| autoprefixer-rails | 9.7.6 | 6.7.7.2 | | |
| bcrypt | 3.1.13 | 3.1.12 | | |
| bindex | 0.8.1 | 0.5.0 | | |
| bootstrap-sass | 3.4.1 | 3.3.7 | | |
| cocaine | 0.6.0 | 0.5.8 | | |
| coffee-rails | 5.0.0 | 4.2.1 | ~> 4.2 | |
| devise | 4.7.2 | 4.4.3 | | |
| faker | 2.12.0 | 1.7.3 | | |
| geocoder | 1.6.3 | 1.4.9 | | |
| i18n | 1.8.3 | 0.9.5 | | |
| jbuilder | 2.10.0 | 2.6.3 | ~> 2.5 | |
| jquery-rails | 4.4.0 | 4.3.1 | | |
| listen | 3.2.1 | 3.1.5 | < 3.2, >= 3.0.5 | |
| mimemagic | 0.3.5 | 0.3.2 | | |
| mini_portile2 | 2.5.0 | 2.4.0 | | |
| multi_json | 1.14.1 | 1.12.1 | | |
| paperclip | 6.1.0 | 5.1.0 | | |
| puma | 4.3.5 | 3.8.2 | ~> 3.7 | |
| rails | 6.0.3.1 | 5.1.7 | ~> 5.1.1 | |
| railties | 6.0.3.1 | 5.1.7 | | |
| rb-fsevent | 0.10.4 | 0.9.8 | | |
| rb-inotify | 0.10.1 | 0.9.8 | | |
| responders | 3.0.1 | 2.4.0 | | |
| sass | 3.7.4 | 3.4.23 | | |
| sass-rails | 6.0.0 | 5.0.6 | ~> 5.0 | |
| spring | 2.1.0 | 2.0.1 | | |
| sprockets | 4.0.2 | 3.7.2 | | |
| sqlite3 | 1.4.2 | 1.3.13 | | |
| thor | 1.0.1 | 0.20.3 | | |
| tilt | 2.0.10 | 2.0.7 | | |
| turbolinks | 5.2.1 | 5.0.1 | ~> 5 | |
| turbolinks-source | 5.2.0 | 5.0.0 | | |
| tzinfo | 2.0.2 | 1.2.7 | | |
| uglifier | 4.2.0 | 3.2.0 | | |
| warden | 1.2.8 | 1.2.7 | | |
| web-console | 4.0.3 | 3.5.0 | | |
| websocket-driver | 0.7.2 | 0.6.5 | | |

この時点でyoutube動画と条件も違うのだが、ここは伊藤さんの動画でやっているとおり、  
雑に`bundle update`してみることにする。

けど、正直トラブルになりそうで、とても怖い。。。
とりあえず、後で戻せるようにコミットだけしておく。

`bundle update`は成功し、RSpecでのテストを実行する。  

```
Finished in 25.46 seconds (files took 1.14 seconds to load)
70 examples, 0 failures
```

無事、成功した！！
ただ、動画で説明のとおり、以下のような警告が出ている。  

```
Sign-ups
DEPRECATION WARNING: [Devise] `DeviseHelper.devise_error_messages!`
is deprecated and it will be removed in the next major version.
To customize the errors styles please run `rails g devise:views` and modify the
`devise/shared/error_messages` partial.
```

ここで、警告に書かれているとおり、 `rails g devise:views`を実行する。  

実行すると、オーバーライドするか尋ねられる。  

最初は`n`として、次のファイルにて`d`を押して、差分を確認して以下をコピーする。
他のファイルについては、全て`n`とする。

```
-   <%= devise_error_messages! %>
+   <%= render "devise/shared/error_messages", resource: resource %>
```

コピーした内容を使って、該当部分だけ修正する。  

`<%= devise_error_messages! %>`を全て修正する。
修正後は、`<%= render "devise/shared/error_messages", resource: resource %>`になる。  

該当ファイルは以下のとおり。

<a href="https://gyazo.com/3c14c8690a9f5fa4926e72e85ec57ba6"><img src="https://i.gyazo.com/3c14c8690a9f5fa4926e72e85ec57ba6.png" alt="Image from Gyazo" width="400" border=1/></a>  

修正後にRSpecにてテストを行うと、警告が消えた。  

bundle outdated を行うと、以下のとおりとなった。  

| gem | newest | installed | requested | groups |
| --- | --- | --- | --- | --- |
| actioncable | 6.0.3.1 | 5.1.7 | | |
| actionmailer | 6.0.3.1 | 5.1.7 | | |
| actionpack | 6.0.3.1 | 5.1.7 | | |
| actionview | 6.0.3.1 | 5.1.7 | | |
| activejob | 6.0.3.1 | 5.1.7 | | |
| activemodel | 6.0.3.1 | 5.1.7 | | |
| activerecord | 6.0.3.1 | 5.1.7 | | |
| activesupport | 6.0.3.1 | 5.1.7 | | |
| arel | 9.0.0 | 8.0.0 | | |
| coffee-rails | 5.0.0 | 4.2.2 | ~> 4.2 | |
| listen | 3.2.1 | 3.1.5 | < 3.2, >= 3.0.5 | |
| mini_portile2 | 2.5.0 | 2.4.0 | | |
| puma | 4.3.5 | 3.12.6 | ~> 3.7 | |
| rails | 6.0.3.1 | 5.1.7 | ~> 5.1.1 | |
| railties | 6.0.3.1 | 5.1.7 | | |
| sass-rails | 6.0.0 | 5.0.7 | ~> 5.0 | |
| sprockets | 4.0.2 | 3.7.2 | | |
| thor | 1.0.1 | 0.20.3 | | |
| tzinfo | 2.0.2 | 1.2.7 | | |
| web-console | 4.0.3 | 3.7.0 | | |
| websocket-driver | 0.7.2 | 0.6.5 | | |

伊藤さんの動画だと、以下のとおりとなっている。  

<a href="https://gyazo.com/d7730fcd7c6ce2b20168b1ceeaaa9f7c"><img src="https://i.gyazo.com/d7730fcd7c6ce2b20168b1ceeaaa9f7c.png" alt="Image from Gyazo" width="500" border=1/></a>  

細かくは確認できていないが、おそらく同じ状態になったと思われる。  

指示に従い、Rubyのバージョンを新しくする。  
現在の最新のバージョン（安定版）は2.7.1とのことだが、ちょっと試すのも怖い。  
（動画では2.6.5が採用されている）  

とはいえ、伊藤さんによるとRubyは後方互換性を意識しているとのことなので、  
commit だけ一応しておいて、バージョン2.7.1で試してみる。  

```
# ruby 2.7.1 をダウンロードする
rbenv install 2.4.9

# rubyのバージョンを 2.7.1 に指定する
rbenv local 2.7.1

# rubyのバージョンが 2.7.1 になっていることを確認する
rbenv version
```
