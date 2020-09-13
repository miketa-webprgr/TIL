# linuxコマンド

## コマンド一覧

ctrl b １文字戻る
ctrl f １文字進む
ctrl a 行頭に戻る
ctrl e 行末に進む
alt b １単語戻る
alt f １単語進む
ctrl h カーソルの前の文字を削除
ctrl d カーソル上の文字を削除
ctrl w １単語を削除
ctrl k カーソル位置から行末までをカットする
ctrl u カーソル位置から行頭までをカットする
ctrl y カットしたものをヤンク（ペースト）する

## VSCodeでの設定方法

vscodeでaltをメタキーとして使用するには、以下を設定する。  

```text
"terminal.integrated.macOptionIsMeta": true
```
