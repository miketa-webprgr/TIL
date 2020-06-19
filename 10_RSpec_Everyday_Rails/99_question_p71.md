# P71 モデルテストのAssociationについて

## 該当箇所

- 第４章 意味のあるテストデータの作成
  - ファクトリで関連を扱う
    - P71（PDF版）

## 質問内容

理解しづらい部分がありました。  
自分なりにこういう理解でよいのか考えてみましたが、その考えに誤りがないか教えてください！  

## 関係コード

```rb: spec/models/note_spec.rb
require 'rails_helper'

RSpec.describe Note, type: :model do
  it "generates associated data from a factory" do
    note = FactoryBot.create(:note)
    puts "This note's project is #{note.project.inspect}"
    puts "This note's user is #{note.user.inspect}"
  end
end
```

```rb: spec/factories/notes.rb
FactoryBot.define do
  factory :note do
    message { "My important note." }
    association :project
    association :user
    # 正解は、user { project.owner }
  end
end
```

```rb: spec/factories/users.rb
FactoryBot.define do
  factory :user, aliases: [:owner] do
    first_name { "Aaron" }
    last_name  { "Sumner" }
    sequence(:email) { |n| "tester#{n}@example.com" }
    password { "dottle-nouveau-pavilion-tights-furze" }
  end
end
```

```rb: spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Project #{n}" }
    description { "A test project." }
    due_on { 1.week.from_now }
    association :owner
  end
end
```

## 理解しづらい文章の抜粋（ PDF版 - P71 ）

ここでは Factory Bot を1回しか呼んでいないにもかかわらず、  
テストの実行結果を見ると必要なデータが全部作成されています。  

```
Note
This note's project is #<Project id: 1, name: "Test Project 1",
description: "Sample project for testing purposes", due_on:
"2017-01-17", created_at: "2017-01-10 04:01:24", updated_at:
"2017-01-10 04:01:24", user_id: 1>

This note's user is #<User id: 2, email: "tester2@example.com", created_at: "2017-01-10 04:\ 01:24", updated_at: "2017-01-10 04:01:24",
first_name: "Aaron", last_name: "Sumner">
```

ですが、この例はファクトリで関連を扱う際の潜在的な落とし穴も示しています。  
みなさんは わかりますか?ユーザーのメールアドレスをよく見てください。  

なぜ tester1@example.com ではなく、tester2@example.com になっているのでしょうか?  
この理由はメモのファクトリが関連するプロジェクトを作成する際に関連するユーザー  
(プロジェクトに関連する owner)を作成し、それから2番目のユーザー(メモに関連するユーザー)  
を作成するからです。

## 自分なりの解釈

概念的な感じでまとめてみました。  
そもそも、associationの理解が不完全なだけなのかもしれませんけど。。。

```rb: spec/models/note_spec.rb
FactoryBot.define do
  factory :note do
    message { "My important note."}
    association :project
      ### ここで projects の FactoryBot が呼ばれる

      # factory :project do
        # sequence(:name) { |n| "Project #{n}" }
        # description { "A test project." }
        # due_on { 1.week.from_now }
        # association :owner
          ### associationにより、User id: 1 のユーザーが作成される
      # end

    association :user
      ### ここで user の FactoryBot が呼ばれる
      ### ここで、User id: 2のユーザーが作成される

      # factory :user, aliases: [:owner] do
      #   first_name { "Aaron" }
      #   last_name  { "Sumner" }
      #   sequence(:email) { |n| "tester#{n}@example.com" }
      #   password { "dottle-nouveau-pavilion-tights-furze" }
      # end

  end
end
```
