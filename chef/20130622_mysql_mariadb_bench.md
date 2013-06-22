# MySQL と MariaDB を単純に比較してみた

## 概要

 * MySQL と MySQL を fork して作られた MariaDB の性能に関して、それぞれインストールしたての状態の性能について比較してみた
 * セットアップとテストの実行は chef の cookbook を使う
 * テストはには `mysqlslap` を使う

## 環境

 * amazon Linux
 * t1.micro

## まずは cookbook を書く

### MySQL のセットアップ

```
knife cookbook create mysql_install -o ./
```

`mysql_install/recipes/mysql.rb` は以下のような感じで書く。

<script src="https://gist.github.com/inokappa/5835562.js"></script>

一応、シンタックスの確認はやっておく。

```
knife cookbook test mysql_install
```

### MariaDB のセットアップ

```
knife cookbook create mariadb_install -o ./
```

yum のリポジトリ追加から始めるにあたり `cookbook_file` リソースを使うので `mariadb_install/files/default/mariadb.repo` を以下のように用意する。

<script src="https://gist.github.com/inokappa/5835565.js"></script>

インストールのレシピ `mariadb_install/recipes/mariadb.rb` は以下のように用意する。

<script src="https://gist.github.com/inokappa/5835575.js"></script>

### ベンチマークの cookbook

```
knife cookbook create db_benchmark -o ./
```

以下のようなシェルを書いてからの `cookbook_file` で設置してから `execute` で叩かせるのでまずは `` に設置する。	
 
<script src="https://gist.github.com/inokappa/5835577.js"></script>

上記のシェルの場合 `--auto-generate-sql-load-type=read` とあるので読み込み性能のみのベンチマークを取得する。

レシピは `db_benchmark/recipes/bench.rb` に書く。

<script src="https://gist.github.com/inokappa/5835580.js"></script>

## 実行！

### knife cookbook upload する

cookbook を Hosted Chef に登録する。

```
kappa@x1carbon:~/git/chef-repo$ knife cookbook upload mysql_install -o ./
Uploading mysql_install  [0.1.0]
Uploaded 1 cookbook.
```
```
kappa@x1carbon:~/git/chef-repo$ knife cookbook upload mariadb_install -o ./
Uploading mariadb_install [0.1.0]
Uploaded 1 cookbook.
```
```
kappa@x1carbon:~/git/chef-repo$ knife cookbook upload db_benchmark -o ./
Uploading db_benchmark   [0.1.0]
Uploaded 1 cookbook.
```

### knife ec2 でインスタンスを作って cookbook を適用（ベンチマークを実行する）

```
knife ec2 server create -S xxxxxxx \
-i ~/Dropbox/xxxxxxx \
-r "recipe[mysql_install::mysql]","recipe[db_benchmark::bench]" \
-I ami-39b23d38 --flavor t1.micro --region ap-northeast-1 --availability-zone ap-northeast-1a -G quick-start-8 -x ec2-user -N mysql -VV
```

```
knife ec2 server create -S xxxxxxx \
-i ~/Dropbox/xxxxxxx \
-r "recipe[mariadb_install::mariadb]","recipe[db_benchmark::bench]" \
-I ami-39b23d38 --flavor t1.micro --region ap-northeast-1 --availability-zone ap-northeast-1a -G quick-start-8 -x ec2-user -N mariadb -VV
```

### error

一応、cookbook の適用まで終わるんだけど、最後に以下のようなエラーが出てしまう...

```
ec2-xx-xx-xx-xx.ap-northeast-1.compute.amazonaws.com [2013-06-22T01:47:13+00:00] FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
DEBUG: received packet nr 144 type 94 len 92
INFO: channel_data: 0 78b
ec2-xx-xx-xx-xx.ap-northeast-1.compute.amazonaws.com [2013-06-22T01:47:13+00:00] FATAL: Net::HTTPServerException: 403 "Forbidden"
```

なんでやろ...ということで、こちらは調査！

## ベンチマークの結果

で、結局、ベンチマークはどうなったかと言うと...

### MySQL

```
Server version: 5.5.31 MySQL Community Server (GPL)
```

```
Benchmark                                                          
        Running for engine innodb                                  
        Average number of seconds to run all queries: 4.553 seconds
        Minimum number of seconds to run all queries: 4.511 seconds
        Maximum number of seconds to run all queries: 4.608 seconds
        Number of clients running queries: 3                       
        Average number of queries per client: 3333                
```

### MariaDB

```
Server version: 5.5.31-MariaDB MariaDB Server
```

```
Benchmark
        Running for engine innodb
        Average number of seconds to run all queries: 4.686 seconds
        Minimum number of seconds to run all queries: 4.638 seconds
        Maximum number of seconds to run all queries: 4.732 seconds
        Number of clients running queries: 3
        Average number of queries per client: 3333
```

ほんの少しだけ MariaDB 遅いのかなと。誤差の範囲かもしれないし、単純に `read` の性能しか計測していないので一概に優劣はつけられない。

## まとめ

 * cookbook 力が足りない...
 * さくっと検証するには ec2 はかなり便利！
 * でも knife ec2 の謎のエラーはなんだろ...
 * 引き続き MySQL と MariaDB の比較もやっていきたい

## 参考

 * [mysqlslapを使ってRDSのMySQLについて各クラスのパフォーマンス測定](http://dev.classmethod.jp/cloud/aws/amazon-rds-performance-test-by-mysqlslap/)
 * [MariaDBをCentOS 6にyumでインストールする方法](http://www.e-agency.co.jp/column/20130208.html)
