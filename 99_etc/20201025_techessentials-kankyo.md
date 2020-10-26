# TechEssentialsのDockerを使わない環境構築

## はじめに

Dockerが重たいという残念なあなたに送る、ローカルの環境構築。  

残念の先駆者、みけたが得た知見を書いてます。合っているかは知りません。  
あくまで参考までに活用ください。  

## DB関係

PostgresとRedisが動けばいけるはず。  
本当の初学者向けに、ここも充実させていけたらいいなと思ったり。  

## Puma関係のエラー

みけたは以下のエラーに遭遇しました。  

```text
An error occurred while installing puma (4.3.5), and Bundler cannot continue. Make sure that `gem install puma -v '4.3.5' --source 'https://rubygems.org/'` succeeds before bundling.
```

解決にあたっては、以下を参考にしました。  

https://github.com/puma/puma/issues/2342  

具体的には、以下のコマンドを打ち込んだら解決しました。  
すみません、勉強不足で理由はよく分かってません汗  

```text
bundle config build.puma --with-cflags="-Wno-error=implicit-function-declaration"
```

## その他

Pumaのエラー以外は特に遭遇せず、スムーズにいきました。  
何か特異なエラーに遭遇した場合、知見を共有できるとよいかも。  

Docker使いに負けず、頑張りましょう笑  