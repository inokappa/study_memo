# Amazon ECS 試行錯誤

## おわび

- 今まで Blog に認めた内容の総集編となります

## 目次

- 自己紹介
- Amazon ECS について
 - 概念
 - 用語
- 試行錯誤
 - プライベートリポジトリ
 - 監視
 - サービスディスカバリ
- まとめ 

***

## 自己紹介

- 地元枠
- Amazon ECS 及び Docker をがっつり使っているわけではない（趣味レベル）

***

## Amazon ECS について

### 参考

- http://www.slideshare.net/AmazonWebServicesJapan/aws-blackbelt-2015-ecs

### 概念

### 用語

### ここまでのまとめ

***

## Amazon ECS 試行錯誤

### プライベートリポジトリを S3 に作った話

- http://inokara.hateblo.jp/entry/2015/07/25/103130
- Amazon ECR ってのももうすぐ出てくる

### Datadog を使ったモニタリング

- http://inokara.hateblo.jp/entry/2015/08/02/015035
- http://inokara.hateblo.jp/entry/2015/09/01/112512
- http://inokara.hateblo.jp/entry/2015/08/19/224532

### Consul を使ったサービスディスカバリ

- http://inokara.hateblo.jp/entry/2015/06/25/005317

### ECS 関係無いけど...バッチコマンドの Docker 化

- バッチ的な処理を Docker 化する
- Docker 化することのメリット
 - スクリプトやスクリプトが動く環境を含めてコード化出来る
 - Docker さえ入っていれば...環境を選ばない
 - 手元で動作確認したものをそのままデプロイすることが出来る（もちろんテストはする）
- Docker 化することのデメリット
 - `--env` とか使うと動作確認がちょっと面倒
- 作るツールは出来るだけ Docker 化していくつもり
