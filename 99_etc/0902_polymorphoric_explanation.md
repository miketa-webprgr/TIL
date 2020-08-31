# ポリモーフィック関連付けをして、createアクションをする方法について考えてみた

Railsのポリモーフィック関連付けの機能を使うと、Modelの操作が簡単になり、  
親オブジェクトや子供オブジェクトの取得が簡単にできるようになります。  

おお、これは便利だと思ってポリモーフィック関連付けをしたモデルを活用した練習アプリを  
作ってみたのですが、意外と考えることが多かったので、その思考の過程を頑張ってまとめてみました。  

## ポリモーフィック関連付けってどうやるか分かっていますか？

その前にアソシエーションって分かっていますか？  

- １対１や１対多のアソシエーションの設定方法
  - [Active Record の関連付け \- Railsガイド](https://railsguides.jp/association_basics.html#belongs-to%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)
- １対１や１対多のアソシエーションを使って、親オブジェクトに紐づく子供オブジェクトを取得する方法
  - `@user.avatar`や`@user.build_avatar`
  - `@user.tweets`や`@user.tweets.build`

じゃあ、ポリモーフィック関連ってどうやるか分かってますか？  

- ポリモーフィック関連のアソシエーションの設定方法
  - [ポリモーフィック関連付け \- Railsガイド](https://railsguides.jp/association_basics.html#%E3%83%9D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)
- ポリモーフィックを使って、親オブジェクトに紐づく子供オブジェクトを取得する方法
  - `@player.tags`や`@player.tags.build`は同じ
  - `@tag.taggable`で子から親が取得できる

## ポリモーフィック関連付けを使ってアプリを作ってみよう！

親とか子供とか書いていると分かりづらいので、アプリに即して説明します。  

Playerモデル（例：メッシ選手）とManagerモデル（例：グアルディオラ監督）とTagモデル（例：スピードスター）の３つがあるとします。  

ここで、まず直面したコントローラの問題について解説します。  
まず、ベーシックなscaffoldのコントローラのコードを貼っておきます。  

みなさん、Tagコントローラもこんな感じのままで大丈夫だと思いますか？  
どう考えても面倒くさいアクションがありませんか？  

```rb
class ManagersController < ApplicationController
  before_action :set_manager, only: [:show, :edit, :update, :destroy]

  # GET /managers
  def index
    @managers = Manager.all
  end

  # GET /managers/1
  def show
  end

  # GET /managers/new
  def new
    @manager = Manager.new
  end

  # GET /managers/1/edit
  def edit
  end

  # POST /managers
  def create
    @manager = Manager.new(manager_params)

    if @manager.save
      redirect_to @manager, notice: 'Manager was successfully created.'
    else
      render :new
    end
  end

  # PATCH/PUT /managers/1
  def update
    if @manager.update(manager_params)
      redirect_to @manager, notice: 'Manager was successfully updated.'
    else
      render :edit
    end
  end

  # DELETE /managers/1
  def destroy
    @manager.destroy
    redirect_to managers_url, notice: 'Manager was successfully destroyed.'
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_manager
      @manager = Manager.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def manager_params
      params.require(:manager).permit(:name, :team)
    end
end
```

そう、createアクションです。  

これはアソシエーションをしているときも似たような問題を抱えるのですが、紐付けさせることで操作が簡単になるのはいいんですが、  
紐付け先の情報を情報を保存しなくちゃいけないがゆえに、createするときはその分面倒くさいんです！！！  

## １対多の場合のcreateアクションを考えてみる

じゃあ、段階を追って、まず１対多のアソシエーションをやっている時、どんな感じでcreateアクションを書いているか確認してみましょう。  

```rb
  def create
    @post = @user.posts.build(post_params)
    if @post.save
      redirect_to posts_path, success: '投稿しました'
    else
      flash.now[:danger] = '投稿に失敗しました'
      render :new
    end
  end

  private

    def post_params
      params.require(:post).permit(:body, :image)
    end
```

本当に適当にコードを引っ張ってきたので、前提条件が分からず混乱したかもしれませんが、とりあえずUserモデルとPostモデルがあって、  
usersはpostsを投稿できると思ってください。  

アソシエーションを使って、紐づくUserという親がいるPostオブジェクトをcreateする場合には、`@user.posts.build`もしくは、  
`@user.posts.new`という形で新しいオブジェクトを生成させて、そこにparamsをガツンとブチ込む！ということをとりあえず覚えておいてください。  

## ポリモーフィックな関連付けの場合、どうすればよいのか基本に立ち返って考える

じゃあ、いよいよポリモーフィック関連に取り組みましょう。  
ここで、難易度が上がるんです。  
  
Playerモデル（例：メッシ選手）とManagerモデル（例：グアルディオラ監督）とTagモデル（例：スピードスター）の話に戻りますが、  
ポリモーフィック関連を使うと、紐づく先の親が何になるか分からないんです！  

```rb
  def create
    # パターン１
    @tag = @player.tags.build(tags_params)
    # パターン２
    @tag = @manager.tags.build(tags_params)
```

シンプルに考えると、分岐しちゃえば大丈夫ですよね。  

ここで、分岐ポイントを洗い出すために、クライアント側からのリクエストからサーバーを経由して、  
ビューを表示させてあげるまでの流れを確認しましょう。  

1. クライアントがタグの内容をフォームに記入する
2. Submitボタン的なものを押して、サーバーに送信する
3. コントローラでparameterを処理して、データベースに保存し、ビューを返す

## コントローラを分けて、createアクションを書いてしまえばよい！

では、順を追って対応方法を考えてみましょう。  

まず、`players/tag_controllers.rb`と`managers/tag_controller.rb`を作成してしまい、  
クライアント側のリクエスト先を適切に分けてあげればよさそうです。  

Playerに関するタグを作成したい場合、Submitボタン的なものを押した時、  
player/tag_controller.rbで処理させましょう。  

そのため、ルーティングを適切に設定してください。  
結論からいいますが、こうしてください。  

```rb
resources :players do
  resources :tags, module: :players
end

resources :managers do
  resources :tags, module: :managers
end
```

単純にネストするのではダメなのかって。  
私もそう思って、とりあえずやってみました。  

すると、こうなるんです。  
長すぎてしんどいので、createアクション以外は消しちゃいました。  

```text
       Prefix   Verb   URI Pattern                             Controller#Action
  player_tags   POST   /players/:player_id/tags(.:format)      tags#create
      players   POST   /players(.:format)                      players#create
 manager_tags   POST   /managers/:manager_id/tags(.:format)    tags#create
     managers   POST   /managers(.:format)                     managers#create
```

分かるとおり、controller#actionの中身が重複しちゃうんです。  

え、それって何なの？
そもそもController#Actionが指し示しているものって何だっけ？

って感じに私もなりましたが、これは参照先のアクションを指し示しています。  

これだと、tags#createでplayerに紐づくtagも、managerに紐づくtagも処理しなければいけないので、  
せっかくコントローラを分けて作っても全く意味がありません（利用されるのは`tags_controller.rb`だけ）。  

コントローラを分けて使ってもらうためにも、以下のとおり設定する必要があります。  

```text
       Prefix   Verb   URI Pattern                             Controller#Action
  player_tags   POST   /players/:player_id/tags(.:format)      players/tags#create
      players   POST   /players(.:format)                      players#create
 manager_tags   POST   /managers/:manager_id/tags(.:format)    managers/tags#create
     managers   POST   /managers(.:format)                     managers#create
```

ここまで来れば、もう簡単ですね。  
場合の切り分けがうまくいったので、１対多のアソシエーションと同じです。  

```rb
# players/tags_controller.rb

  def create
    @tag = @player.tags.build(tags_params)
    # 他は省略
  end

  private

    def tags_params
      params.require(:tag).permit(:body)
    end
```

```rb
# managers/tags_controller.rb

  def create
    @tag = @manager.tags.build(tags_params)
    # 他は省略
  end

  private

    def tags_params
      params.require(:tag).permit(:body)
    end
```

ここから更にDRYに書きたい人は、共通のアクションは親コントーラにまとめてしまい、  
分岐させなくちゃいけないものだけ、子供コントーラに書くという方法があります。  

このあたりの方法については、以下のGitHubに参考例があります。  

- [gorails\-episode\-36/app/controllers at master · gorails\-screencasts/gorails\-episode\-36](https://github.com/gorails-screencasts/gorails-episode-36/tree/master/app/controllers)

## コントローラを分けないでやる方法も考えてみる

まあ、先ほど紹介した方法が一番よいと思われるので、これで話を終えた方がいいような気がするのですが、  
「コントローラを何個も作るとかだるくない？」と思っている方に向けて、他の方法論も紹介します。  

紹介していく中で、きっと先ほど紹介した方法が一番良さそうというのが分かっていくと思うので笑  

先ほどの方法論においては、フォームのparamsの送信先、つまりクライアント側のリクエスト先を適切に分けてあげたのですが、  
今回はそういうことはしたくないので、「リクエスト先は`tags_controller.rb`の一択、`tags_controller.rb`のファイルは分けない！」  
という前提でいきましょう。  

すると、紐付け先の情報を何らかの方法で取得し、そこからtagsテーブルに必要になってくる`taggable_type`と`taggable_id`を埋める  
ということをやらなければならないのですが、私がGoogle検索に検索を重ねて、無い脳味噌を必死に振り絞った結果、以下の２つの解決策を思いつきました。  

1. フォームから`taggable_type`と`taggable_id`を送ってあげる！
2. クライアントのリクエスト先のURLを活用して、`taggable_type`と`taggable_id`を取得する。

## フォームから`taggable_type`と`taggable_id`を送ってやる方法

PlayerモデルとManagerモデルとTagモデルの話にまた戻りますが、今回は@playerの場合と@managerの場合と切り分けることができないので、  
`tags_params`に`taggable_type`と`taggable_id`もparamsに含めて、あげることが必要になります。  

```rb
# これは今回やれないという条件が課せられている

  def create
    # パターン１
    @tag = @player.tags.build(tags_params)
    # パターン２
    @tag = @manager.tags.build(tags_params)
    # 他は省略
  end

  private

    def tags_params
      params.require(:tag).permit(:body)
    end
```

```rb
# そこでこうしないといけない
# @somethingとしましたが、@taggableとするのが慣例のようです
# ツッコミどころがあるんですが、イメージ重視であえてこうしてます

  def create
    @tag = @something.tags.build(tags_params)
    # 他は省略
  end

  private

    def tags_params
      params.require(:tag).permit(:body, :taggale_type, :taggable_id)
    end
```

## そもそも、フォームからparamsってどうやって送っているんだっけ？

まず、段階を追って理解していくために、`players/tags_controller.rb`と  
`managers/tags_controller.rb`に分けて対応するパターンについて確認します。  

これは、先ほども書きましたが、１対多のアソシエーションと同じです。  

`players/tags_controller.rb`のコントローラにアクセスする場合のフォームと  
`managers/tags_controller.rb`の場合のフォームを作る必要があるのですが、  
以下には`players/tags_controller.rb`にアクセスするフォームを見ていきます。  

```erb
<%= form_with(model: [@player, @tag], local: true) do |form| %>
  <% if @tag.errors.any? %>
    <% # エラーメッセージに関するコード %>
  <% end %>

  <div class="field">
    <%= form.label :body %>
    <%= form.text_field :body %>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```

このform_withのモデルの書き方、見慣れないと思う人もいるかもしれません。  
そんな時は、以下を参考にしてじっくりと理解を深めていきましょう。  

- [【Rails】form\_with/form\_forについて【入門】 \- Qiita](https://qiita.com/snskOgata/items/44d32a06045e6a52d11c)
- [【Rails】form\_withの使い方を徹底解説！ \| Pikawaka \- ピカ1わかりやすいプログラミング用語サイト](https://pikawaka.com/rails/form_with#%E3%83%8D%E3%82%B9%E3%83%88%E3%82%92%E3%81%97%E3%81%A6%E3%81%84%E3%82%8B%E6%99%82%E3%81%AE%E6%9B%B8%E3%81%8D%E6%96%B9)

見ていただくと分かりますが、コード上で明示されているのは`:body`というキーだけですが、  
`form_with(model: [@player, @tag]`としてあげることで、`@player.id`をparamsに含めることができます。  

そして、このような書き方をすることで、`/player/:player_id/tags`にPOSTメソッドでリクエストすることになります。  

以上は、これから理解をしていく上でのベースになるので、まず抑えておいてください。  
（まあ、私もこうやって文章を書くことによって、記憶に何とか定着させているんですけど笑）

## フォームから`taggable_type`と`taggable_id`を送ってあげるにはどうすればよいのか？

いくつか方法論があるかと思いますが、シンプルに考えるとこうなると思います。  
隠れパラメータとして送っちゃえばいいんです！！！  

```erb
<%= form_with(model: [@player, @tag], local: true) do |form| %>
  <% if @tag.errors.any? %>
    <% # エラーメッセージに関するコード %>
  <% end %>

  <div class="field">
    <%= form.label :body %>
    <%= form.text_field :body %>
  </div>

  <%= form.hidden_field :taggable_id, value: @player.id %>
  <%= form.hidden_field :taggable_type, value: @player.class %>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```

なお、今回のケースの場合、コントローラを分けないという前提なので、  
`routes.rb`は以下のとおり修正しています。  

```rb
# routes.rb

resources :players do
  resources :tags
end

resources :managers do
  resources :tags
end
```

一応、ルーティングについても再掲しておきます。  
例の如く、createアクション限定ですが。  

```text
       Prefix   Verb   URI Pattern                             Controller#Action
  player_tags   POST   /players/:player_id/tags(.:format)      tags#create
      players   POST   /players(.:format)                      players#create
 manager_tags   POST   /managers/:manager_id/tags(.:format)    tags#create
     managers   POST   /managers(.:format)                     managers#create
```

なお、今回の場合、処理するのは`tags#create`だけなので、  
以下のような`routes.rb`でも構いません。むしろ、望ましいかもしれません。  

```rb
# routes.rb

resources :players
resources :managers
resources :tags
```

ルーティングについて。  
なお、`form_with(model: [@player, @tag]`ではなく、`form_with(model: @tag`とすることをお忘れなく。  

```text
    Prefix   Verb   URI Pattern           Controller#Action
   players   POST   /players(.:format)    players#create
  managers   POST   /managers(.:format)   managers#create
      tags   POST   /tags(.:format)       tags#create
```

## それで、コントローラのcreateアクションををどうすればいいの？  

長くなりましたが、`tags_controller.rb`のcreateアクションについて確認していきましょう。  
短絡的に考えた場合、コードは以下のとおりとなります。  

```rb
  def create
    @tag = Tag.new(tag_params)
    # 他は省略
  end

  private

    def tag_params
      params.require(:tag).permit(:body, :taggable_type, :taggable_id)
    end
```

なお、色々と個人的に検証した結果、最初の例でもモデルの制約が働くので問題がないと思いますが、  
`@taggable.tags.build(tag_params)`の形に落とし込みたい場合、以下のように書くことができます。  

```rb
  def create
    @tag = @taggable.tags.build(tag_params)
    # 他は省略
  end

  private

    def set_taggable
      # constantizeメソッドを使うことで文字を定数化できる
      @taggable = taggable_params[:taggable_type].constantize.find(taggable_params[:taggable_id])
    end

    def taggable_params
      params.require(:tag).permit(:body, :taggable_type, :taggable_id)
    end

    def tag_params
      params.require(:tag).permit(:body)
    end
```

非常にまどろっこしいですが、まず`taggable_params`を使って、`Player.find(@player.id)`した後に、  
`@player.tags.build(tag_params`しているということです。  

## クライアントのリクエスト先のURLを活用して、`taggable_type`と`taggable_id`を取得する方法

コントローラを以下のように書きましょう！  
この方法論を採用する場合、フォームで隠しパラメータを送信する必要がありません。  

```rb
def create
  @tag = @taggable.tags.build(tag_params)

  # 他は省略するが`@tag`をsave + redirectする（失敗した場合、render)
end

private

  def set_taggable
    # `/players/2/tags`の場合、'players'と'2'という要素を取得できる
    resource, id = request.path.split('/')[1,2]
    # `/player/2/tag`の場合、Player.find(2)となる
    # singularizeメソッドにより、'players'が'player'になる
    # classifyメソッドにより、'player'が'Player'になる
    # constantizeメソッドを使うことで文字を定数化できる
    @taggable = resource.singularize.classify.constantize.find(id)
  end

  def tag_params
    params.require(:tag).permit(:body)
  end
```

## 補足： accepts_nested_attributes_forを使う

例えば、PlayerモデルとTagモデルに関する情報を同時にフォームから送りたい場合、  
`accepts_nested_attributes_for`を使おうという情報が出てきます。  

- [ポリモーフィックの子から親を生成する \- インターファーム開発部ブログ](http://interfirm.hatenablog.com/entry/2014/07/30/200431)

この使い方については全然勉強不足なので何とも言えないのですが、ざっと見た限り、  
隠しパラメータを送るという方法を採用しているので、ただTagモデルだけを扱うのであれば、  
あえて`accepts_nested_attributes_for`を使う必要性はないんじゃないかなと感じました。  

## まとめ - どの方法が一番よいのか

初心者の意見ですが、こう考えています。  

- 基本的にはコントローラを分けた方がよい
- コントローラをいくつも作るのがどうしても面倒くさいなら、隠しパラメータを送るのがよい

なぜコントローラを分けた方がよいのか。  
それは、Railsの規約におそらく一番則った方法になるからです。  

誰もが見慣れたコードになって分かりやすいです。  
また、アプリの使用を変える場合にあたっても、柔軟性があります。  

例えば、playerに関するとmanagerに関する閲覧権限を切り分けたいとなった場合、  
コントローラが分かれていると`require ~`とすれば簡単に設定できます。  

ただ、コントローラが１つしかないと、ロジックを書かなくちゃいけないので非常に面倒です。  
