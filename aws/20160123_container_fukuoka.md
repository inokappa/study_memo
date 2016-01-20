# Amazon EC2 Container Service と EC2 Container Registry つまみぐい + おまけ

## アジェンダ

- 注意
- 自己紹介
- 俺の Amazon ECS & ECR 入門
- 俺の Amazon ECS & ECR 一問一答
- おまけ
- 終わり

## 注意

- 私個人が独自の観点で Amazon EC2 Container Service と Amazon EC2 Container Registry について調べたメモです
- 運用事例等の役立つ情報は無いかもしれません
- 本資料に記載された内容の全ては作成時点の内容となります
- 本資料内で Amazon EC2 Container Service を Amazon ECS と Amazon EC2 Registry を Amazon ECR と省略して記載している部分があります

## 自己紹介

- 川原洋平(@inokara)

## 俺の Amazon ECS & ECR 入門

### Amazon ECS とは

####  概要

- マネージドなクラスタ管理
- スケジューリング
- AWS の各種サービスとシームレスな連携

#### 構成要素と用語

- Cluster
 - Container Instance の集合
 - Cluster 内の Instance リソースを管理
- Container Instance
 - コンテナが起動するインスタンス
 - VPC 内で任意のインスタンスを利用することが出来る
 - Amazon ECS optimized AMI が提供されている
 - ECS Ageent を動かすことが出来れば Ubuntu 等のインスタンスでも OK
- ECS Agent
 - Amazon ECS Endpoint との通信を行うエージェント
 - こやつも Docker コンテナで Go で実装されている
 - Docker Hub で提供されている
- Task Definition
 - コンテナの設定を JSON ファイルで定義したもの
 - コンテナリソースを(CPU/Memory/Port)定義
 - ボリュームも定義することが可能
 - docker run する時のオプションを予め定義するイメージ
- Task
 - Task Definition が展開されたもの
 - リソースに余裕がある Container Instance で実行される
- Container
 - Task の実体
 - Container Instance 上で実行される
- Run Task
 - Task Definition から指定した数の Task を実行する
 - 単体のコンテナを docker run するイメージ
- Service
 - Web サービス等の長期稼働するアプリケーションで利用する
 - スケジューラが Task の数を維持してくれる
 - Task Definition をデプロイしながら Task を切替することが出来る
 - ELB との連携が可能

#### なんぼ？

 - ECS 自体の利用は無料
 - Container Instance として利用する EC2 の料金(Instance と EBS の料金)
 - ELB を利用する場合には ELB の料金

#### 操作

 - マネジメントコンソール
 - AWS CLI
 - ECS CLI
 - SDK

### Amazon ECR とは

#### 概要

- Docker Hub とほぼ同じ使い方が出来る
- 但し、Automated Build とかは無い
- 現在は us-east-1 のみで提供されている

#### 構成要素と用語

- Docker のサポート
 - Docker Registry HTTP API V2
 - ECS に限らず様々な Docker 環境から利用出来る
- 高可用性と耐久性
 - コンテナイメージは S3 に保存される
- アクセスコントロール
 - IAM を利用してコンテナイメージへのアクセスを制御出来る
- 暗号化
 - S3 のサーバー側暗号化を使用して保管中のイメージが自動的に暗号化される
 - コンテナイメージの送受信は HTTPS 経由で行われる

#### なんぼ？

- ストレージ使用料 0.10 USD/GB/Month
- 受信：Amazon EC2 Container Registry への転送(docker push)は $0.000 /GB
- 送信：Amazon EC2 Container Registry からの転送(docker pull)は $0.000 /GB 〜
- 同一リージョン間は無料
- リージョンを跨ぐ場合には受送信ともに費用が発生する

#### 操作

- マネジメントコンソール
- AWS CLI
- SDK

***

## Amazon ECS & ECR つまみ食い

### コンテナのログを CloudWatch Logs に飛ばしたい

- Docker の Logging Driver で CloudWatch がサポートされた
- Docker 1.9 からサポートされた(Fluentd は Docker 1.8 から)
- Amazon ECS Optimized Amazon Linux でも Docker 1.9.1 がサポートされたのであまり意識せずに飛ばせるようになったはず
- Container Insntance に以下の IAM Policy をアタッチしておく
- デフォルトではコンテナ ID が Log Stream 名となる
- 同一の Log Stream に複数のコンテナからログを送ることは非推奨(パフォーマンスに影響が出るとのこと)
- Task Difinition で定義することは現時点では出来ないので注意

### コンテナのモニタリング

- CloudWatch
 - コンテナそのもののモニタリングはサポートされていない
 - Container Instance
 - Cluster 及び Service 全体の CPU とメモリの使用率を確認することが出来る
- その他のサードパーティ製ツール
 - Datadog
 - Mackerel

### プライベートリポジトリを構築したい

- Amazon ECR 一択では無いが、個人的には Amazon ECR オススメ(但し、us-east-1 のみ展開されているので注意)
- ECS クラスタ内にプリベートリポジトリを構築することが出来る

### ECS 以外から ECR を利用したい

####  Elastic Beanstalk で利用する
#### 普通の Docker から利用する

### ELB と Dynamic Port Mapping

- ELB と Dynamic Port Mapping が連携出来ると嬉しいけど...
- ELB 側の制約により実現出来ない

## 一旦、終わり

### Amazon ECS を触った感想

### 参考

- http://www.slideshare.net/AmazonWebServicesJapan/aws-blackbelt-2015-ecs

***

## おまけ 〜最近 LXD を少し触ってみました〜

### LXD とは

- LXD とは
- LXD API に注目

### せっかくなので何か作ろう

- oreno_lxdapi(LXD API のクライアントライブラリ for Ruby)
 - ざくっと作った
- oreno_lxdapi を利用した test-kitchen ドライバ(kitchen-lxd_api)
 - test-kitchen とは Chef 向けに開発されたテスティングフレームワーク
 - とは言え、Ansible や Puppet を Provisioner として利用可能
 - テストを実行する環境についても Vagrant / Docker や 各種 IaaS 環境に対応したドライバが提供されている
- LXC 対応を謳っているドライバもあるけど動かなかった
- LXD に対応したドライバも実は存在する(kitchen-lxd_cli)

### demo

- 時間があったら

### 以上

- おまけでした
- よければ使ってみて下さい

***

## 質問

...はお手柔らかに...
