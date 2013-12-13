# fluentd Casual Talks #3

 * 2013/12/13 19:00 ~ 21:00
 * DeNA

## @tagomoris さん/ norikraの話

 * [資料](http://www.slideshare.net/tagomoris/fluentpluginnorikra-fluentdcasual)
 * fluent-plugin-norikra

###  なに？

 * スキーマレスイベントストリーム（Fluentd  の上を流れているもの）
 * SQL 
 * GPL
 * 3 分でインストールで利用可能
 * ストリーム対して SQL で検索出来る
 * 結果も JSON で返ってくる

### 詳細

 * time_batch => 時間でバッチ処理をする
 * クエリが追加されたらその結果も追加で出力される
 * 配列アクセス $0,$1 で書く
 * 超簡単

### どうやって使うか？

 * numeric_monitor
 * datacounter

### 3 つのプラグイン

 * in_norikra
 * out_norikra
 * out_norikra_filter
  * とりあえず使ってみようのヒト向け

***
## @sonots さん/  (DeNA枠)Fluentdでshadowサーバ用意したら捗った話

 * [資料](http://www.slideshare.net/sonots/shadow-fluentdcasualtalks-20131213)
 * Haikanko
 * Fluentd クラスタ管理ツール（Fluentd の設定ファイルを自動生成）
 * Yohoushi
 * どんどん課金して下さい

### DeNA の導入事例

 * ログ監視
 * ログの可視化
 * fluent-agent-lite
 * agent -> worker -> serializer (watcher node)
 * 152 プロセス
 * 4 台
 * 10 万行/sec
 * fluentd.conf は 2 万行

### 困っているんです

 * 本番に入れたらなんかおかしい
 * そこで shadow サーバー

### shadow サーバーとは？

 * kage
 * Geest
 * Delta
 * fluentd ならば type copy して shadow サーバーにログを流す
 * メモリの使用量が増えた！

### 性能検証

 * dummy_loh_generator（単純にダミーのログを吐くだけ）
 * 読み込み
  * fluent-agent-lite 56 万行/sec
  * in_tail 15 万行/sec
  * in_tail_ex 15 万行/sec
  * v11 15 万行/sec
 * チャンクにためて TCP を開いておくって、閉じる

### 送信の検証

 * buffer_chunk_limit
  * 1m
 * keep_forward
 * fluent-agent-lite
 * out_forward はなぜ詰まる？

### まとめ

 * 送信制御は out_forward で行われる
 * fluent-agent-lite は素晴らしい
 * out_forward 安定パラメタ
  * flush_interval 0s
  * chunk_limit 1m…
  * num_therads は大きめに 

***

## @stanaka さん / fluentd go implementation (仮)

 * [資料](://speakerdeck.com/stanaka/alternative-fluentd-implementation-in-go)
 * Go で Fluentd を実装した
 * from hatena
 * LTSV
 * immutable infrastructure / Docker

### はてなでどんなふうに使っているのか？

 * LogServer(Mongo)
 * elasticsearch(kibana)

### Go

 * Go コン（Go カンファレンス）
 * 2009 年リリース
 * 2013 年 Go 1.2
 * コンパイルして使う
 * 型を推論
 * コンカレンシー（goroutine / channel）
 * パフォーマンスが高い（C や C++ にはかなわないけど、Perl 等より一割速い）
 * メモリ消費量は少ない
 * コードフォーマット、テスト・ツール
 * ライブラリが豊富
 * Docker / Packer

### ik

 * @moriyoshito
 * v10 の設定がそのまま読み込める
 * プラグインシステム（現状はハードコード）
 * インストールは簡単
 * メモリの消費量が少ない（fluentd の 6 分の 1 程度）

***

## @okahashi117 さん / Windows版fluentdで幸せになれますか

 * Windows 版！

### 幸せになれますか？

 * 企業では Windows が多い
 * ニーズがありそう
 * 0.10.35 ベース
 * demo
 * cool.io は 1.2
 * バッファはメモリのみ対応

### エンジン

 * cool.io 1.2
 * シグナルハンドラ（HUP,USR1をインストールしない）
 * fork を使わない
 * そのまま非 WIndows 環境で動く

### プラグイン

 * inode 値（inode の一意性は NTFS では保証されない）
 * ファイルの open 処理
 * Windows 固有の共有モード指定サポート

### 是非お試し下さい

 * 是非、お試しください
 * 本番で使っちゃダメ

## @kzk_mover さん / Fluentd のモニタリングのはなし

 * [資料](http://www.slideshare.net/treasure-data/treasure-agent-monitoring-service)
 * ちゃんとモニタリングをやりましょう
 * データ解析の世界をシンプルに
 * 世界中で使われている！

### モニタリング

 * プロセス監視
 * ポート
 * システム情報
 * 内部情報

### モニターエージェント

 * 内部情報を取得出来る

### モニタリングサービス

 * クラウドサービス
 * fluent-plugin-monitoring
 * td-agent v1.1.18 には含まれている
 * 統計情報の定期送付
  * システムの統計情報
 * type td_monitor_agent
 * アカウントが必要
 * instance_id （ホスト名等 EC2 の場合にはインスタンス ID を追加）
 * td_counter
 * CPU / memory / buffer etc...
 * アラート通知

### 提供形態

 * フリー
 * ベータリリース
 * 要フィードバック

## @kenjiskywalker さん / 増えすぎた設定ファイルの行数をどうするかみたいな話をChefのcookbookとくっつけて（LT 枠）

 * [資料](https://speakerdeck.com/kenjiskywalker/large-td-agent-dot-conf-with-chef)
 * td-agent.conf を chef で管理する
 * include ディレクティブ
 * include_recipe
 * S3 の情報を data_bags に置く

## @yoshi_ken さん / Fluentd as a Middleware Engine

 * [リリースノート](://github.com/y-ken/yamabiko/releases/tag/2013.12.13)
 * MySQL のテーブルを Elasticsearch にレプリケート？
 * Yamabiko（既存の Ruby や td-agent と鑑賞しない）
 * MySQL から Elasticsearch にデータを流す（非同期に）
 * fluent-plugin-mysql-repilicator
 * fluentd + fluent-plugin-mysql-replicator をデーモン化

### yamabiko の活用方法

 * RPM パッケージを配布中
 * gem でもインストール可能
 * 28 テーブル
 * 4 ギガテーブル
 * 1vCPU/memory 1GB
 * MySQL との並行運用
 * 全文検索

## @bash0C7 さん / ご家庭でfluentd

 * [関連記事](http://qiita.com/bash0C7/items/a8aea4810cc5ec6f1eb9)
 * pixiv
 * 38 億 pv
 * Macbook Air to ラズベリー・パイ(AWS SQS から pop)
 * アイドルの画像を集めて tumblr に投稿
 * おお！楽しそう！
 * 楽しく使いましょう

## @kazegusuri さん / OutputとBufferedOutputの間をうめる

 * [資料](https://speakerdeck.com/kazegusuri/outputtobufferedoutputfalsejian-falsehe-ka)
 * BufferdOutput プラグイン
 * Output プラグイン
 * 二つの間を満たすなにか？
 * fluent-plugin-bufferize
 * Outpu プラグインに以下の機能を付与する

## @choplin さん / postgres関連の何か

 * PostgreSQL
 * JSON 型 + fluent-plugin-pgjson
 * pg_msgpack
 * mongoDB は遅い（一部の指標を除く)

## @frsyuki さん / v11 の話

 * [資料](http://www.slideshare.net/frsyuki/whats-new-fluentd-casual-talks-3?utm_source=dlvr.it&utm_medium=twitter)

### 無停止再起動

 * woker の中で tcp listen
 * v11 で supervisor で tcp listen

### multi process 化

 * <worker> タグ
 * それぞれの worker の中で異なるプラグインを動かすことが出来る

### エラーストリーム

 * <label @ERROR> タグ

###  プラグインのバージョン管理

 * Gemfile を /etc/td-agent に置くとプラグインのバージョン管理が出来る
 * Gem "fluentd" って書くと新しい fluentd を使うことが出来る

### ログレベル

 * プラグインごとに log_level を分けることが出来る

### 変数を埋め込める

 * タグの書き換えが要らない

### リリース

 * gem install fluentd -v 0.11.0.preview1

***

## 試す！

 * 基本、全部試したい
 * fluent-plugun-norikra
 * fluent-plugin-monitoring
 * Haikanko
 * Yamabiko
 * v11
