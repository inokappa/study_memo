# redis-router を試してみる

## 概要

[redis-router](https://github.com/emre/redis-router) という python で書かれた redis のインターフェースがあるらしいので試す。きっかけは [@sawanoboly](https://twitter.com/sawanoboly) さんのツイート。

[https://twitter.com/sawanoboly/status/350221849596145664:embed#ほう Redis-router http://t.co/sdZOMG4rGQ via @zite]


### redis-router とは

 * 複数の redis server を共有する為の Library や API を提供する
 * [コンシステントハッシュ法](http://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%B3%E3%82%B7%E3%82%B9%E3%83%86%E3%83%B3%E3%83%88%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5%E6%B3%95)というハッシュ技術を用いて処理の分散を行なっている

***

## セットアップ

### 検証環境

 * Debian 7（Wheezy）
 * localhost にポートで分けた redis server を 2 個起動する

### トポロジ

今回は以下のような構成で検証を行う。

<p><span itemscope itemtype="http://schema.org/Photograph"><img src="http://cdn-ak.f.st-hatena.com/images/fotolife/i/inokara/20130630/20130630115106.png" alt="f:id:inokara:20130630115106p:plain" title="f:id:inokara:20130630115106p:plain" class="hatena-fotolife" itemprop="image"></span></p>

※図は [blockdiag](http://blockdiag.com/ja/index.html) を使って生成しました。

### redis server の準備

redis server を localhost に chef を使ってセットアップする。

```
sudo chef-solo -c ~/path/to/solo.rb -j ~/path/to/localhost.json
```

cookbook は[こちら](https://github.com/inokappa/redis_2_cookbook.git)の cookbook を使い、JSON ファイルは以下のように設定した。

<script src="https://gist.github.com/inokappa/5892845.js"></script>

インストール後、redis の tar ball に含まれていた redis.conf を /usr/local/etc/ 以下にコピーして以下のルールに従い起動するポート番号ごとに設定を行う。また、合わせてデーモンモードで起動する為の設定も合わせて行う。

| ホスト | ポート番号 |
|:-------|:----------|
| localhost | 6379 |
| localhost | 6380 |

以下は設定変更箇所。

```
port 6379
```

port に続く番号を任意の番号に設定する。また、デーモンモードで起動する為に以下のように設定する。

```
daemonize yes
```

それぞれの redis.conf を設定後、以下のようにして redis server を起動する。

```
sudo /usr/local/bin/redis-server /usr/local/etc/6379.conf
sudo /usr/local/bin/redis-server /usr/local/etc/6380.conf
```

特にエラーも無く起動した場合には `redis-cli -p 6379` 及び `redis-cli -p 6380` として、それぞれの redis server にアクセス出来ることを確認しておく。

### pip のインストール

redis-router は python のモジュールとして提供される為、python モジュールの管理ツールである pip を利用してインストールする。尚、検証環境には pip がインストールされていないため pip のインストールから行う。

```
sudo apt-get install python-pip
```

あら、簡単。

### redis-router のインストール

[README](https://github.com/emre/redis-router)を見ると `install libketama/ketama_python first.` とあるので、以下のようにインストールする。

<script src="https://gist.github.com/inokappa/5892846.js"></script>

python_ketama のインストールが終わったら、本命の redis-router のインストール。これは、先ほどインストールした pip にて行う。

```
pip install redis-router
```

### redis-router を試すにあたって...

検証環境において redis-router のインストール後、[README](https://github.com/emre/redis-router) に書かれている内容を試すにあたり以下のパッケージを追加でインストールする必要があった。環境によってはすでにインストールされているかもしれないので適宜読み替えること。

Debian パッケージ。

```
sudo apt-get install libevent-dev
```

pip パッケージ。

```
sudo pip install gevent
sudo pip install greenlet
sudo pip install ketama
sudo pip install flask
```

***

## telnet でアクセスしてみる

### tcp インターフェースサーバーを起動

[README](https://github.com/emre/redis-router) の通り、以下のような python コードを書いてみる。

<script src="https://gist.github.com/inokappa/5892849.js"></script>

どうやらこれで redis-router の tcp インターフェースを立ち上げることが出来るらしい。

以下のように起動する。

```
python tcp.py &
```

### アクセスしてみる

アクセスしてみる。

```
telnet localhost 5000
```

以下のように telnet を介してキーと値の登録が出来る。

```
Trying ::1...
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
set hoge test1
True
get hoge
test1
^]
```

登録の状態を redis-cli で確認する。

```
$ redis-cli -p 6379 get hoge
"test1"
$ redis-cli -p 6380 get hoge
(nil)
```

`6379` ポートで起動している redis server に書き込まれている。

## http リクエストでアクセスしてみる

### http インターフェースサーバーの起動

telnet と同様に [README](https://github.com/emre/redis-router) の通り、以下のような python コードを書いてみる。

<script src="https://gist.github.com/inokappa/5892852.js"></script>

また、`/etc/redis_router/servers.config` を設定する必要があるので、下記のように redis server の IP と Listen しているポート、重み付けを設定する。

```
# IP address:port weight
127.0.0.1:6379 100
127.0.0.1:6380 100
```

このファイルのパスは `/usr/local/lib/python2.7/dist-packages/redis_router/http_interface.py` に直書きされているので、修正することで任意のパスに記載することが可能かと思われる。（以下、抜粋）

<script src="https://gist.github.com/inokappa/5892854.js"></script>

以下のように起動する。

```
python httpd.py &
```

### アクセスしてみる

curl を使って先ほど登録したデータを取得してみる。

```
curl -X POST --data "command=get&arguments=hoge" http://localhost:5001
```

POST メソッドで以下のようなレスポンスが返ってくる。

```
127.0.0.1 - - [2013-06-29 16:04:42] "POST / HTTP/1.1" 200 25 "-" "curl/7.26.0"
{
  "response": "test1"
}
```

ついでにデータを登録してみる。

```
curl -X POST --data "command=set&arguments=huga,test2" http://localhost:5001
```

下記のようなレスポンスが返ってくる。

```
127.0.0.1 - - [2013-06-29 16:07:44] "POST / HTTP/1.1" 200 22 "-" "curl/7.26.0"
{
  "response": true
}
```

登録したデータを確認してみる。

```
curl -X POST --data "command=get&arguments=huga" http://localhost:5001
```

以下の通り、ちゃんと登録されている。

```
127.0.0.1 - - [2013-06-29 16:08:52] "POST / HTTP/1.1" 200 25 "-" "curl/7.26.0"
{
  "response": "test2"
}
```

redis-cli でも確認。

```
$ redis-cli -p 6379 get huga
(nil)
$ redis-cli -p 6380 get huga
"test2"
```

今度は `6380` ポートで起動している redis server に登録された。

***

## バランシング

検証環境において redis-router 配下には 2 つの redis server がそれぞれ別ポートで稼働しているところで、重み付けを変えてアクセスしてみる。

### 同比（1:1）

`/etc/redis_router/servers.config` を以下のように記載する。

```
127.0.0.1:6379 100
127.0.0.1:6380 100
```

リクエストを投げてみる。

```
$ curl -X POST --data "command=set&arguments=huga,test2" http://localhost:5001 
127.0.0.1 - - [2013-06-29 16:15:38] "POST / HTTP/1.1" 200 22 "-" "curl/7.26.0"
{
  "response": true
}
$ curl -X POST --data "command=set&arguments=hoge,test1" http://localhost:5001
127.0.0.1 - - [2013-06-29 16:15:48] "POST / HTTP/1.1" 200 22 "-" "curl/7.26.0"
{
  "response": true
}
```

確認してみる。

```
$redis-cli -p 6379
redis 127.0.0.1:6379> keys *
1) "hoge"
redis 127.0.0.1:6379> quit
$ redis-cli -p 6380
redis 127.0.0.1:6380> keys *
1) "huga"
redis 127.0.0.1:6380> quit
```

### 1:2

`/etc/redis_router/servers.config` を以下のように記載する。

```
127.0.0.1:6379 100
127.0.0.1:6380 200
```

リクエストを投げてみる。

```
$ curl -X POST --data "command=set&arguments=hoge,test1" http://localhost:5001
127.0.0.1 - - [2013-06-29 16:30:31] "POST / HTTP/1.1" 200 22 "-" "curl/7.26.0"
{
  "response": true
}
$ curl -X POST --data "command=set&arguments=huga,test2" http://localhost:5001
127.0.0.1 - - [2013-06-29 16:30:40] "POST / HTTP/1.1" 200 22 "-" "curl/7.26.0"
{
  "response": true
}
$ curl -X POST --data "command=set&arguments=aho,test3" http://localhost:5001
127.0.0.1 - - [2013-06-29 16:30:54] "POST / HTTP/1.1" 200 22 "-" "curl/7.26.0"
{
  "response": true
}
```

確認してみる。

```
$ redis-cli -p 6379
redis 127.0.0.1:6379> keys *
1) "aho"
redis 127.0.0.1:6379> quit
$ redis-cli -p 6380
redis 127.0.0.1:6380> keys *
1) "huga"
2) "hoge"
```

ちゃんと 1:2 の割合で登録されている。

***

## まとめ

という感じで駆け足で redis-router を弄ってみた感想は下記の通り。（追記していく予定）

 * redis server へのアクセスを簡素化してより redis をより使い易い KVS にしてくれてる
 * レスポンスが JSON 形式なので加工とかもしやすそう
 * redis-router を書き込み用のポートと読み込み用のポートを別々で起動することで読み込みの負荷分散を手軽に行えそう
