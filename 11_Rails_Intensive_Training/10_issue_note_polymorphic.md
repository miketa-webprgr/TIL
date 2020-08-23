# ポリモーフィック関連とは

## ポリモーフィック関連の概要について

いくつかの記事をざっと読んだが、まず以下の記事を読むとイメージが掴みやすいかと思う。  

Railsガイドをよく読みましょうというが、今回のケースについては本当に難しくないし、  
さくっと読み終わるので、一番最初にまず読んでもよいかもしれない。  

- [2.9 ポリモーフィック関連付け --- Active Record の関連付け \- Railsガイド](https://railsguides.jp/association_basics.html#%E3%83%9D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)

「ポリモーフィック」という言葉をよく聞いていたが、要は、複数のモデルに紐づけるようなモデルを  
実装したい場合に使うと便利であるものらしい。  

例えば、studentsテーブルとteachersテーブルがあり、それぞれにprofilesテーブル（名前・住所・顔写真情報などのカラムを所持）  
を関連付けたい場合に、ポリモーフィック関連を使うとよい。Railsガイドに書かれているとおり、実装も簡単である。  

## ポリモーフィック関連って必要？

ざっと理解した感想だが、「え、ポリモーフィックってカッコいい名前のくせに、意外とやることショボいんだな」  
「ポリモーフィック関連って必要ないんじゃない？」「これって何がすごいの？」というものだった。  

だって、シンプルに考えてみれば、こんな感じのテーブルを作ればよいと思ったからだ。  
このテーブルの場合、おそらく、`belongs_to`や`has_many`でうまく関連付けができるような気がする。  
（いや、この事例がググっても出てこないので、おそらくできないのだろうけど。。。）  

```text
studentsテーブル
id: 1
grade: 5 （小学校５年生）

teachersテーブル
id: 1
subject: Social Studies（社会科）

profilesテーブル
id: 1
student_id: 1
teacher_id: NULL
name: miketa
address: Saitama
avatar: miketa.jpg

id: 2
student_id: NULL
teacher_id: 1
name: onizuka
address: Kichijoji
avatar: onizuka.jpg
```

ただ、ポリモーフィック関連の記事等を読んで気付いたが、NULLが入ることを避けるのであれば、以下のようなテーブル設計が  
賢いのかもしれないということが分かった。このテーブル構造こそが、ポリモーフィック関連を使う場合に求めているものである。  
（ただ、本当にこっちの方が良いのか、あまり咀嚼できていない。。。どうなんだろう。。。）  

```text
studentsテーブル
id: 1
grade: 5 （小学校５年生）

teachersテーブル
id: 1
subject: Social Studies（社会科）

profilesテーブル
id: 1
target_table: students
target_id: 1
name: miketa
address: Saitama
avatar: miketa.jpg

id: 2
target_table: teachers
target_id: 1
name: onizuka
address: Kichijoji
avatar: onizuka.jpg
```

ポリモーフィックが求めているテーブル構造が賢そうということが分かったが、この場合、当然ながら`belongs_to`と`has_many`では、  
テーブル間をうまく紐づけることができない。（`target_table`の値から紐付け先の親テーブルを判断するため）  

そこで、Railsガイドで説明されているとおり、以下のとおり書けば関連付けすることができる。  
（検証していないので間違えているかも。もしくは動かないかも）  

```rb
class Profile < ApplicationRecord
  # 通常はモデル名を流用する形で`belongs_to :-----able`とするのが慣習らしい
  belongs_to :profileable, polymorphic: true
end

class Student < ApplicationRecord
  has_many :profiles, as: :target

class Teacher < ApplicationRecord
  has_many :profiles, as: :target
end
```

## ポリモーフィック関連ってやっぱり不要じゃない？

以下の記事を読んで、いろいろな書き方があることを知った。  

詳細は記事に委ねるが、要は中間テーブルを使ってしまえば、`NULL`を含むことなく実装できるらしい。  
また、親テーブルを用意してしまえば、polymorphicを使うことなく実装できてしまうらしい。  

- [複数のテーブルに対して多対一で紐づくテーブルの設計アプローチ｜スパイスファクトリー株式会社](https://spice-factory.co.jp/development/has-and-belongs-to-many-table/)
- [SQLアンチパターンを読んで （ポリモーフィック関連について） \| MotiMoti\+\+](https://blog.motimotilab.com/?p=207)

中間テーブルを実装する例についてはイメージがつきやすいので省略するが、親テーブルを用意する場合、  
先程の事例で言えば、profilesテーブルを用意するのではなく、共通部分をusersテーブルのカラムに入れてしまい、  
studentsに特有のgradeカラムとteachersに特有のsubjectカラムだけ別テーブルにしてしまえばよい。  

```text
usersテーブル
id: 1
name: miketa
address: Saitama
avatar: miketa.jpg

id: 2
name: onizuka
address: Kichijoji
avatar: onizuka.jpg

studentsテーブル
id: 1
user_id: 1
grade: 5 （小学校５年生）

teachersテーブル
id: 1
user_id: 2
subject: Social Studies（社会科）
```

・・・ん、事例にもよるだろうが、この親テーブルパターンが最強じゃないか。。。  
あと、よく分かっていないが、いくつかの記事にはポリモーフィック関連はSQLアンチパターンと書いてあるし、  
ポリモーフィック関連ってなんかいいことあるのだろうか。  

## ちょっと寄り道して、単一テーブル継承(STI:Single Table Inheritance)についても理解してみる

ポリモーフィック関連について調べていく中で、単一テーブル継承なるものが頻出した。  
理解するのに時間がかかったが、落ち着いて読んでいくと理解できたのでまとめてみた。  

- [【Rails】ActiveRecord：単一テーブル継承\(sti\)とポリモーフィック関連を未だにぱっと思い出せないのでまとめ。 \- 訳も知らないで](https://shirusu-ni-tarazu.hatenablog.jp/entry/2012/11/04/173742)

単一テーブル継承とは、その言葉のとおり、一つの巨大なテーブルに全てを打ち込んでしまう  
という考え方だ。今までのリレーショナルデータベースとは一体何だったのか。  

つまり、先程の例で言えば、こんな感じになる。  
当然、`NULL`は入ってくる。  

```text
usersテーブル
id: 1
name: miketa
address: Saitama
avatar: miketa.jpg
type: student
grade: 5（小学校５年生）
subject: NULL（teacherじゃないので）

id: 2
name: onizuka
address: Kichijoji
avatar: onizuka.jpg
type: teacher
grade: NULL（studentじゃないので）
subject: Social Studies（社会科）
```

実は、以下の記事によると、テーブル設計においては以下の３つの考え方があるらしく、  
もっと積極的に単一テーブル継承について考えてもよいと主張している。  
（とはいえ、コメント欄を見る限り、単一テーブル継承については反対している人が多いけど）  

1. 単一テーブル継承
2. クラステーブル継承
3. 具象クラス継承に

【テーブル設計についてのQiita記事】  

- [みんなRailsのSTIを誤解してないか\!? \- Qiita](https://qiita.com/yebihara/items/9ecb838893ad99be0561)
- [単一テーブル継承・クラステーブル継承・具象クラス継承について \- Qiita](https://qiita.com/bmf_san/items/a03820b14a72db618d15)

以上の記事を読んでいて、混乱が生じ、そして時間が経過してやっと気付いたのだが、  
そもそもテーブルとモデルというのは、必ずしも同じような構造でなくてもよいらしい。  
（Railsでは基本的にクラステーブル継承の考え方に基づいて設計されているので、そのように勘違いしていた）  

例えば、以下では単一テーブル継承をしつつ、ポリモーフィック関連を実装する方法を紹介しているが、  
データベース上では巨大なテーブル１つしか用意しないけど、モデルは３つ用意するなんていうのもありだ。  

- [STIとポリモーフィックを利用したコメント機能の作成 \- Qiita](https://qiita.com/shii_yu/items/08aebde89f1e1c144054#comments)

データベースとモデルは同じ構造でなければいけないと思い込んでいたので、ここに気付けたのは大きな発見だった。  

## ポリモーフィック関連の利点とは

この記事に書いてあるが、結局、ポリモーフィック関連が便利なのは、  
「一つのモデルを同じインターフェースを持ったものが扱う（ダックタイピングする）」場合なのだと思う。  

- [Railsのポリモーフィック関連とはなんなのか \- Qiita](https://qiita.com/itkrt2y/items/32ad1512fce1bf90c20b)

例えば、既出のProfile（ポリモーフィック）・Student・Teacherという３つのモデルの場合、  
StudentとTeacherモデルにおいて`introduce_youself`というメソッドをそれぞれ定義したとすれば、  
`Profile.find(params[:id]).profileable.introduce_youself`とすれば、モデルの差異を気にする必要がない。  

stundentの場合であれば、"I am a 5th grade student"と自己紹介してくれるし、  
teacherの場合であれば、"I am teaching history"と自己紹介してくれる。  

## ポリモーフィック関連の利点についてさらに考える

以上のように書いたが、例えば５つのテーブルにおいて共通のことを表現したい場合、  
ポリモーフィック関連は便利かもしれない。

- 中間テーブルを作ってもよいが、５つも作らないといけなくて面倒。
- 親テーブルを作ってもよいが、関連する５つの全てのテーブルに外部キーを持たせなければいけない。  
  追加的に新しくテーブルを作りたくなった場合、ポリモーフィックの方が楽かもしれない。  

## その他の参考記事

- [【初心者向け】Railsのポリモーフィック関連付けを理解しよう \- Qiita](https://qiita.com/yu-croco/items/0aa9b2f8a797ee0a3515)
- [ポリモーフィック関連のコントローラー \- 猫Rails](http://nekorails.hatenablog.com/entry/2019/06/13/031003)
- [そろそろポリモーフィック関連について一言いっとくか \- Qiita](https://qiita.com/joker1007/items/9da1e279424554df7bb8#comments)

特に後者の２つについては、理解がある程度深まってから読むと良いと思う。  
ある程度理解が深まってから読むと意味が分かったが、最初のうちに読んでもかなり意味不明である。  
