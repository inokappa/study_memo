serverspec と jenkins でサーバー監視を試してみた
==

Chef Casual Talks Vol.3

はじめに
==

 * chef の話から離れてしまいますがご了承下さい
 * serverspec と jenkins を使ってサーバーの監視を行なってみたお話です

自己紹介
==

 * 川原洋平（@inokara）
 * 都内でインフラエンジニアをやってます
 * Chef 好きですが永遠の初心者です

きっかけ
==

 * とある案件（サーバー 2 台）
 * 絶対イケるサービス（担当者談）だからいろいろ心配
 * 監視も設定して欲しいな

検討
==

 * zabbix とか nagios？
  * たった二台の為に設定とか面倒くさい
 * serverspec で試してみよう
  * それなら定期的に働いてくれる jenkins に serverspec を叩かせよう

構成
==

以下のような感じ。

![](http://cdn-ak.f.st-hatena.com/images/fotolife/i/inokara/20130617/20130617171257.png)

図がちっちゃい。

準備（1）
==

まずは serverspec の準備をする。

試しに `MySQL` の稼働状況を下記のように作成した。

<script src="https://gist.github.com/inokappa/5737542.js"></script>

他にも `Apache` が稼働していたら `Apache` 用のテストを書く。

準備（2）
==

も少しだけ serverspec 側を準備する。

監視先のホストでのパスを `spec_helper.rb` に設定する。

<script src="https://gist.github.com/inokappa/5737514.js"></script>

`c.path = '/sbin:/usr/sbin'` を追加する。

準備（3）
==

監視対象ホストの設定を行う。

`sudo` で利用出来るパスと `jenkins` ユーザーがパスワード入力無しで `sudo` コマンドが利用出来るように設定する。

<script src="https://gist.github.com/inokappa/5737533.js"></script>

準備（4）
==

jenkins にジョブとして登録する。

 * `新規ジョブ作成` をクリックして `フリースタイル・プロジェクトのビルド` を選択する。
 * `ビルド・トリガ` は `定期的に実行` にチェックを入れて `スケジュール` を設定する。

<p><span itemscope itemtype="http://schema.org/Photograph"><img src="http://cdn-ak.f.st-hatena.com/images/fotolife/i/inokara/20130609/20130609130911.png" alt="f:id:inokara:20130609130911p:plain" title="f:id:inokara:20130609130911p:plain" class="hatena-fotolife" itemprop="image"></span></p>

また、 `シェルの実行` に以下のコマンドを記載する。また、必要に応じて `ビルド後の処理` を設定する。

こんな感じになる（1）
==

図のようにジョブ（テストと称した監視）が登録されている。

<p><span itemscope itemtype="http://schema.org/Photograph"><img src="http://cdn-ak.f.st-hatena.com/images/fotolife/i/inokara/20130609/20130609131036.png" alt="f:id:inokara:20130609131036p:plain" title="f:id:inokara:20130609131036p:plain" class="hatena-fotolife" itemprop="image"></span></p>

こんな感じになる（2）
==

コンソール出力は下記の通り。

<p><span itemscope itemtype="http://schema.org/Photograph"><img src="http://cdn-ak.f.st-hatena.com/images/fotolife/i/inokara/20130609/20130609131115.png" alt="f:id:inokara:20130609131115p:plain" title="f:id:inokara:20130609131115p:plain" class="hatena-fotolife" itemprop="image"></span></p>

良いと思う点
==

 * サーバー、クライアント共に小難しい監視システムを構築する手間が省ける
 * ssh のポートが空いていれば監視出来る（監視システム用のポートを考慮する必要が無い）
 * クライアント側からの push 監視も可能（cron の結果を post で受け取る）
 * お手軽

検討が必要な点
==

 * jenkins ユーザーでのログインを許可するなどセキュリティ面を検討する必要がある
 * リソース監視は不可能では無いが手間が増える
 * 残念ながらグラフとか書けない
 * OS の設定に依存する部分がある

とは言え
==

お手軽にサーバーの監視をするなら serverspec と jenkins の組み合わせはオススメかも！
