# cookbook の書き方諸々〜 redis をソースコードからインストールしてみる（3） 〜

## 概要

 * Chef 推しなのにそもそも cookbook をまともに書けてない自分を戒めたい
 * テーマとして redis サーバーをセットアップする cookbook を書く
 * LWRP で書いてみる
 
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

## LWRP を使った cookbook

[前回](http://inokara.hateblo.jp/entry/2013/06/01/104246)は attribute を使った cookbook を書いた。

### LWRP とは

 * [こちら](https://speakerdeck.com/d_higuchi/lets-use-lwrp-should-we)と[こちら](https://speakerdeck.com/d_higuchi/lets-use-lwrp-should-we)が分り易い資料！
 * Lightweight Resources and Providers の略
 * 独自リソースを定義するフレームワーク
  * resource とはあるべき姿
  * providor とはあるべき姿にする為の手順

ということで、[前回](http://inokara.hateblo.jp/entry/2013/06/01/104246)のベタな recipe を LWRP でリファクタリングしてみる...

### resource の定義

### providor の定義

### こんな感じになりました

### node に適用してみる

## Why use LWRP ?

