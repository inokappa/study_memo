# 第二回 elasticsearch 勉強会

***

## elasticssearch の routing 機能

### 自己紹介

 * @johtani さん
 * Solr のコンサルティング
 * Doorkeeper
 * メーリングリスト elasticsearch-jp
 * 書籍（洋書）
 * [資料](http://blog.johtani.info/images/entries/20131112/About_es_routing.pdf)

### インデックスの構成

#### 論理

 * クラスタの中に複数のインデックス
 * Solr と異なる点 type の存在
 * シャードとは小さい index の集まり
 * シャード数は index 作成時のみ設定可能

#### クラスタ構成

 * ノードの集まり
 * レプリカの数は後から調整可能

#### データ登録

 * ドキュメント ID をシャード数で割った数＝登録するシャード番号

#### 検索

 * クラスタ内の各々のノードに検索を投げる

### ルーティング機能

 * ルーティングパラメータを指定
 * ID の代わりに指定された値のハッシュを計算して利用
 * 特定のシャードのみを検索する
 * カンマ区切りで複数指定可能

### スケールアウト

 * elastic
 * シャーディングによるスケールアウト数＝インデックス作成時に決定

### エイリアス

 * インデックスに別名を指定することできる
 * 複数インデックスを 1 つのインデックスとして利用可能
 * 検索可能

## 宿題、宣伝

 * Solr 入門

***

## Elasticsearch 試行錯誤

### 自己紹介

 * @pisatoshi さん
 * 事例
 * [資料](https://speakerdeck.com/pisatoshi/elasticsearch-trial-and-error)

### 背景

 * mBasS ??
 * mobile backend as a service
 * appiaries / Parse
 * couchdb-lucene
 * Java + Elasticsearch へ乗り換え

#### システム構成

 * プライマリデータストアとして ES
  * ノード数 5
  * シャード数 10
  * レプリカ 1

#### 扱っているデータ

 * 業務データ
 * トラッキングデータ
 * 大きくなりがち

### mBaaS として使う為に

 * マルチテナント
 * 更新の即時反映
  * レプリケーションのコストが高い <-> レプリカを減らす（耐障害性が落ちる）
 * ルーティングによる性能向上
  * ドキュメント ID のハッシュ値からシャードを自動選択
  * 検索対象のシャードを絞り込む
  * 検索の負荷を軽減
 * バックアップ
  * ES に登録と同時に MySQL にもデータを登録する
  * バックアップは MySQL のスレーブで行う
  * ES のインデックスのバックアップは取らない
  * インデックスが壊れたということは無い
  * リストアの時間が掛かり過ぎ

### マッピング

 * ES はスキーマレスが売り
 * DynamicMapping
  * 入力データから型を推測して自動的にマッピングを登録
  * マッピング定義が肥大化
  * データ型のコンフリクト
  * 大規模開発では破綻しかねない
 
 * マッピングが肥大化すると
  * インデキシング性能劣化
  * マッピング追加を伴うドキュメント登録が 3 sec（マッピングサイズ 80 MB）
  * 大量データ投入はシングルノードの方が早い
   * 登録後シャードを追加

 * データ型のコンフリクト
  * 検索出来なくなる
  * ソートキーを外すと検索出来るようになる

 * マッピング定義をアプリで制御
  * MultiField 型でマッピング
  * マッピングの最大サイズを固定（肥大化を防止）

### デバッグ、テスト

 * elasticsearch-head
 * bigdesk
 * SENSE（Chrome Extension）
 * Slow query log
  * 閾値 を 0ms にすると各シャード毎の全リクエストを記録
 * ユニットテスト
  * elasticsearch-test 
  * NodeClient

### クラスタ名

 * デフォルトから変更することがオススメ

### 質問

 * 大量データでのテスト
  * 数百万から 3000 万ドキュメント
 * サーバー台数
  * ES だけで 5 台
 * バックアップ
  * アーカイブして戻せる状態にまでは検証したことはある
  * 1.0 まで待つのが吉

***

## kibana 入門

### 自己紹介

 * @y_310 さん
 * from cookpad
 * [資料](https://speakerdeck.com/y310/kibanaru-men)

### なぜ kibana

 * elasticsearch 公式ツール
 * 作った dashboard も ES に保存
 * ストレージ不要！
 * 簡単に使える
 * 軽いデータ


### 使い方

 * 上部 Navigation
 * 行単位でパネルを追加

#### クエリー

 * luene のクエリが掛ける

#### フィルタリング

 * クエリに対する期間の絞込
 * タイプ等

#### save と load

 * ダッシュボード作ったらリロード前に保存する！

#### histgram

 * 時系列データを表示する

#### hits

 * クエリ毎に総ヒット件数をグラフ化

#### Sparklines

 * クエリごとの傾向だけを可視化

#### Terms

 * ES の facets の結果をグラフ化

#### Trends

 * 指定した時点からの値の変化を表示

#### Map

 * facets の結果を地図上で可視化
 * 日本地図は議論中（630）

#### BetterMap

 * 緯度経度をマッピング

#### Table

 * クエリにマッチしたレコードを表示する

#### Text と Colum

### クエリの書き方

 * title:"word"
 * filed 名 : "名前"
 * AND
 * 複数のクエリの結果を比較
 * ちょっとした計算（平均値、最大値、最小値、合計等）

### tips 

#### index と type

 * 1 つの index に異なるスキーマを持つデータを入れることが出来る
 * グラフを重ねて比較することが出来る

#### mapping

 * 自動的に定義される
 * index template _template/logstatsh_template

#### 性能

 * EC2 m1.large x 1 台
 * インデックスサイズが 10GB を超えるあたりで ES が詰まる
 * JVM の GC パラメータチューニング
 * ピーク時で 6Mbps 程度のトラフィックに耐えられる
 * オブジェクトが大量に生成、削除されることで頻繁に Full GC が走っていたのが原因

#### 最新情報

 * github の master
 * ES の blog
 * demo.kibana.org

### 質問

 * 日付が UTC で ES に登録されている
 * ちゃんと Timezone をつけて ES に登録すること
 * とにかく日付（Timezone）には注意が必要
 * ES のインデックスであれば kibana で読み込める
 * faset を利用してグラフ化している

***

## Elasticesearch と kibana を使った可視化テクニック

### 自己紹介

 * @yoshi_ken さん
 * リブセンス
 * [資料](http://www.slideshare.net/y-ken/elasticsearch-kibnana-fluentd-management-tips)

### kibana3

 * fluent-plugin-geoip

### システム構成

 * フロントエンド
  * Nginx
 * 検索システム
  * ES
 * ログアグリゲータ
  * fluentd
  * geoip
  * elasticsearch

### Fluentd との連携サンプル

 * Apache のログを IP をベースに地域情報の抽出を行う
 * fluent-plugin-geoip
 * munin との連携（★）
 * dstat との連携（★）
 * twitter との連携

### JDBC の利用

 * ES にデータを流し込む

### 運用ノウハウ

 * エラーログ
 * ページの応答速度
 * HTTP のステータスコード
 * GoogleBot 等
 * ログイン
 * 広告露出数

 * ES_HEAP_SIZE
  * 潤沢に設定
  * 4GB サーバーで 3GB 割り当てる（ES 専用サーバー）

 * cluster.name
 * node.name
 * discover,zen…-> false 一台で使う時
 * http.max_content_length -> 増やそう 100MB（デフォルト）より増やす

 * バックアップ
 * 保守ツール
  * 1 日ごとにインデックスが増える
  * インデックスを削除するツール

 * elseql
  * SQL で ES に問い合わせ出来る 

#### fluentd との連携時の tip

 * 要資料確認

#### ディスクフル

 * 要資料確認

***

## fluent-plugin-kibana-server

### 自己紹介

 * @repeatedly さん
 * Treasure Data
 * [資料](https://t.co/4G9hQv8ddQ)

### fluentd の中で kibana を動かす
 
 * js で実装されている
 * input プラグインで作られている
 * kibana_server
 * type kibana_server だけで動く
 * logstash と同じアプローチ
 * ES の index を自動で消してくれる

***

## Elasticsearch auth プラグインについて

 * @shinsuke_sugaya さん
 * [資料](http://t.co/uce0VmFDxj)

## アクセス制御

 * アクセス制御したい
 * クラスタの _shutdown 等特定ユーザーに許可したい
 * ユーザー管理
 * REST API のアクセス管理 
 * ログイン、ログアウト
 * 情報はインデックスに登録される
 * プラグインのインストールは簡単
 * 前方一致のパスで判断する
