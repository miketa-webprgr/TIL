# Git+Githubのノート

### ○ 基本的な使い方

1. git add . (全てのファイルの場合)
2. git commit
    - git commit . (新規ファイルの作成がない場合、git add . が省略できる)
    - 「-m "メッセージ"」でメッセージを追加できる
    - 「-v」でエディターにてメッセージを追加できる
3. git status, git log, git diff などで状況を確認
    - git log --oneline
    - git log 
    - git diff --staged にてステージとローカルレポジトリの差分を確認できる
4. git push リモート名 ブランチ名 にてリモートレポジトリにプッシュできる

ノートとしてまとめたい項目
    -.gitignoreについて
    -merge/fetch/pullについて
    -mergeとrebaseについて
    -pullrequestについて
    -branchとcheckoutについて
    -tagやstashについて
    -実際の開発を想定した流れについて

