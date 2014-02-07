# 第三回 elasticsearch 勉強会

## Florian Schilling, Elasticsearch Inc, Title：Geohashing with Elasticsearch

### overview

 * 多分、位置情報と Elasticsearch ってことかな
 * http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/mapping-geo-point-type.html
 * http://ja.wikipedia.org/wiki/%E3%82%B8%E3%82%AA%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5

### Geo Types

 * type geo_point

### Getohashing

## 株式会社イプロス　外山　寛さん　@toyama0919 タイトル：「AWSで構築するsharding」

### PRODUCTION 環境で使った

 * パフォーマンス

### elasticsearch-cloud-aws

 * elasticsearch を EC2 で動かす為のプラグイン
 * multicast 的なことが出来る
 * EC2 の tag が使えたりする（クラスタを作れる）
 * SecurityGroup でクラスタ化
 * tag で AutoDiscovery
 * ZenDiscovery を無効

### Routing

 * 大事！
 * index に対して有効 user=1 => index1
 * RDBMS の index と一緒
 * Routing 無し => 全ての shard を検索する
 * Rouging あり => 1 つの shard を検索する
 * type をまたぐことが出来る
 * データが大きくなると有効
 * index をまたいだ Routing が出来ない（logstash index だと無理）
 * 後付で routing が出来ない

### logstatsh の見解

 * 時系列でインデックスが作られる
 * kibana との連携しやすい
 * fluentd-plugin-elasticsaerch と連携しやすい
 * アプリケーションとの相性が悪い（✕）
 * 長期間（3 年うらい）のログ検索には向いて無い（✕）

### インデックスを 1 つにするメリット、デメリット

 * アプリケーションと連携しやすい
 * routing が生きる
 * バックアップがし辛い（1.0 のスナップショット機能がほしい）
 * リカバリ方法をシビヤに

### fluentd との連携メリット、デメリット

 * elasticsearch がダウンしても fluentd には再送機能がある
 * kibana を使うには小細工が必要
 * kibana の日付フォーマットが厳しい
 * type record_reformer（timestamo をイジる）

### rails との連携

 * tire
 * elasticsearch-rails
 * ライブラリが乱立してる

### まとめ

 * m1.large x 2（ドキュメントサイズが 100GB）

## 株式会社じげん　多田 雅斗さん　@tady_jp タイトル：実サービスでのElasticsearch設定・使用例（仮）（テスト駆動検索のススメ）

 * tadyjp/test_driven_search

### Solr との比較

 * そろそろ Solr とも差が...

### 特徴

 * クラウド時代にマッチしている
 * plugin
 * REST full な index

### 調査

 * 公式サイト
 * 英語の書籍（4 冊出ている）
 * Stack OverFlow
 * 直接クエリを調査

### 使用例

 * 全文検索
 * 複数カラムにまたがった検索
 * 高度なスコアリング
 * ファセット検索
 * 複雑なグルーピング機能

### Test Driven Search

 * livedoor グルメ DataSet
 * elasticsearch-ruby
 * Tokenizer
 * TokenFilter
 * _id Elasticsearch 内でユニークな ID
 * suggest
 * Chrome の Sensu

### Sensu

 * marvel

### question

 * フィールドの追加はインデックスを作り直す必要はない
 * type の変更は必要

## 株式会社リブセンス 吉田 健太郎さん @yoshi_ken タイトル：MySQLユーザ視点での、小さく始めるElasticsearch

### 実データと ES の連携

 * MySQL のデータを ES に
 * river-jdbc
 * MySQL bin-log API
 * Yamabiko

### yamabiko

 * MySQL => ES 非同期で反映

### 緯度経度検索

 * MySQL の Geo 検索はイケてない

### Elasticsearch のハマりどころ

 * Query DSL が複雑
 * mapping 定義を更新するためにには index/type を消す必要がある
 * noSQL 的な概念の理解
 * noSQL ゆえに JOIN 出来ない
 * 各 type に同じデータが入る

### AWS RDS との連携

## 株式会社リクルートテクノロジーズ 相野谷 直樹さん @naokiainoya タイトル：nodeJS+DynamoDB＋Elasticsearchで全社基盤を作った話

### 背景

 * push 通知システム

### システム構成

 * DynamoDB（KVS 検索が苦手）
 * Elasticsearch を検索エンジンとして...

### ES の使い方

 * クラスタ構成
 * Autoscaling
 * MultiAZ（2AZ）
 * Scan を使用（深いページングになっても高速）

### リストア

 * マスターデータは DynamoDB
 * ES はいつ落ちても復旧出来るようにしておく

### まとめ

 * AutoScaling でどんな風にあがるのか？
 