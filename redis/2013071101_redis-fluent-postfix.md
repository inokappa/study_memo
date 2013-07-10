# postfix のログを fluentd を使って redis に突っ込んでみる（1）

## きっかけ

 * いつの間にか膨大になっている maillog をなんとかしたい
 * 「メールが届いてないんだけど？」という問い合わせに膨大なログから手早く該当のログを見つけ出すカッコイイ方法はないかなと思いたったが吉日

***

## 環境

 * Debian 6.0.x
 * Postfix
 * fluentd
 * redis

***

## 準備

### postfix の準備

Debian の標準 MTA は `Exim4` なので postfix と入れ替える

```
apt-get install postfix
```

冒頭で `Exim4` をアンインストールするか否かを問われるので恐縮しつつアンインストール。

### fluentd の準備

`ruby 1.9.3p392 (2013-02-22) [x86_64-linux]` な環境に `gem` でインストールしてみる。

```
sudo gem install fluentd --no-ri --no-rdoc -V
```

実行後、暫くすると...

```
Successfully installed fluentd-0.10.35
```

と出力され、無事にインストールが出来たようなので、[こちら](http://docs.fluentd.org/ja/articles/install-by-gem)の通りにインストールを確認してみる。

```
fluentd --setup ./fluent
fluentd -c ./fluent/fluent.conf -vv &
echo '{"json":"message"}' | fluent-cat debug.test
```

上記を実行すると以下のように出力された。

```
% echo '{"json":"message"}' | fluent-cat debug.test
2013-07-11 05:24:59 +0900 [trace]: plugin/in_forward.rb:147:initialize: accepted fluent socket object_id=18704480
2013-07-11 05:24:59 +0900 debug.test: {"json":"message"}
2013-07-11 05:24:59 +0900 [trace]: plugin/in_forward.rb:188:on_close: closed fluent socket object_id=18704480
```

とにかく速いって思った。

### redis の準備

泣く子も黙る `redis` は以下のようにビルドしてインストール。

```
wget http://redis.googlecode.com/files/redis-2.6.14.tar.gz
tar zxvf redis-2.6.14.tar.gz
cd redis-2.6.14
ls
make
sudo make install
```

こんなに毎回素直にビルドが通るプロダクトは個人的には初めて。好きです `redis`

一応、デーモンモードで起動したいので `redis.conf` の `daemonize no` を `daemonize yes` として `redis-server` を起動する。尚、`redis.conf` は展開したソースコード内にあるので適宜コピーして利用する。

```
sudo /usr/local/bin/redis-server /usr/local/etc/redis.conf
```

以下のように起動を確認する。

```
/usr/local/bin/redis-cli ping
```

起動していれば `PONG` と返ってくる。標準では `6379` ポートで Listen する。

### fluentd の redis プラグインの準備

`fluentd` と `redis` を連携させる為の `fluent-plugin-redis` をインストールする。これも `gem` でインストール。

```
sudo gem install fluent-plugin-redis --no-ri --no-rdoc -V
```

暫くすると以下のようにインストールが完了する。

```
Successfully installed redis-2.2.2
Successfully installed fluent-plugin-redis-0.2.0
```

***

## やってみる

### fluent.conf の設定

`fluent.conf` を以下のように記載する。

```
<source>
  type tail
  format /^(?<date>[^ ]+) (?<host>[^ ]+) (?<process>[^:]+): (?<message>((?<key>[^ :]+)[ :])? ?((to|from)=<(?<address>[^>]+)>)?.*)$/
  path /var/log/mail.log
  tag redis.maillog
  pos_file /tmp/fluent.pos
</source>

<match redis.**>
  type redis

  host 127.0.0.1
  port 6379

  # database number is optional.
  db_number 0        # 0 is default
</match>
```

`format` の正規表現は[こちら](http://d.hatena.ne.jp/sfujiwara/20120326/1332760934)をそのまま利用させて頂きました。

### mail.log のフォーマットを修正

デフォルトは mail.log のフォーマットは下記のようなフォーマットとなり、上記の正規表現では `fluentd` 側でログをパース出来ない...

```
Jul 11 05:11:05 debian postfix/master[3179]: daemon started -- version 2.7.1, configuration /etc/postfix
```

原因は日付の部分が `Jul 11 05:11:05` となっている為で、ヘタレな自分はとりあえずログのフォーマットを変更した。

`/etc/rsyslog.conf` を以下のように修正し、`/etc/init.d/rsyslog restart` した。

```
#$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
$ActionFileDefaultTemplate RSYSLOG_FileFormat
```

修正後は以下のようになった。

```
2013-07-11T06:31:31.032225+09:00 debian postfix/qmgr[7636]: 06D5723728: from=<>, size=2036, nrcpt=1 (queue active)
```

### 起動

以下のように `fluentd` を起動する。

```
sudo fluentd -c fluent.conf -vv
```

`sudo` を付けないと `/var/log/mail.log` を権限の関係で参照出来ないので起動しない。

***

## 確認

### テストメール

mail コマンドでメールを送ってみる

```
mail hoge@hoge.com
Subject: test
test
.
Cc: 
```

### 確認

`redis-cli` を利用して `redis-server` にアクセスしてテストメールの内容がログに記録されているかを確認する。

```
/usr/local/bin/redis-cli
```

`redis-server` にアクセスしてキーの一覧を取得する。

```
redis 127.0.0.1:6379> keys *
 1) "redis.maillog.1373493704.9"
 2) "redis.maillog.1373493704.4"
 3) "redis.maillog.1373493704.2"
 4) "redis.maillog.1373493704.1"
 5) "redis.maillog.1373493704.7"
 6) "redis.maillog.1373493704.0"
 7) "redis.maillog.1373493704.3"
 8) "redis.maillog.1373493704.8"
 9) "redis.maillog.1373493704.5"
10) "redis.maillog.1373493704.6"
```

おお、ログが記録されてるじゃあーりませんか。

```
redis 127.0.0.1:6379> HGETALL redis.maillog.1373493704.2
 1) "host"
 2) "debian"
 3) "date"
 4) "2013-07-11T07:01:44.672404+09:00"
 5) "process"
 6) "postfix/qmgr[7636]"
 7) "message"
 8) "A20D023759: from=<kappa@debian.inokara.com>, size=303, nrcpt=1 (queue active)"
 9) "address"
10) "kappa@debian.inokara.com"
11) "key"
12) "A20D023759"
```

ハッシュの中身も上記のように記録されている。ということで、思い立ったが吉日でサクッとログの記録まで試せたのは良かった。そして、今日も先人たちの努力に感謝。

***

## 課題

 * 業務現場に投入する場合にはログのフォーマットを確認する必要があり、ログフォーマットを変更するのかパースする正規表現を頑張るかしないといけない
 * 記録するログを精査した上で `redis-server` に放り込むようにしたい 

***

## 参考

 * [#fluentd で maillog を読み込んで MongoDB に投入](http://d.hatena.ne.jp/sfujiwara/20120326/1332760934)
 * [Ruby GemからFluentdをインストールする](http://docs.fluentd.org/ja/articles/install-by-gem)
 * [fluent-plugin-redis](https://github.com/yuki24/fluent-plugin-redis)
 * [fluentd のストレージとして redis を使ってみる一部始終（調査編）](http://inokara.hateblo.jp/entry/2013/04/27/092845)
