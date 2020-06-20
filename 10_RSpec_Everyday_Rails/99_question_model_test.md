# モデルテスト関係（どちらかというとRubyの話かも）

<br>

## 質問内容

モデルスペック及びファクトリボットの使い方を学んだので、  
実際のアプリに対してテストを書いているところです。  

そこで、以下のようなコードを書いてみたのですが、  
なぜか想定したとおりに  ならないことに気付きました。  

デバッグツールを使ってみると、user_aliceの中身がどんどん空になっていっていることが
分かったのですが、どういうキーワードで調べていけばよいですか？

user_aliceを代入している認識なので、user_aliceは影響しないという認識なんですが、
何が影響して、user_aliceの属性値が空になっているのでしょうか。。。？

<br>

## 関係コード

```rb
RSpec.describe User, type: :model do
  
  user_alice = User.new(
    name: "Alice",
    email: "alice@example.com",
    password_digest: "aliceinwonderland",
  )  
  
  it "name, email, password_digest, admin があれば有効な状態であること" do
    expect(user_alice).to be_valid
  end

  it "nameがなければ無効な状態であること" do
    # こういう書き方がよくないのかも？（けど、なぜかがピンとこない・・・）
    no_name_user = user_alice
    
    binding.pry
    # この時は user_alice に name: "Alice", email: "alice@example.com", password_digest: "aliceinwonderland" が存在する
    
    no_name_user[:name] = ""
    no_name_user.valid?
    expect(no_name_user.errors[:name]).to include("を入力してください")
  end
  
  it "emailがなければ無効な状態であること" do
    no_email_user = user_alice

    binding.pry
    # この時は user_alice の name: "" となっている

    no_email_user[:email] = ""
    no_email_user.valid?
    expect(no_email_user.errors[:email]).to include("を入力してください")
  end
  
  it "password_digestがなければ無効な状態であること" do
    no_pass_user = user_alice
  
    binding.pry
    # この時は、user_alice が name: "", email: "" となっている

    no_pass_user[:password_digest] = ""
    no_pass_user.valid?
    expect(no_pass_user.errors[:password]).to include("を入力してください")
  end

end
```

## 回答

[他の変数を代入 \- 変数 \- Ruby入門](https://www.javadrive.jp/ruby/var/index4.html)

EverydayRailsの後ろの方に出てくるletを活用すると良いですよ。  
インスタンス変数は基本定義しませんね。  
