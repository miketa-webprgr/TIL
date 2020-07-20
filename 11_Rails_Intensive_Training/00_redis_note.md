# Redisノート

## ここを見るとよい

- [Redisとは？RailsにRedisを導入 \- Qiita](https://qiita.com/hirotakasasaki/items/9819a4e6e1f33f99213c)
- あと、だいそんさんが動画で解説してくれている

## よく使うコマンド

### Redisサーバーの起動

redis-server

### Redisサーバーに接続

$ redis-cli
127.0.0.1:6379>

### KVS(Key Value Store)を利用する

redis> SET mykey "Hello"
OK
redis> GET mykey
"Hello"
redis> quit
