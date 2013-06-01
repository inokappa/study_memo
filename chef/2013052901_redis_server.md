# cookbook の書き方諸々〜 redis をソースコードからインストールしてみる（1） 〜

## 概要

 * Chef 推しなのにそもそも cookbook をまともに書けてない自分を戒めたい
 * テーマとして redis サーバーをセットアップする cookbook を書く
 
## 参考

検証にあたっては @sawanoboly さんが書かれた、下記を参考にさせて頂きました。

  * [ChefでSourceから何かをインストールするCookbookのウォークスルー](http://qiita.com/items/9c09afc49f5229f646ed)
  * ["LWRPによる"続・ChefでSourceから何かをインストールするCookbookのウォークスルー](http://qiita.com/items/e4d840da4d91b0379c65)
  
## ちなみに redis サーバーについて

  * memcached と似ている KVS
  * でも、レプリケーションが出来る
  * 様々なデータ構造が扱える
  * データの永続化が出来る
  * 詳しくは...[こちら](http://redis.io/)の...[コマンド各種](http://redis.io/commands)

## 普通の cookbook

### まずはこう書く

`redis_1_cookbook/site-cookbook/recipes/resdis_install.rb`

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

自己評価で 30 点。

## 一部の情報を attribute で定義してみる

### 最初の recipe の問題点

最初に掲げた recipe の問題点として考えられるのが、以下のような情報を recipe にベタ書きしてしまっている点。

 * バージョン番号
 * インストールパス
 * ソースコードの URL
 
等、流動的な情報（バージョン番号）や環境によって差異が生まれやすい情報を recipe に直接記載することで cookbook の汎用性をものの見事に損ねていると思われる。

では、上記の情報を attribute を使って外出しにしてみる。

### attribute

`redis_2_cookbook/attributes/default.rb`

```ruby
# 作業用ディレクトリ
default['redis']['work_dir'] = '/usr/local/src/redis/'

# ソースコードの URL
default['redis']['source_ver_num'] = '2.6.13'
default['redis']['source_url_path'] = 'http://redis.googlecode.com/files/'
default['redis']['source_file_name'] = 'redis-#{default['redis']['source_ver_num']}.tar.gz'

# redis-server のインストールパス
default['redis']['server_install_path'] = '/usr/local/bin/redis-server'
```

こんな感じに流動的な情報については attribute に書いてみた。

### recipe

上記の attribute を受けての recipe は下記のようにに書き換えた。

`redis_2_cookbook/recipes/redis_install.rb`

```ruby
# Debian 系と redhat 系の分岐が必要だけど...
#package "tcl8.5" do
#  action :install
#end

# 作業用ディレクトリの作成
directory node['redis']['work_dir'] do
  action :create
  not_if "ls -d #{node['redis']['work_dir']}"
end

# 最新のソースコードを取得
remote_file node['redis']['work_dir'] + node['redis']['source_file_name'] do
  source node['redis']['source_url_path'] + node['redis']['source_file_name']
  not_if "ls #{node['redis']['server_install_path']}"
end

# ソースコードのアーカイブを展開して make && make test && make install
bash "install_redis_program" do
  user "root"
  cwd node['redis']['work_dir']
  code <<-EOH
    tar -zxf #{node['redis']['source_file_name']}
    cd #{::File.basename(node['redis']['source_file_name'], '.tar.gz')}
    make
    make install
  EOH
  not_if "ls #{node['redis']['server_install_path']}"
end

# サービスを起動する
bash "start_redis_server" do
  user "root"
  cwd "/usr/local/bin"
  code <<-EOH
    #{node['redis']['server_install_path']} &
  EOH
  not_if "ps aux | grep \[r\]edis-server"
end
```

## node に適用してみる

### テスト

`knife cookbook test` してみる。

```
knife cookbook test redis_2_cookbook -o ./
```

```
checking redis_2_cookbook
Running syntax check on redis_2_cookbook
Validating ruby files
Validating templates
```

問題無さそう。


`foodcritic` してみる。

```
foodcritic redis_2_cookbook
```

```
FC008: Generated cookbook metadata needs updating: redis_2_cookbook/metadata.rb:2
FC008: Generated cookbook metadata needs updating: redis_2_cookbook/metadata.rb:3
```

こちらはとりあえず無視。

### Chef Server にアップロード

Chef Server は [chef-zero](https://github.com/jkeiser/chef-zero) を使う。手軽に使えるのがマジ良い。

```
knife cookbook upload redis_2_cookbook -o ./
```

### 適用

knife xenserver を使って node に適用。

```
knife xenserver vm create --vm-template 886cd33d-380d-ac9a-83ab-ba0a00c4b7e1 --vm-name redis2 --vm-networks 'Pool-wide network associated with eth0' -r "recipe[setup
]","recipe[redis_2_cookbook::redis_install]" -d chef-full
```

`setup` という cookbook も一緒に適用。この cookbook は `make` や `gcc` をインストールする cookbook 。

### 確認

node にアクセスしてプロセスを確認。

```
# ps aux | grep \[r\]edis-server
root      3072  0.0  0.4  40456  2120 ?        Sl   09:27   0:02 /usr/local/bin/redis-server
```

ついでに redis-server にアクセスする。

```
# redis-cli
redis 127.0.0.1:6379> help
redis-cli 2.6.13
Type: "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit
redis 127.0.0.1:6379> 
```

## まとめ

 * 少なくとも attribute を使ったほうが良さそう...
 * 次は LWRP を試してみる

## github

 * [https://github.com/inokappa/redis_1_cookbook.git](https://github.com/inokappa/redis_1_cookbook.git)
 * [https://github.com/inokappa/redis_2_cookbook.git](https://github.com/inokappa/redis_2_cookbook.git)
