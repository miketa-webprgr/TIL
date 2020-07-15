# Issue03 投稿のCRUD機能実装

## 求められている機能実装について

1. kaminariを使って、投稿のページネーションを実装する
2. ページネーションにもbootstrapを適用する
3. 1ページあたり15件とする

## 分からない単語・概念等の一覧

- kaminari

### kaminariについて

シンプルに実装できる。  

- gemの導入
- rails g kaminari:views bootstrap4（自動でslimに変換されるはず）
  - [公式Github: kaminari（#for-hamlslim-users)](https://github.com/kaminari/kaminari#for-hamlslim-users)
  - [公式Github: kaminari（#theme)](https://github.com/kaminari/kaminari#themes)
- `config/initializers/kaminari_config.rb`の設定
  - このあたりの設定については公式やブログで詳しく解説されている
- コントローラとビューにて設定を追加
  - ビューにはページネーションだけでなく、「何件中の何件表示」と示すこともできる
  - ビュー関係のヘルパーは公式に詳しく掲載されている
    - [公式Github: kaminari（#helpers)](https://github.com/kaminari/kaminari#helpers)
- 国際化の場合はここを参照
  - [公式Github: kaminari（#i18n-and-labels)](https://github.com/kaminari/kaminari#i18n-and-labels)

### kaminariの非同期通信

- 非同期で実装する場合は`remote: true`とすること
  - [公式Github: kaminari（#ajax)](https://github.com/kaminari/kaminari#ajax-links-crazy-simple-but-works-perfectly)
- その場合、JSを書く必要がある
  - 少しハードルが高そうなので、１時間強ほど試行錯誤した後に今回は見送った

## 参考

> -[公式Github: kaminari](https://github.com/kaminari/kaminari)  
> -[【Rails】kaminariを使ってページネーションを実装する方法 \- Qiita](https://qiita.com/tomo_k09/items/b9242b6795f867a1844f)  
> -[Railsライブラリ紹介: ページングを行う「kaminari」 \| TECHSCORE BLOG](https://www.techscore.com/blog/2013/01/07/rails%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E7%B4%B9%E4%BB%8B-%E3%83%9A%E3%83%BC%E3%82%B8%E3%83%B3%E3%82%B0%E3%82%92%E8%A1%8C%E3%81%86%E3%80%8Ckaminari%E3%80%8D/)
> -[【Rails】kaminariを使用してページネーション機能を実装 \- Qiita](https://qiita.com/ryota21/items/29fa282745afb1474059)
> -[kaminari徹底入門 \- Qiita](https://qiita.com/nysalor/items/77b9d6bc5baa41ea01f3)  

## 動作確認方法

1. `git clone https://github.com/miketa-webprgr/instagram_clone.git`
2. `git checkout git checkout -b feature/01_login_logout origin/feature/01_login_logout`
3. `bundle install`
4. `yarn install`
5. MySQL と Redis を立ち上げる
6. `rails db:migrate`
7. `rails db:seed`
