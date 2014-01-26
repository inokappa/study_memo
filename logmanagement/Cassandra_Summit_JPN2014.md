# Cassandra Summit JPN 2014

## Cassandraを含むシステム全体の可視化とパフォーマンス分析

### 登壇者

 * タクトシステムズ / 菅野 正樹

### 概要

 * dynaTrace の宣伝
 * システム全体の可視化
  * パフォーマンス分析（ボトルネックの特定）

### システムの複雑化

 * ログ
  * ログの出力がシステムのボトルネックに
 * 監視ツール
  * サーバー単位での状況を把握することは出来る
  * 処理全体の問題は判断出来ない

### 解決策

 * dynaTrace によりシステム全体の可視化

### dynaTrace の概要

 * 分析
 * アプリ監視
 * Java/.NET/PHP/JavaScript に対応
 * 導入が簡単
 * トランザクションのパフォーマンス情報を収集
 * 負荷が低い（2% くらい）
 * アクセスメソッドの実行回数や時間の把握
 * いいお値段
 
### 仕組み

 * エージェント式でサーバーに情報を送る
 * トランザクションに ID を仕込む
 * 分析結果はクライアントで

### Cassandra への対応

 * 1.0 1.1.x 1.2.x
 * Thrift
 * Datastax 1.0.2 for Cassandra 1.2.x(CQL3)
 * アクセスメソッドのトレース
 * JVM のリソース状況の把握

***

## Cassandra導入事例と現場視点での苦労したポイント

### 登壇者

 * 株式会社ぐるなび / 羽毛田　敦
 
### アジェンダ

 * 自己紹介
 * Cassandra 導入経緯と事例紹介
 * Cassandra 導入メリット、デメリット

### 自己紹介 / 会社紹介

 * トラッキング管理システムの開発、運用担当
 * 月間 10 億 pv
 
### Cassandra 導入の経由

 * レストラン情報 RDB と XML
 * 一部情報を memcached を 2010 年から...
 * データの永続性、データ量、アクセス数増加に耐えれない
 * Cassandra version 0.6 から検証開始
 
#### 2010 年の 7 月

 * 16 ノード構成（Ver 0.6.1）

#### 2011 年 11 月 ver 1.1.15

 * 同一 Cassandra リング内で Keyspace を分割して管理

#### 2013 年 6 月から

 * 70 ノード
 * 3 つのリング（データセンター）
 * 基本データ 42 ノード/ 画像 18 ノード/　バックアップ 10 ノード

### 構成

#### 以前

 * 情報のリアルタイム性
 * データの柔軟性
 * 耐障害性が低い
 
#### Cassandra 導入

 * 動的なページ更新と柔軟な部分更新
 * データの分散と耐障害性
 * 高負荷
 * マルチデータセンタ構成
 * バックアップリングから遠隔地の NAS へバックアップ
 * ヘクター（API）で Cassandra にアクセスしている
 
### トラッキング管理システム

 * ユーザーのアクセス履歴を管理するシステム
 * 小さいデータの頻繁な書き込みが発生
 * 18 ノードで運用
 * 5 億件
 * 1TB
 * 1500 アクセス/sec
 * 無停止のスケールアウト

#### 検証

 * 利用実績と Write の速さを評価して Cassandra を採用

### 利用事例まとめ

 * SPOF がない
 * 柔軟なテーブル定義
 * 無停止スケールアウト
 * 大量データ、大量アクセス

### メリット、デメリット

#### メリット

 * MySQL の容量逼迫（5 億件 / 約 1TB）
 * スケールアップの為に停止せざるえない
 * Cassandra で複数サーバーで絵d−たを管理
 * 無停止でのスケールアウト
 * 高頻度、高速アクセスレスポンスを返してくれる（数msec）

#### デメリット

 * トランザクション制御が出来ない
 * 削除周りが難しい

#### デメリット（1）

 * CAP の AP を採用
 * 整合性が取れない
 * トランザクション制御、排他制御出来ない
 * ZooKeeper （ZooKeeper Lock）を利用して排他制御を行う
 * ZK Lock
 * アプリケーションは ZooKeeper Lock を介してシーケンス番号を確認してロックの状態を判断する
 * Cassandra はクライアントの時刻ズレが問題となる（データにタイムスタンプが付くので）
 * ZooKeeper を使うことで時間がズレてしまう問題を解決出来る
 * ZooKeeper を介すと遅くなる可能性がある
 * ZooKeeper の構成上リーダーとフォロワーがあるリーダーのヒープ障害
 * 他の排他制御もある Cassandra 内にデータ（ロック用のデーブル）を持つ
  * TTL を経過するとロックを無効
  * 同一データの連続ロックで処理遅延が見られる

#### デメリット（データ削除が難しい）

  * tombstone（墓石）という論理削除フラグ
  * SSTable（ディスク）から物理削除
  * データ削除の情報が insert される
  * gc_grace_second（猶予期間）
  * compaction 発生（物理削除）
 * 削除データ復活する...
  * Hinted Handoff
  * gc_grace までに全ノードへの削除伝播が必要
 * 古いデータが消えにくい
  * 大きな SSTable が出力されると Compacion の対象になるには時間がかかる
  * 古いデータ消すには...Major Compaction / Leveed Compaction
  * Tombstone Compaction（Cassandra 1.2 から）
  * TTL を設定したデータの単独 Compaction 発生
 * データ復活の可能性
 * 古いデータの物理削除には時間がかかる
  * ディスク容量に余裕をもたせる

### 質疑応答

 * ローリングアップデート（後で調べる）
  * 異なるバージョンはリングの中で混在出来ない
  * 但し、混在出来るバージョンの範囲がある

## Cassandra100台クラスタを運用して

### 登壇者

 * 株式会社サイバーエージェント / oranie(成田 俊)

### はじめに

 * Version 1.1 系
 * http://www.slideshare.net/TakehiroTorigaki/cassandra-21191674

### 運用システム

 * s.amebame.com
 * ガールフレンド（仮）
 * データ量、負荷から Cassandra を採用
 * SPOF Free
 * read 45000q/sec write 4000qps
 * 35TB
 * 35GB/node
 * 4msc / 0.1
 * 物理サーバー 16 core or 24 core / 64GB / sas600GB RAID 10（SSD や SATA）
 * 15 => 30 => 45 => 60 => 90 => 100（ver 1.1.5-2）
 * 低いレイテンシーとデータ量の対応

### 日々の運用

 * 公式ブログ
 * 構築は Chef fabric jenkins で
 * token の設定のみ
 * Nagios Jenkins Perl（メール、Push 通知）
 * JMX(jolokia.jar) REST API
 * Cassandra status / nodetool / ring changes / job result
 * OS / JVM heap memory / query count latency etc..
 * Cacti
 * Proteus-monitor
 * growthforecast + yohoushi
 * Opscenter（100 台を超えるとキツイ）パッと見る場合
 * 各ノードの細かい情報は yohoushi で
 * Websocket でリアルタイム監視

### オペレーション

 * nodetool cmd(ver 1.1)
 * ring(status は 1.2 から)
 * info
 * cfstats column family 毎の統計情報
 * tpstats
 * compactionstats
 * gossipinfo
 * netstats（repaire 等他のノードとデータをやりとりする Stream 状況を確認）
 * drain
 * decommission
 * move
 * removetoken
 * flush
 * repair
 * cleanup（重複データを削除※論理削除）
 * compact
 * scrub
 * upgradesstables

#### node down

 * cassandra restart

#### slow down

 * 再起動
 
#### ハードウェア障害

 * yukim @ github
 * クラスタの健全性

#### データの肥大化

 * 適切な node 追加
 * クラスタ分割（今後の予定）
 
#### バックアップ

 * nodetool snapshot で生成されたハードリンクを一定期間ノード上で保存＋外部ストレージにバックアップ

### バッドノウハウ

#### リペア時のデータ肥大化

 * repair が完了後、肥大化したカラムファミリに対して cleanup を...

#### スキーマ設計は重要

 * JVM のヒープサイズ

#### 連続稼働するとレイテンシーが下がる

### H/W の安定性の重要

 * SAS Disk ではなく SSD を使っていた
 * RAID カードとの相性
 * 十分な検証を行っていないハードウェアは使わない

### ここは困るよ Cassandra

#### repair が重い、データが肥大化

 * node 障害からの復旧は時間がかかる
 * 300GB 〜 400GB /1 node
 * compaction 5 日
 * compaction がカラムファミリ単位で並列化されている
 * multithreaded_compaction は不安定

#### slow query log が無い

 * query trace は 1.2 系から
 * アプリケーション側で slow log を実装しましょう

#### nodetool の出力が微妙に違う

 * 監視に困る
 * nodetool cfstats は json で出力出来るように

#### データダンプ

 * snapshot

### 主観的なクラスタ設計

 * 1 cluster の node 台数 50
 * repair の実行時間が長い
 * node の追加も一気に出来ない
 * 10 台ずつ token レンジの標準化で move が必要だけど重い
 * データ量 100 〜 200GB /node
 * EBS の場合は 1TB max なので注意する
 * EBS ストライピング
 * 50GB /1カラムファミリ
 * wide row は NG / アクセスが集中して分散システムの意味がない
 * アーカイブ、ヒストリー系等のデータ量が多い部分にオススメ

### 今後の予定

 * 2.0 系、2.1 系
 * ネットワークトポロジストラテジーの導入
 * 監視系の統合
 * netflix の各種ツール
 * 2.0 系から CQL が強化等
 * RDBMS から Cassandra への置き換えは慎重に...

### 質疑応答

 * レプリケーションファクタ 3（隣接ノードが 2 台以上落ちなければ大丈夫）
 * どんなデータ？
  * ユーザーログイン情報
  * プラットフォームのデータほとんどは Cassandra に
 * ネットワークトラフィックの障害は無い（H/W サーバー）
 * Fusion IO は検討したけど Disk I/O にボトルネックにならないので採用は検討してない
 * row に大量の read をかけると gosship の time out が発生
 
***

## Large nodes with Cassandra

### 登壇者

 * THE LAST PICKLE / Aaron Moton
 
### Lerge Node

 * 500GB over /1 node
 * 一台のノードにどのくらいのデータをのせられるか？
 * row は 10 億程度

### 1.2 以前

 * 大型ノードは大変だった
 * Bloom Filter Size
 * 1 億 row
 * 10 億 row 1.2 GB のストレージ
 * compression meta data の容量
 * 10 億 row / 230MB
 * JVM のヒープサイズ　8GB 以上
 * bootstraping
  * token range
  * stream_throuhupput... 

### 1.2 で

#### DiskManagement

 * ディスクの管理が難しい
 * 500GB => 数 TB
 * 75% キャパを超えると遅くなる
 * RAID0
 * RAID10
 * リペアが難しい
 * Validation Compaction
 * SSTable

#### Mermoy Management

 * Bloom Filter
 * Reduce Index Sample size
 * chunk_length_kb
 * MAX_HEAP_SIZE 12GB
 * HEAP_NEWSIZE 1.2GB

#### Moving Node Work Around

 * nodetool setstreamthroughput

#### Repair Work Arounds
#### Compaction Word Arounds

### 1.2 => 2.0

#### bootstrap

 * VirtualNode

#### Disk Layout

 * JBOD 設定
 * disk_failure_policy (ignore / stop（ノードがクラスタから自動的に切り離す） / best_effort（ディスクの利用を止めて、read /write プロセスは生きる）)
 
#### Repaire

 * 差分リペア
 * 1 億 row => 4 億 row
 
### Q and A

 * JVM
 * JNA

***

## Cassandraでここまで出来る！こう使える！　技術ネタからEC、監視、M2M案件まで

### 登壇者

 * ウルシステムズ / 岸本 康二

### はじめに

 * RDB に飽きました...
 * 業務向けになると高い
 * 枯れた技術

### NoSQL

 * Cassandra だ
 * SPOF が無い
 * スケーラビリティが高い
 * OSS
 * エンジニヤリング魂
 
### cassandra でどこまで出来るか？

 * 業務処理に難
 * トランザクション
 * 本気で業務システムを作ってみたぜ
 * BlueRabbit

### 事例

 * EC サイト
 * リアルタイム視聴・ポイントシステム（write 20000 回/sec）

### Nanahoshi

 * NoSQL にトランザクション、検索機能を付加
 * トランザクションを用いて index を使う
 * Mongos がスケールしない
 * テーブルごとにアクセスクラスタを選択可能
 * Thrift から CQL3 に移行した

### RabbitScope

 * クラウド監視サービス
 * 電話を永遠とかけ続ける
 * 電話だけでサーバーを追加したり出来る
 * NoSQL を考慮して設計は必要
 * M2M は書き込み中心の世界
 
### Q and A

 * クラスタの規模 20 ノード程度
 * http://tocos-wireless.com/jp/tech/M2M.html
 * 単一障害点が無い

## Keyword

 * bootstrap
  * 新規ノードを追加すること
 * rolling update
 * compaction（ファイル圧縮）
  * ファイルの圧縮
 * SSTables
  * Sorted Strings Table
 * JVM
 * JNA
 * Zookeeper
 * Bloom Filter