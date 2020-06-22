# P77 has_secure_password メソッドなどを使って認証機能を自分で作っている場合、どのように自分でヘルパーメソッドを定義するのか

## 該当箇所

- 第５章 コントローラスペック
  - 認証が必要なコントローラスペック
    - P77（PDF版）

## 質問内容

Everyday Rails（P77）には、以下のように書いてあります。  

> ・・・もし Rails が提供している has_secure_password メソッドなどを
> 使って認証機能を自分で作っている場合は、次のようにして自分でヘルパーメソッド  
> を定義してみてください。

```rb
# 自分で対処する ....
def sign_in(user)
  cookies[:auth_token] = user.auth_token
end
```

ここを実践＋理解したいと思い、`bcrypt`・`has_secure_password`メソッド・`auth_token`メソッドと`RSpec`
のキーワードを組み合わせて調べてみたのですが、それらしき情報が全くヒットしません。。。

そもそも、`auth_token`メソッドに関する情報が見つかりませんし、`binding pry`で試してみても、
うまくいかないので、`auth_token`メソッドってあるのだろうか・・・と疑っているところです。。。

```
[1] pry(#<RSpec::ExampleGroups::CooksController::Show>)> @user
=> #<User:0x00007fa7946bd620
 id: 116,
 name: "Alice",
 email: "alice@example.com",
 password_digest: "aliceinwonderland",
 created_at: Mon, 22 Jun 2020 14:21:30 UTC +00:00,
 updated_at: Mon, 22 Jun 2020 14:21:30 UTC +00:00,
 admin: false>
[2] pry(#<RSpec::ExampleGroups::CooksController::Show>)> sign_in @user
NoMethodError: undefined method `auth_token' for #<User:0x00007fa7946bd620>
from /Users/HOGE/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/activemodel-5.2.4.3/lib/active_model/attribute_methods.rb:430:in `method_missing'
[3] pry(#<RSpec::ExampleGroups::CooksController::Show>)> @user.auth_token
NoMethodError: undefined method `auth_token' for #<User:0x00007fa7946bd620>
from /Users/HOGE/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/activemodel-5.2.4.3/lib/active_model/attribute_methods.rb:430:in `method_missing'
```

今後は、deviseを使うことの方が多くなってくるかと思うので、  
あまり重要でなければスルーしようかと思いますが、
少し気になったので質問しました。  

スルーした方がよければ、そう言ってもらえると助かります！
