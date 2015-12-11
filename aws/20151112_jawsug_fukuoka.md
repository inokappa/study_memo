# 2015 年のうちにやっておきたい Amazon Elasticsearch Service 入門

## アジェンダ

- 注意
- 自己紹介
- Amazon Elasticsearch Service について
- Demo の準備
- 個人的な事例(Demo を含む)
- 終わり

## 注意

- 自分が Amazon Elasticsearch Serivce に入門したメモです
- 運用事例等の役立つ情報は無いかもしれません
- 本資料に記載された内容の全ては作成時点の内容となります
- 本資料内で Amazon Elasticsearch Serivce を Amazon ES と省略して記載している部分があります

## 自己紹介

- 川原洋平(@inokara)

## Amazon Elasticsearch Service について

### その前に Elasticsearch とは

- https://speakerdeck.com/johtani/elasticsearchfalseshao-jie-tote-zheng-cross-2015
- スキーマフリー
- 分散ドキュメントストア
- REST & API
- インストールが簡単、導入が簡単
- Java で実装されている

### ボクと Elasticsearch の過去

- 導入し易いが故の...
 - JVM ...
 - プラグイン ...
 - アクセスコントロール ...
- 個人的に大変な過ちを犯したことがありますが、また今度...

### Amazon Elsticsearch Service について(概要)

- 簡単デプロイ
- 管理(可用性の確保、パッチ管理、障害検出、ノードの交換、バックアップ、モニタリング等)は AWS にある程度お任せ
- CloudWatch と連携した高いスケーラビリティ
- Kibana 入り(はじめから Kibana 入ってます)
- IAM ポリシーによって安全にクラスタにアクセス出来る

### Amazon Elsticsearch Service について(導入 1)

- Manegement Console からであれば数ステップ
- 後ほど Demo します
- クラスタ = ドメイン
- ドメインを作るところから始める
- 10 分位でアクセス可能な状態になる

### Amazon Elsticsearch Service について(導入 2)

- インスタンスタイプ
 - T2 テスト、クラスタ管理向け
 - R3 メモリ重視
 - I2 ハイパフォーマンス
 - M3 一般的な利用

### Amazon Elsticsearch Service について(導入 3)

- ドメインのノード数
 - Enable dedicated master => クラスタ管理のみを行うノードを作成する(奇数で用意することを推奨)
 - Enable zone awareness(あうぇあねす) => リージョン内の複数の AZ にノード分散配置する

- Advance setting(今回は特に触れない)
 - rest.action.multi.allow_explicit_index => URL Base のアクセスポリシーを指定
 - indices.fielddata.cache.size => fielddata のキャッシュサイズを指定

### Amazon Elsticsearch Service について(導入 4)

- ストレージ
 - T2 シリーズ以外は Instance Store を選択することができる
 - EBS は Magnetic / GP2 / PIOPS から選択することが出来る
 - EBS のサイズは最低 10GB から最大 35GB まで指定することが出来る
 - スナップショットの時間を指定する（今回は特に指定せずデフォルトのまま）

### Amazon Elsticsearch Service について(導入 5)

- アクセスポリシーの設定
 - IAM や IP で制御することができる(後述)

### Amazon Elsticsearch Service について(管理)

- モニタリングは CloudWatch で
- 必要に応じて手動でスケールアウトすることができる
- スナップショット
 - デフォルトでは自動的に取得される
 - スナップショット取得時間は指定可能
 - 保存期間は 14 日間
 - レストアするには AWS サポートにお願いする必要がある...
 - Elasticsearch API を使うことで手動でスナップショットを作成することができる(後述)

### Amazon Elsticsearch Service について(API)

- 以下の API がサポートされている
 - http://docs.aws.amazon.com/ja_jp/elasticsearch-service/latest/developerguide/es-gsg-supported-operations.html
- サポートされていない API もある...例えば、status とか(elasticsearch-head とかで使われている)
 - https://www.elastic.co/guide/en/elasticsearch/reference/2.1/cluster-state.html

### Amazon Elsticsearch Service について(プラグイン)

- 以下のプラグインが導入されている
 - Kinaba 3
 - Kibana 4
 - jetty
 - cloud-aws
 - kuromoji
 - icu
- 自分でプラグインが追加出来ない...例えば、elasticsearch-head とか
 - そもそもインストールできたらマネージドではなくなるので仕方ないのかなと思っている

### Amazon Elsticsearch Service について(アクセスポリシー)

- IAM ユーザー、AWS アカウント
- IP アドレス
- アプリケーションから IAM で制御しているアクセスする際には面倒だったので挫折した
 - IAM でアクセス制御している API にリクエストを投げる場合にはアクセスを許可されたリソースの credential を使った署名を付与する必要がある
 - SDK は上記を程よくラッピングしているので意識する必要は無い

***

## Demo の準備

### Amazon ES を構築してみる

- あとで Demo に使うのでクラスタを構築する
- terraform で立ち上げる

***

## 個人的な事例

### 手動によるスナップショット

- モチベーション
 - 自動でスナップショット取得してくれるのは嬉しい
 - レストアには AWS サポートにお願いする必要がある...
- 構成イメージ
- 手順
 1. S3 Bucket 作る
 2. IAM role 作る
 3. IAM role にポリシーアタッチ
 4. スナップショットレジストリの登録
 5. スナップショット作成
- やってみた感じ
 - Elasticsearch API の操作になるので特に難しいことはなかった
 - http://inokara.hateblo.jp/entry/2015/12/05/181358

### Fluentd + CloudWatch Logs との連携

- モチベーション
 - ふつうに Fluentd → Amazon ES は面白くなさそう
 - CloudWatch Logs からは Lambda を介して Stream 処理が出来るらしい
- 構成イメージ
- 手順
 1. CloudWatch Logs の Log Group の作成
 2. Log Group に Log Stream を作成
 3. Log Stream にログが入ってくる...
 4. Log Group を選択して [Start Streaming to Amazon Elasticsearch Service] を選択
 5. Amazon ES のドメインを選択、Lambda Function 用の IAM role を作成（既存の role を選択するか新規作成する）
 6. Log Format を選択（今回は JSON を選ぶ）
 7. 確認して Start Streaming
 8. Amazon ES にインデックスが作成されていることを確認する
- やってみた感じ
 - CloudWatch Logs のコンソールから数ステップで Amazon ES に放り込むことができた
 - http://inokara.hateblo.jp/entry/2015/12/06/192436

### Amazon S3 + S3 Event notification + SQS との連携

- モチベーション
 - 自分で作ったサイトのアクセスログ取ってないなあ
 - せっかくだから取ってみるか
- 構成イメージ
- 手順
 1. Web Site Hosting している S3 Bucket で Access Log を有効にする
 2. Access Log を保存する S3 bucket で Event Notification を有効、通知先を SQS の ARN を指定する
 3. SQS のキューをポーリングして Access Log に保存されるログ(オブジェクト) のキーを取得
 4. 3. で取得したキーの中身(ログ)を S3 Bucket から取得
 5. 4. で取得したログをパースして Amazon ES に放り込む
- やってみた感じ
 - も少しログのパースをしっかりやりたい
 - 当初は Lambda でやろうとしたけど挫折した
 - http://inokara.hateblo.jp/entry/2015/12/10/224031

### Demo(そらまめ君のデータを放り込んで Kibana 4 で可視化)

- やること
 - 九州地方の PM2.5 のデータが公開されている
 - そのデータのうちから前日分を取得して Amazon ES に放り込んで Kibana4 で見てみる
- 構成イメージ
- 手順
 1. すでに Amazon ES は構築済み
 2. Ruby スクリプトで Amazon ES に放り込む
 3. Kibana で可視化

***

## 終わり

### Amazon ES を触った感想

- 簡単構築、運用管理のコストは下がると思う
- 基本的な操作は Rest API で出来るので普通の Elasticsearch となんら遜色が無い
- 但し、利用が制限されている API があるので注意
- インデックスの Mapping についても自分で面倒を見る必要がある(逆に言うと任意に定義出来る)
- Kibana 同梱は嬉しいが自分でプラグインインストールできなさそうなのは辛いけど我慢しよう
- Lambda との連携が理想的だけどアクセスポリシーと併用する際には注意が必要(SDK が対応してくれると嬉しい)

質問...はお手柔らかに...
