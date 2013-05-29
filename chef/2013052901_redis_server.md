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

なんの技も使わない（使えない）ベタな書き方。

`redis/site-cookbook/resdis.rb`

```ruby
# 作業用ディレクトリの作成
directory '/tmp/redis' do
  action :create
  not_if "ls -ld /tmp/redis"
 end

#  redis のアーカイブを取得する
remote_file "/tmp/redis/redis-2.6.9.tar.gz" do
  source "http://redis.googlecode.com/files/redis-2.6.9.tar.gz"
  not_if 
  notifies :run, "bash[install_program]", :immediately
end

# アーカイブを展開してインストール
bash "install_program" do
  user "root"
  cwd "/tmp/redis"
  code <<-EOH
    tar -zxf redis-2.6.9.tar.gz
    (cd redis-2.6.9/ && make test && make && make install)
  EOH
   not_if "ls /usr/local/bin/redis-server"
end

# redis サーバーを起動する

```

この書き方のダメな点としては、色々な resource がベタ書きで汎用性が非常に低い点が挙げられる。

### node 適用してみる

## LWRP を使った cookbook

### LWRP とは


### resource の定義


### providor の定義

### こんな感じになりました

### node に適用してみる

## Why use LWRP ?

## まとめ