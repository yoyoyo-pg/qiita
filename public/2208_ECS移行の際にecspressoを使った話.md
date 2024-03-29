---
title: EC2→ECS移行の際に、本番環境へのデプロイにecspressoを用いた話
tags:
  - AWS
  - ECS
  - ecspresso
private: false
updated_at: '2024-03-16T08:45:40+09:00'
id: 4fd9d198e9ebd035c685
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

本日のテーマはWebアプリケーションのEC2 → ECS Fargate移行の際に、
ECSデプロイツールであるecspressoを利用した話です。

### 移行対象のアプリケーションについて

- JavaのWebアプリケーション
- テスト・デプロイ作業は手動な部分が多い

元々のアプリケーションは複数機能が同一EC2上に同居しており、今回その一部アプリケーションをリプレースすると共に、コンテナ化する形を取りました。

## プロジェクトの全体像

- コンテナ移行（EC2→ECS Fargate）
- CICDパイプライン用意（開発→テスト環境）
- **リリースシェル用意（テスト→本番環境）**  👈今回のテーマ

開発環境と本番環境のAWSアカウントが分かれており、
既存Webサーバーと同様に夜間リリースの運用にしたいという事情がありました。

他にも、テスト環境と本番環境の設定ファイルを書き換えたいといった事情もあり、今回はCICDパイプラインと別に

- **テスト環境でテスト済のコンテナイメージを基に、本番環境にデプロイする為のリリースシェル**

を用意しました。

## リリースシェルについて

CICDパイプラインによって、テスト済のDockerイメージがテスト環境側のECRにある状態とはなりますが、ここで厄介なのがDockerイメージを「どのように本番ECRに持っていくか？」でした。

[クロスアカウントレプリケーション](https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/replication.html)を使えば、テストECRのイメージを本番ECRにレプリケーションする事は可能ですが、以下の問題がありました。

- 本番アカウントのECRに、リリースされていないDockerイメージも混在してしまう
  - 結果、ECRリポジトリ内の見通しが悪くなる
- テスト環境と本番環境用のWebアプリケーションの設定ファイルの書き換えが必要
  - 本番ECRにレプリケーションされたイメージを更に加工する必要がある

問題を解決するうえで、リリースシェルは以下の方式を取ることにしました。

## リリースシェル概要

1. テスト済のDockerイメージをテストECRからプル
2. コンテナの再ビルド（設定ファイルを書き換え）
3. イメージを本番ECRにプッシュ
4. 本番ECSにデプロイ

上記の方法を取ることで、本番ECRに「リリースするDockerイメージ」のみ保持するようにしました。

また、本番ECRにプッシュしたイメージのタグは「リリース日」+「ビルド前のイメージタグ」とすることで、イメージの追跡が行える状態にしました。

リリースシェルの一部は以下の記事に記載してあります。

https://qiita.com/yoyoyo_pg/items/71827342f2870beae098

## ecspresso採用の経緯

ここまで説明したリリースシェルですが、他にも以下の検討点がありました。

- ECSの構成のコード管理（タスク定義、サービス定義）をどうするか？
- 複数のテスト環境への簡易デプロイ手段の確立

特にECSのコード化については、デプロイの度にタスク定義・サービス定義が変わり、必要タスク数の増減もある為CloudFormationで管理しにくいという事情もあり、コード化+デプロイも行えるという点が便利だと感じた為ecspressoを採用しました。

## ecspressoについて

- カヤックさんが公開しているオープンソースのECSデプロイツール
- ECSサービス、タスクに関わる最小限のリソースをコード管理できる
- 既存の ECSサービス、タスクの情報を元に構成ファイルを生成する機能
- ECSサービスのタスクを入れ替える機能

※作者の方がアドベントカレンダーを公開しており、こちらを参考にしました。

https://adventar.org/calendars/5916

元々VPC周りは既に構築されている物を利用するという事情もあり、
ecspressの**ECSサービス、タスクに関わる最小限のリソースをコード管理できる**点が良かったのと、
Javaのメモリリーク対策を考えた際に**ECSサービスのタスクを入れ替える（refresh）機能**が非常に役に立ちました。

### ecspressoのコマンド紹介

ecspressoによる実際のデプロイを説明する上で、リリースシェルに組み込んだコマンド等を先に紹介します。

```sh
# 既存のECR構成を取り込み（オプションは省略）
ecspresso init
# 以下3ファイルが生成
config.yaml            リージョン、クラスター、サービス名を定義
ecs-service-def.json   ECSサービス定義
ecs-task-def.json      ECSタスク定義
# 上記設定ファイルを基にECSへデプロイ
ecspresso deploy --config config.yaml
```

## リリースシェルの詳細

リリースシェルの具体的な内容について、実際のリリース手順と共にご紹介します。

### リリース手順1

- リリース前にcronに「リリース日時」「ECRイメージURI」をセット　※リリース実行用のEC2上

```sh
* * * * * /bin/sh release.sh イメージURI
```

### リリース手順2

**ここからは`release.sh`内で実行される処理です。**

```sh
# リリース対象のイメージURI（再ビルド後）を定義
RELEASE_IMAGE_URI=${ACCOUNT_ID_RELEASE_IMAGE}.dkr.ecr.
${REGION_RELEASE_IMAGE}.amazonaws.com/${REPO_NAME_RELEASE_IMAGE}
:IMAGE_TAG_FOR_RELEASE_IMAGE
```

- cronで引数として渡した「ECRイメージURI」
- シェル内で定義している本番アカウントの「アカウントID」「リージョン名」

を基に、動的にリリース対象のイメージURIを定義しています。

※ここから、設定ファイル書き換えの為の再ビルド→本番ECRに対してのプッシュをシェル内で実施していますが、こちらについては[別の記事](https://qiita.com/yoyoyo_pg/items/71827342f2870beae098)で説明している為割愛します。

### リリース手順3

**こちらも`release.sh`内で実行される処理です。**

```sh
# ecspresso initにより事前に準備した設定ファイルのディレクトリに移動
cd /home/ec2-user/deploy/config

# タスク定義、サービスをデプロイ
ecspresso deploy --config=config.yaml
```

事前に`ecspresso init`で定義した設定ファイル（`config.yaml`、`ecs-service-def.json`、`ecs-task-def.json`）を参照し、`ecspresso deploy`を実行します。

「どのようにリリース対象のECRイメージを指定するか？」ですが、
ここではecspressoの**テンプレート記法**を利用しています。

```ecs-task-def.json
  "containerDefinitions": [
    {
      "cpu": 0,
      "environment": [],
      "essential": true,
      "image": "{{ must_env `RELEASE_IMAGE_URI` }}",
```

`ecs-task-def.json`内のimageURIを定義する箇所でテンプレート記法を利用しています。
※`RELEASE_IMAGE_URI`は、リリース手順2で定義した「再ビルド後のイメージURI」です。

これにて、事前定義した設定ファイルを基にしたECSへのデプロイは完了です！

## その他にも

定期実行しているメモリリーク対策のシェルにも、ecspressoを利用しています。
`ecspresso refresh`により、タスク定義を新しく登録せずにタスクの再デプロイが可能です。

```refresh.sh
ecspresso init --config=config.yaml --region=リージョン --cluster=クラスター名 --service=サービス名
ecspresso refresh --config config.yaml
```

## 参考文献

- [【レポート】第2回 AWS Fargate かんたんデプロイ選手権 #AWSDevDay](<https://dev.classmethod.jp/articles/awsdevday2020-deploy-fargate-easily/>)
- [あなたの組織に最適なECSデプロイ手法の考察](<https://dev.classmethod.jp/articles/ecs-deploy-all/>)

ecspresso advent calendar

- [ecspresso advent calendar 2020 day 3 - 既存ECSサービスの取り込み](https://zenn.dev/fujiwara/articles/f2314651691adcae5215)
- [ecspresso advent calendar 2020 day 4 - deploy](https://zenn.dev/fujiwara/articles/b5cb8bc64c83a6c8ca12)
- [ecspresso advent calendar 2020 day 5 - テンプレート記法](https://zenn.dev/fujiwara/articles/ecspresso-20201205)
- [ecspresso advent calendar 2020 day 8 - refresh](https://zenn.dev/fujiwara/articles/ecspresso-20201208)
