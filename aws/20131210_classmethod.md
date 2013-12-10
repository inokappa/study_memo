## URL

 * http://dev.classmethod.jp/news/developersio-meetup-01/

## 場所

 * SAP
 * #cmdevio

***

## 資料

 * [6 リージョン同時 75 万接続のメッセージ配信基盤を CloudFormation と Capistrano で](http://www.slideshare.net/takipone/675cloudformationcapistrano)
 * [CloudFormationとSerfで作る全自動インフラ](http://www.slideshare.net/masaomoc1015/cloudformationserf-29075637)
 * [Infrastructure as Code から Full Reproducible Infrastructure へ](http://www.slideshare.net/daisuke_m/20131210-cmre-growthawsmiyamoto)
 * [6 リージョン同時 75 万接続のメッセージ配信基盤を 3 日で考えた話](http://www.slideshare.net/satoshi1977/developersio-meetup-01)

***

## Amazon Redshift & EMR & CloudTrail を TableauDesktop

 * redbull とポカリ
 * @shinyaa31
 * Cloudtrail
 * EMR -> Redshift 
 * 全てのサービスで使えるわけではない
 * RecordBody
 * EMR でパイプ区切りに整形したい
 * Hadoop Streaming
 * Mapper プログラム、Reducer プログラムを S3 上に置く
 * ターミネートを自動にしておく
 * Redshift
  * 列志向
  * クエリ分散
  * 1.25$
 * TimeStamp がソートキー
 * TableauDesktop

 * 何を分析したいか？
 * 武器が必要
 * ビジネス知識
 * 自動化の知識

***
## re:Invent の傾向と対策

 * 武川  努さん

### どうやったらいけるの？
 
 * classmethod に入社するｗ
 * 最新技術の発表
 * 既存技術の深堀り

### 行く前の不安とその結果

 * 英語わからない
  * ツアー
  * 日本語同時通訳
  * 専用トラック
  * Facebook グループ
  * AWS の中のヒトと仲良くなれる
 * ラスベガスって
 * 時差ボケ
 * 仕事

### re:Invent 2013 の反省点と対策

 * 170 セッション
 * 当日新サービスの発表がある
 * iPhone アプリを活用しよう

***

## EMR with the MapRは何がうれしいの（仮）

 * @n3104 さん

### EMR

 * Elastic MapReduce
 * Hadoop のディストリビューション
 * 保守不要
 * S3 に入出力ファイルを置く
  * HDFS の障害を考慮しなくて済む
  * 容量制限を考慮しなくて済む

### Hadoop

 * HDFS（分散ファイルシステム）
 * MapReduce

### MapR とは？

 * Hadoop のディストリビューションの一つ
 * C++ で書きなおしている
 * 性能を向上させている、その他、本家に無い機能が追加されていいる

### EMR with the MapR

 * MapR を EMR 上で利用出来るオプション

### 検証

 * Big Data Benchmark
 * benchmark gitbub repo
 * Amazon Hadoop vs MapR M3
 * MapR がかなり早い
 * コストを下げることが出来る！

***

## 運用担当者からみた、AWSへシステム移行する際に気にしてほしい5つのこと

 * 運用からみた AWS 以降のハマりどころ
 * 植木さん

### とある web システム

 * PHP
 * MySQL

 * 協力会社
  * FTP
  * DB
  * mail サーバー
  * システムのデータの流れを全てを洗い出す
  * ログ・ファイル
  * おペトレ

 * リリース
  * AMI
  * Chef / puppet
  * 更新分は S3 から git から持ってくる
  * EC2 のローカルにデータを置かない

 * 監視
  * ELB を使いましょう
  * ミドルウェアの監視（Zabbix や Cacti で取る）
  * CloudWatch のメトリクスは一年保存
  * HOOT24

 * 保守
  * （Zabbix や Cacti で取る）
  * RDS のログも保存
  * 保守用毛色
  * AWS サポート（利用料金の 10%）

 * 数年後の担当者
  * IT 会社の勤続年数 3 〜 5 年
  * プログラマ定年
  * 貴方が構築したシステムは貴方がいなくなっても動き続ける
  * チームをまとめてフローを整備し、組織として運用を行う
  * 後世に知見を残す

## CloudWatchで出来ること

 * 横田慎介さん
 * SQS / SNS

### CloudWatch とは

 * 各種リソースの監視
 * 監視データの倉庫

### 用語

 * メトリクス
  * データポイントのいれもの
 * ネームスペース
 * ディメンジョン
 * メモリの使用率
 * カスタムメトリクスには費用がかかる

### データの登録

 * メトリック
  * ネームスペース
  * メトリック名
  * ディメンジョン

### 取得

### AWS メトリクス

 * CPU Utilization
  * top コマンドとの差異がある
  * CloudWatch の値を信用したほうが良い
 * Disk I/O の場合には
  * EBS
 * ELB
  * AZ の healthy と unhealthy
  * RequestCount
  * SurgeQueueLength

### まとめ

 * 期待しているものと違う値をとっている
 * 複数の監視システムと連携することが適している

## 6 リージョン 75 万接続の…CloudFormation と Capistrano で 3 日で構築した話

 * たきぽんさん
 * Storage Gateway仮想テープライブラリによるバックアップ戦略 は JAWS-UG で

### システム概要

 * テレビ番組連動
 * お題をプッシュ
 * インフラのみ
 * 6 リージョン 180 台
 * AutoScaling は使えない
 * 時間との闘い

### ツールという翼

 * CloudFormation
 * Capistrano

### CloudFormation

 * JSON テンプレート
 * AWS のコンポーネントを自動作成
 * Parameters / Outputs
 * 全コンポーネントを単一テンプレートで記述しない
  * 作成でコケると全部ロールバックしてしまいなにも残らない
  * AutoScaling は上限に達してもエラーを出さない
  * AWS 全体のキャパシティ

 * ELB
 * EIP -> セキュリティグループ
 * 複数テンプレートを組み合わせると Management Console での実行は破綻する
  * ID の受け渡し

### Capistrano

 * Capistrano
 * AWS SDK for Ruby との組み合わせが超強力
 * ユーザー権限のバッティング
 * 冪等性がない
  * むりくり : をつけた

### まとめ

 * CloudFormation は自動構築、複数リージョン対応
 * Capistrano と SDK の組み合わせが最高

## AWS SDK for .NETを利用したExcelによるAWS環境定義書自動生成

 * @Cronoloves さん

### ドキュメント作成

 * 大企業
 * 金融系

### VBA VS Office アドイン

 * AWS SDK が使える（Office アドイン）

## CloudFormationとSerfで作る全自動インフラ

 * @Canelmo さん
 * aws.rb

### Cloudformation

 * AWS リソースを JSON 定義
 * 外部パラメータ、結果 Output
 * AWS がサンプルを用意
 * CloudFormaition 入門
 * ボタン一個で環境が構築出来る
 * Stack 削除で全部消える
 * AWS::CloudFormation::Init でなんでも出来る
 * 長い JSON
 * デバッグし辛い
 * 適度な粒度で分割

### Serf

 * オーケストレーションツール
 * イベントに応じてカスタムスクリプトを実行することが出来る
 * 軽量
 * クラスタリング
  * 特定の親を持たない
 * イベントハンドリング
  * member-join
  * member-failed
  * member-leave
  * user
 * どう使う？
  * APP サーバーのアプリケーションデプロイ
  * 監視サーバーへの自動追加

### CDP

 * HA NAT

### まとめ

## Infrastructure as CodeからFull Reproducible Infrastructureへ

 * @daisuke_m
 * Full Reproducible Infrastructure（再現可能）
 * AWS ひとり DevOps が
 * Jiemamy
 * 依存リソースを極限まで排除
  * private な API
  * AWS Accuont ID 付きの ARN 等
 * DB ホスト名はどうやって？
 * CloudFormaiton
 * コマンド一発で環境が再現出来ることが大切
 * 抽象化しすぎると大変
 * ElasticBeansTalk

## 6 リージョン 75 万接続のメッセージ配信基盤

 * 横田聡さん
 * Classmethod 代表取締役
 * お誕生日おめでとうございます

### 月曜日〜木曜日

 * 60  万人
 * 来週までに
 * 上限緩和申請（20 台が上限）
 * 暖機申請
 * ゴールデン AMI 作成
 * HTML * Scoket.IO
 * Node + Redis
 * 既設によってインスタンス数確保問題

 * 突発的なアクセスへのリミット制限
 * ネットワークキャパシティ
 * 障害発生時のフェールオーバー待ち
 * プッシュ配信安定する？
 * ログは？
 * キャリアが DNS リゾルバキャッシュすることがある
 * Socket.IO のシュミレーションに JMater だと同じ条件で出来ないよね？

### ログ

 * fluentd
 * TreasureData
 * 8 億レコード

### 学び

 * マルチリージョン推し
 * AZ で可用性を確保しよう→マルチリージョンで可用性を高める（クロスリージョン）
