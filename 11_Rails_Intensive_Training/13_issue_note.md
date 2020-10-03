# Issue13 通知の設定を実装

## どんな感じ？

通知をするかしないかをユーザーが設定できる機能を実装する。  
なお、画面のイメージは以下のとおり。  

<a href="https://gyazo.com/09006c3070ba5ace05cf563386dd6047"><img src="https://i.gyazo.com/e5c3a7136eaf2dc61f154bc66f9b00aa.png" alt="Image from Gyazo" width="500" border=1/></a><br>  

## 求められている機能実装・実装条件について

マイページに通知設定というメニューを追加する。  
設定画面から、以下のオンオフを切り替えられるようにする。  

- コメント時の通知メール
- いいね時の通知メール
- フォロー時の通知メール

## 分からない単語・概念等の一覧

特になし。  

これまでのメール通知機能実装のロジックはシンプルなものだったので、  
そこのロジックをきちんと書いてもらうのがこのIssueの目的。（と思われる）  

## 実装手順

ある程度実力もつき、コードリーディングせずにいけそうなので、挑戦してみる。  
まず、大まかな実装方針から考える。  

1. ルーティングを設定する（edit, updateのみ）（デフォルトも設定する）
2. メニューのパーシャルのビューを作る。（通知一覧の下に通知設定を追加する）
3. セレクトフォームとボタンを追加。CSSフレームワークをあてる。
4. usersテーブルにカラムを追加
5. コントローラでeditとupdateアクションを実装する
6. メール送信のロジックに変更を加える

## 実装１（ルーティングについて）

ユーザーの管理画面で使う機能になるので、namespace配下に設定する。  
ユーザーはメール通知に関する設定を2個も3個もするわけではないので、resourceと単数形になる。  
（ユーザーは複数の設定リソースを作って、好きなように切り替えられる場合は別だけど）  

```rb
  namespace :mypage do
    resource :account, only: %i[edit update]
    resources :notifications, only: %i[index]
    resource :notification_setting, only: %i[edit update]
  end
```

## 実装２（メニューのパーシャルのビューを作る）

これは、既に作成したものをベースにして作れば簡単。  

```slim
nav
  ul.list-unstyled
    li
      = link_to 'プロフィール編集', edit_mypage_account_path
      hr
    li
      = link_to '通知一覧', mypage_notifications_path
      hr
    / 今回追加したもの
    li
      = link_to '通知設定', edit_mypage_notification_setting_path
      hr
```

なお、とりあえず仮のmypage/edit.html.slimを作成しておく。  
すると、ルーティングやコントローラが設定できていれば、このような形で表示できる。  

<a href="https://gyazo.com/09006c3070ba5ace05cf563386dd6047"><img src="https://i.gyazo.com/c53acc0c3bbd23ddeb43528fc8c189a4.png" alt="Image from Gyazo" width="500" border=1/></a><br>  

少しつまづいたのが、コントローラに関する設定。  
以下のとおりとなる。  

```rb
# Mypage::BaseControllerを引き継ぐことで、共通のlayoutとrequired_loginを継承できる
# Mypage配下にコントローラを置いているため、Mypage::NotificationSettingsControllerと書く
class Mypage::NotificationSettingsController < Mypage::BaseController
  def edit
  end

  def update
  end
end
```

信頼度はよく分からないが、以下のようなQiita記事があった。  

> 名前空間の（トップレベルからのパス）絶対パスで指定する場合に：：ダブルコロンが使われる。
>
> [rubyにおける：：（ダブルコロン、コロンコロン）についてのメモ \- Qiita](https://qiita.com/hatorijobs/items/87a2bd93f8666d77d711)

## 実装３（メール通知設定フォームのビュー作成）

今回は、いままでおざなりにしてきたビューについても力をいれてみる。  
BMDをCSSフレームワークとして活用しているので、以下を参照する。  

- [Forms · Bootstrap Material Design](https://fezvrasta.github.io/bootstrap-material-design/docs/4.0/material-design/forms/)

おそらく、チェックボックスを使っているので、HTMLはこんな感じになるはず。（実際はだいぶ違いますが）  
なお、上記サイトからイメージで書いているだけなので、slimで書くとまた異なる。  

考える上で、改めてこれにお世話になった。  

- [RUNTEQの講師をやってみてわかった初学者にありがちなパターン20選（後編） \- Qiita](https://qiita.com/DaichiSaito/items/cd66115569b0a75f1bfa)

```html
<form>
  <div class="checkbox">
    <label>
      <input type="checkbox" name="user[notification_on_comment]" value="true"> コメントがあった時に通知する
    </label>
  </div>
  <div class="checkbox">
    <label>
      <input type="checkbox" name="user[notification_on_like]" value="true"> いいねがあった時に通知する
    </label>
  </div>
  <div class="checkbox">
    <label>
      <input type="checkbox" name="user[notification_on_follow]" value="true"> フォローされた時に通知する
    </label>
  </div>
  <button class="btn btn-default">Cancel</button>
  <button type="submit" class="btn btn-primary btn-raised">Submit</button>
</form>
```

これをRailsの`form_with`を使うと、こんな感じ。  
なお、エラーメッセージも表示させるので、以下のとおりとなる。  

```slim
= form_with model: @user, url: mypage_notification_setting_path, method: :patch, local: true do |f|
  = render 'shared/error_messages', object: @user
  .form-group
    .checkbox 
      = f.check_box :notification_on_comment
      = f.label :notification_on_comment
    .checkbox 
      = f.check_box :notification_on_like
      = f.label :notification_on_like
    .checkbox 
      = f.check_box :notification_on_follow
      = f.label :notification_on_follow
  = f.submit class: 'btn btn-primary btn-raised'
```

HTMLとしては、以下のとおり出力される。  

```html
<form action="/mypage/notification_setting" accept-charset="UTF-8" method="post">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input type="hidden" name="_method" value="patch" />
  <input type="hidden" name="authenticity_token" value="h8PX/1JSHT3aslqZTmQPlGkU3GSuAJmQOW6IXmq0ACYlM/uaiAO6z7JtD6SZm+gr9DzufoKh22IaVF2FlCPulg==" />
  <div class="form-group">
    <div class="checkbox">
      <input name="user[notification_on_comment]" type="hidden" value="0" />
      <input type="checkbox" value="1" checked="checked" name="user[notification_on_comment]" id="user_notification_on_comment" />
      <label for="user_notification_on_comment">Notification on comment</label>
    </div>
    <div class="checkbox">
      <input name="user[notification_on_like]" type="hidden" value="0" />
      <input type="checkbox" value="1" checked="checked" name="user[notification_on_like]" id="user_notification_on_like" />
      <label for="user_notification_on_like">Notification on like</label>
    </div>
    <div class="checkbox">
      <input name="user[notification_on_follow]" type="hidden" value="0" />
      <input type="checkbox" value="1" checked="checked" name="user[notification_on_follow]" id="user_notification_on_follow" />
      <label for="user_notification_on_follow">Notification on follow</label>
    </div>
  </div>
  <input type="submit" name="commit" value="更新する" class="btn btn-primary btn-raised" data-disable-with="更新する" />
</form>
```

## 実装4（usersテーブルにカラムを追加）

`form_with`を使う場合、きちんとモデルの設定をしてあげる必要がある。  

まだusersテーブルに`notification_on_commment`などのカラムがないので、  
マイグレーションファイルを作成し、テーブル構造に変更を加える。  

```rb
# usersテーブルにメール通知設定に関するカラムを3つ追加するマイグレーションファイル

class AddNotificationFlagsTousers < ActiveRecord::Migration[5.2]
  def change
    add_column :users, :notification_on_comment, :boolean, default: true
    add_column :users, :notification_on_like, :boolean, default: true
    add_column :users, :notification_on_follow, :boolean, default: true
  end
end
```

そして、インスタンス変数を引き渡してあげないと、エラーメッセージを出すパーシャルにおいて例外処理となってしまうので、  
取り急ぎコントローラに以下を記載しておく。  

```rb
# mypage/notification_settings_controller.rb

class Mypage::NotificationSettingsController < Mypage::BaseController
  def edit
    @user = User.find(current_user.id)
  end

  def update
  end
end
```

すると、以下のとおりとなった。  

<a href="https://gyazo.com/09006c3070ba5ace05cf563386dd6047"><img src="https://i.gyazo.com/a9cce6f3bf8f658f6e7c5848ec40502b.png" alt="Image from Gyazo" width="500" border=1/></a><br>  

また、日本語にするため、以下を追加する。  

```yml
notification_on_comment: 'コメントがあった時に通知する'
notification_on_like: 'いいねされた時に通知する'
notification_on_follow: 'フォローされた時に通知する'
```

以下のとおりとなった。  

ちなみに、実際の作業においては、この段階でBMDを導入した。  
不要な気がするが、BMDのクラスを適用したために文字が半透明になった。

<a href="https://gyazo.com/09006c3070ba5ace05cf563386dd6047"><img src="https://i.gyazo.com/08c66a511091d974219eaa474fe404f8.png" alt="Image from Gyazo" width="500" border=1/></a><br>  

## 実装５（メール送信のロジックに変更を加える）

`notification_settings_controller.rb`にて、editアクションとupdateアクションを実装する。  
特に難しい実装はなく、一般的なCRUD実装と同じように行う。  

```rb
class Mypage::NotificationSettingsController < Mypage::BaseController
  def edit
    @user = User.find(current_user.id)
  end

  def update
    @user = User.find(current_user.id)
    if @user.update(notification_settings_params)
      redirect_to edit_mypage_notification_setting_path, success: 'メール通知設定を更新しました'
    else
      flash.now['danger'] = 'メール通知設定の更新に失敗しました'
      render :edit
    end
  end

  private

  def notification_settings_params
    params.require(:user).permit(:notification_on_comment, :notification_on_like, :notification_on_follow)
  end
end
```

## 実装６（メール送信のロジックに変更を加える）

メール送信の判定ロジックを追加し、送信設定がオフの場合、メールを送信しないようにする。  

アンチパターンではあるが、だいさんさんの実装と異なってコールバックを活用しているため、  
コールバックの発動に条件をつける。  

- [Active Record コールバック \- Railsガイド](https://railsguides.jp/active_record_callbacks.html#%E6%9D%A1%E4%BB%B6%E4%BB%98%E3%81%8D%E3%82%B3%E3%83%BC%E3%83%AB%E3%83%90%E3%83%83%E3%82%AF)

ダックタイピングを活用しているため、Notifiableモジュールと関連先モデルにてロジックを実装する。  
具体的には、以下のとおりの実装となる。  

```rb
# Comment, Like, Relationshipモデルで以下のメソッドを使用する
module Notifiable
  # コールバックなどを使うために必要
  # https://stackoverflow.com/questions/7444522/is-it-possible-to-define-a-before-save-callback-in-a-module
  extend ActiveSupport::Concern

  # URLヘルパーを使うために導入
  include Rails.application.routes.url_helpers

  included do
    has_one :notification, as: :notifiable, dependent: :destroy
    # after_create_commitは、after_commitのエイリアスメソッド
    # after_saveというメソッドもあるが、こちらはDBにsaveする直前に発火するメソッド
    # DBの制約に抵触して保存できない場合も考慮して、after_create_commitとする
    after_create_commit :create_notifications
    after_create_commit :send_notification_mail, if: :send_mail?
  end

  def partial_name
    raise NotImplementedError
  end

  def resource_path
    raise NotImplementedError
  end

  private

  def create_notifications
    raise NotImplementedError
  end

  def send_notification_mail
    raise NotImplementedError
  end

  def send_mail?
    raise NotImplementedError
  end
end
```

```rb
# Comment.rbに以下のメソッドを追加

  # ダックタイピングのため、overrideする
  def send_mail?
    user.notification_on_comment
  end
```

```rb
# Like.rbに以下のメソッドを追加

  # ダックタイピングのため、overrideする
  def send_mail?
    user.notification_on_like
  end
```

```rb
# Relationship.rbに以下のメソッドを追加

  # ダックタイピングのため、overrideする
  def send_mail?
    followed.notification_on_follow
  end
```
