# 2016 年からはじめる Amazon EC2 Container Service と EC2 Container Registry 入門 + おまけ

## アジェンダ

- 注意
- 自己紹介
- 俺の Amazon ECS & ECR 入門
- 俺の Amazon ECS & ECR 一問一答
- おまけ
- 終わり

## 注意

- 自分が Amazon EC2 Container Service と Amazon EC2 Container Registry に入門したメモです
- 運用事例等の役立つ情報は無いかもしれません
- 本資料に記載された内容の全ては作成時点の内容となります
- 本資料内で Amazon EC2 Container Service を Amazon ECS と Amazon EC2 Registry を Amazon ECR と省略して記載している部分があります

## 自己紹介

- 川原洋平(@inokara)

## 俺の Amazon ECS & ECR 入門

### Amazon ECS とは

- 概要
 - マネージドなクラスタ管理
 - スケジューリング
 - AWS の各種サービスとシームレスな連携
- 用語を抑えておきましょう
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
  - 
 - Task
 - Container
 - Run Task
 - Service

### Amazon ECR とは

- 概要
 - Docker Hub とほぼ同じ使い方が出来る
 - 但し、Automated Build とかは無い
- 用語
- アーキテクチャ

***

## 俺の Amazon ECS & ECR 一問一答

- コンテナのログを CloudWatch Logs に飛ばしたい
 - Docker の Logging Driver で CloudWatch がサポートされた
 - Docker 1.9 からサポートされた(Fluentd は Docker 1.8 から)
 - Amazon ECS Optimized Amazon Linux でも Docker 1.9.1 がサポートされたのであまり意識せずに飛ばせるようになったはず
 - Container Insntance に以下の IAM Policy をアタッチしておく
 - デフォルトではコンテナ ID が Log Stream 名となる
 - 同一の Log Stream に複数のコンテナからログを送ることは非推奨(パフォーマンスに影響が出るとのこと)
- コンテナのモニタリング
 - CloudWatch
  - コンテナそのもののモニタリングはサポートされていない
  - Container Instance
　- Cluster 及び Service 全体の CPU とメモリの使用率を確認することが出来る
 - その他のサードパーティ製ツール
  - Datadog
  - Mackerel
- プライベートリポジトリを構築したい
 - Amazon ECR 一択では無いが、個人的には Amazon ECR オススメ
- ECS 以外から ECR を利用したい
 - Elastic Beanstalk で利用する
 - 普通の Docker から

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
