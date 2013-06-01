# cookbook の書き方諸々〜 redis をソースコードからインストールしたホストを serverspec でテストする 〜

## 概要

 * [前回](http://inokara.hateblo.jp/entry/2013/06/01/104246)セットアップしたホストを serverspec でテストする

## 参考

今回は下記を参考にさせて頂きました。

 * [http://serverspec.org/](http://serverspec.org/)
 * [serverspecでZabbixサーバの稼働テストを書いてみた](http://d.hatena.ne.jp/ike-dai/20130514/1368534178)
  
## serverspec についての認識

 * [こちら](http://mizzy.org/blog/2013/03/23/1/)に書かれているようにサーバー上で稼働しているサービスや開いているポート等をテストするツール
 * 対象となるホストに対して SSH でアクセスするので物理サーバーでも仮想サーバーでもテストが可能

## serverspec でテストする

### serverspec をインストール

gem でインストールする。

```
sudo gem install serverspec --no-ri --no-rdoc
```

### SSH 関連の設定

SSH で対象のホストにアクセスする場合、root ユーザーでパスワード入力なしでログインするか sudo でルートになるユーザーでのパスワード入力なしでのログインが必要なことから運用する現場で設定の方法を検討する必要があると思われる。ちなみに、検討中の運用としては...

 * 仮想サーバーの場合には仮想サーバーのテンプレートに予め serverspec を実行するユーザーのアカウントを作成し鍵の設定をしておく
 * 物理サーバーの場合にはサーバー設定の際に serverspec を実行するユーザーのアカウントを作成し鍵の設定を行う

今回は serverspec を実行するユーザーは `hoge` を使う為 `hoge` ユーザーで対象ホストにアクセスすることになる。ということで...

対象ホストに `hoge` ユーザーを作成し、パスワードなしでログイン出来るようにしておく。また、sudo が利用出来るように `hoge` ユーザーは wheel グループに所属させておき、`/etc/sudoers` は下記のように設定しておく。

```
%wheel  ALL=(ALL)       ALL
```

### テストする項目をピックアップ

今回テストする項目としては...（多少冗長かもしれないけど...）

 * redis-server がインストールされている（ファイルとして存在しつつ、実行権限がある）
 * redis-cli がインストールされている（ファイルとして存在しつつ、実行権限がある）
 * redis-server サービス（プロセス）が起動している
 * redis-server の Listen ポート（6379）がオープンになっている

この位はテストしたい。

[こちら](http://serverspec.org/tutorial.html)でテスト出来るリソースを確認することが出来る。

### ピックアップしたテストを元にしてテストを書く

以下のように書いてみた。

```ruby
require 'spec_helper'                           
                                                
describe file('/usr/local/bin/redis-server') do 
  it { should be_file }                         
  it { should be_executable }                   
end                                             
                                                
describe file('/usr/local/bin/redis-cli') do    
  it { should be_file }                         
  it { should be_executable }                   
end                                             
                                                
describe service('redis-server') do             
  it { should be_running   }                    
end                                             
                                                
describe port(6379) do                          
  it { should be_listening }                    
end                                             
```

### テストしてみる

sudo が利用出来るユーザーでのテストを行う場合には下記のように[実行するとのこと](http://serverspec.org/tutorial.html)。

```
cd ${serverspec}
ASK_SUDO_PASSWORD=1 rake spec
```

以下のようにテストが終了する。

```
/usr/local/rbenv/versions/1.9.3-p392/bin/ruby -S rspec spec/xxx.xx.xx.xx/redis_spec.rb
Enter sudo password:                                                                 
......                                                                               
                                                                                     
Finished in 1.13 seconds                                                             
6 examples, 0 failures                                                               
```

ちなみにホストの radis-server を停止した状態でテストすると...

```
/usr/local/rbenv/versions/1.9.3-p392/bin/ruby -S rspec spec/xxx.xx.xx.xx/redis_spec.rb
Enter sudo password:                                                                  
....FF                                                                                
                                                                                      
Failures:                                                                             
                                                                                      
  1) Service "redis-server"                                                           
     Failure/Error: it { should be_running   }                                        
     # ./spec/xxx.xx.xx.xx/redis_spec.rb:14:in `block (2 levels) in <top (required)>' 
                                                                                      
  2) Port "6379"                                                                      
     Failure/Error: it { should be_listening }                                        
     # ./spec/xxx.xx.xx.xx/redis_spec.rb:18:in `block (2 levels) in <top (required)>' 
                                                                                      
Finished in 2.64 seconds                                                              
6 examples, 2 failures                                                                
                                                                                      
Failed examples:                                                                      
                                                                                      
rspec ./spec/xxx.xx.xx.xx/redis_spec.rb:14 # Service "redis-server"                   
rspec ./spec/xxx.xx.xx.xx/redis_spec.rb:18 # Port "6379"                              
rake aborted!                                                                         
/usr/local/rbenv/versions/1.9.3-p392/bin/ruby -S rspec spec/xxx.xx.xx.xx/redis_spec.rb
 failed                                                                               
                                                                                      
Tasks: TOP => spec                                                                    
(See full trace by running task with --trace)                                         
```

上記のように見事にエラーとなる。

## まとめ

 * serverspec を使うことでホストのあるべき状態をテストすることが出来た
 * cookbook のテストというよりもサーバー設定そのものをテストするツールとして幅広く運用を考えていきたい！
