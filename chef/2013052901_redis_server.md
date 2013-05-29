# Chef を使って redis サーバーをセットアップする一部始終

## 概要

 * redis サーバーをセットアップする
 * せっかくなので Chef を使ってセットアップするが LWRP を使ったレシピを書いてみる
 * LWRP を使ったレシピと使わないレシピの比較をしてみる
 
## 参考

検証にあたっては下記を参考にさせて頂いた。

  * ["LWRPによる"続・ChefでSourceから何かをインストールするCookbookのウォークスルー](http://qiita.com/items/e4d840da4d91b0379c65)

## redis サーバーについて

 * aaa
 * aaa

## 普通の cookbook

### まずはこう書く

ベタな書き方。

`redis/site-cookbook/resdis.rb`

```ruby
# Debian 系と redhat 系の分岐が必要だけど...
package "tcl8.5" do
  action :install
end

# 作業用ディレクトリの作成
directory "/usr/local/src/redis" do
  action :create
  not_if "ls -d /usr/local/src/redis"
end

# 最新のソースコードを取得
remote_file "/usr/local/src/redis/redis-2.6.13.tar.gz" do
  source "http://redis.googlecode.com/files/redis-2.6.13.tar.gz"
  not_if "ls /usr/local/bin/redis-server"
end

# ソースコードのアーカイブを展開して make && make test && make install
bash "install_redis_program" do
  user "root"
  cwd "/usr/local/src/redis/"
  code <<-EOH
    tar -zxf redis-2.6.13.tar.gz
    (cd redis-2.6.13/ && make test && make && make install)
  EOH
  not_if "ls /usr/local/bin/redis-server"
end

# サービスを起動する
bash "start_redis_server" do
  user "root"
  cwd "/usr/local/bin"
  code <<-EOH
    /usr/local/bin/redis-server &
  EOH
  not_if "ps aux | grep \[r\]edis-server"
end

```

この書き方のダメな点としては、バージョン番号等がベタ書きで汎用性が非常に低い点が挙げられる...。

### node に適用してみる

せっかくなので chef-zero サーバーにアップして knife xenserver vm create してみる。

```
cd ${chef-repo}/site-cookbook/
knife cookbook upload redis -o ./
```

```
knife xenserver vm create...
```

### 確認

```
ssh node
```

```
ps aux |grep \[r\]edis-server
```

```
```

起動してるので、一応、アクセスしてみる。

```
redis-cli
```

```
```

## LWRP を使った cookbook

### LWRP とは

 * [こちら](https://speakerdeck.com/d_higuchi/lets-use-lwrp-should-we)と[こちら](https://speakerdeck.com/d_higuchi/lets-use-lwrp-should-we)が分り易い資料！
 * Lightweight Resources and Providers の略
 * 独自リソースを定義するフレームワーク
  * resource とはあるべき姿
  * providor とはあるべき姿にする為の手順

では、前述のベタなレシピを LWRP でリファクタリングしてみる...

### resource の定義



### providor の定義

### こんな感じになりました

### node に適用してみる

## Why use LWRP ?

## まとめ
