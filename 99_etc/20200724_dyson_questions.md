# 20200724 質問リスト

## 質問１

Railsアプリを作っての相互レビューについて（どういった課題にするか）

## 質問２

業務効率化テクニック（Alfred・Vim・ショートカット ）

## 質問３

カスタムバリデーションについて

- 自分で条件を設けてバリデーションを設けたい場合、どうすればよいのか
- applicationコントローラのようにどこでも使えるバリデーションはどう設定すればよいのか

> - [Active Record バリデーション \- Railsガイド](https://railsguides.jp/active_record_validations.html#%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89)

## 質問４

就職活動関連

- 色々と調べるとフロントサイドはVueよりReactが多い印象だった
- Reactの勉強はVueと比べて大変なのか

## 質問5

なぜRelationshipのマイグレーションにindexを貼るのか

```rb
class CreateRelationships < ActiveRecord::Migration[5.2]
  def change
    create_table :relationships do |t|
      t.integer :follower_id
      t.integer :followed_id

      t.timestamps
    end
    # フォローするユーザー・フォローされるユーザーのidについて、indexを貼る
    # これは何のために必要なのか？ 検索を早くするため？
    add_index :relationships, :follower_id
    add_index :relationships, :followed_id
    # フォローやアンフォローができないよう、ユニーク制約をつける
    add_index :relationships, [:follower_id, :followed_id], unique: true
  end
end
```

## 質問6

フォロー・アンフォローについて
（active_relationshipsとpassive_relationshipsを定義する意味について）
→ 済

## 質問7

Railsテストの解説

## 質問8

リファクタについて（Rubyアプリ）
