# Issue01 Gitflowとは

## git flowとは

git flowで出てくるブランチについて、ざっと確認。  
だいそんさんのノートを読むまで、developブランチの意義が理解出来ずにいた。  
特に、図が分かりやすくてありがたかった。  

## 開発者視点でまとめていく  

### 平常時

1. developブランチからfeatureブランチを作成する
2. featureブランチで機能実装などを行う
3. プルリクを送り、レビューを受ける
4. レビューでOKが出たら、developブランチにマージする
5. featureブランチは捨てる
6. developブランチがデプロイできるぐらいの作業量になったら、releaseブランチを作成する
7. releaseブランチにて、デプロイして問題がないか確認する
8. バグが見つかれば、developにマージしておく
9. 問題がなければ、masterにマージ！！！

### 緊急時

1. masterブランチからhotfixブランチを作成
2. 急いでバグを修正！！！
3. masterにマージ！！！！！

### 今回のケースにおいて何をやって行けばいいのか

1. `YAY! Your're on Rails!`の状態まで持っていき、`rails new` する
2. `git flow init`する
3. `git remote add origin 追加したいリポジトリ` にて登録してリモートレポジトリを登録しておく
4. `git push origin master`にてリモートに初期状態をpushしておく
5. `git flow feature start 01_login_logout（新しいfeatureブランチ）` する
6. featureブランチで作業を進める
7. 適切な粒度でcommitする
8. `git flow feature publish 01_login_logout`をする（pushしているのと同じ）
9. developブランチにマージできるところまで作業が進んだら、プルリクエストを送る
10. レビューをパスしたら、developブランチにマージする（される）
11. developブランチにcheckoutして、developブランチをpullする
12. 新しいfeatureブランチを切り、作業を進める

### 参考資料

- [git flowについて · DaichiSaito/insta\_clone Wiki](https://github.com/DaichiSaito/insta_clone/wiki/git-flow%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)
- [Git\-flowって何？ \- Qiita](https://qiita.com/KosukeSone/items/514dd24828b485c69a05)
- [いまさらだけどGitを基本から分かりやすくまとめてみた \- Qiita](https://qiita.com/gold-kou/items/7f6a3b46e2781b0dd4a0)
