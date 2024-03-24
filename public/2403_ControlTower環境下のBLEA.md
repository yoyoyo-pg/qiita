---
title: AWS Control Tower × BLEAの概要について調べてみる
tags:
  - AWS
  - CDK
  - ControlTower
  - BLEA
private: false
updated_at: '2024-03-24T17:26:44+09:00'
id: f9f1a392e62eb8ed2903
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

Control Tower と BLEA を用いたゲストアカウントのセットアップについて、BLEA 公式のGitHubの`README`から概要をまとめました

## 前提

- AWS CDKや Control Tower の基礎知識

## BLEAとは

- Baseline-Environment-on-AWS の略です
- スタンドアロン又は Control Tower ベースのマルチアカウントAWS環境において、安全なベースラインを確立するためのCDKテンプレートです
- [aws-samples / baseline-environment-on-aws](https://github.com/aws-samples/baseline-environment-on-aws)上にコードが公開されています

## Governance baselines

Control Tower のゲストアカウントに対して、以下3つの導入方法が提供されています

1. 手元からの直接展開(`blea-gov-base-ct.ts`)(デフォルト)
2. cdkPipelinesを使用した展開(`blea-gov-base-ct-via-cdk-pipelines.ts`)
3. ControlTowerのAFCを使用した展開(`blea-giv-base-ct-via-cdk-pipelines.ts`)

各テンプレートはリポジトリ内の[usecases/blea-gov-base-ct](https://github.com/aws-samples/baseline-environment-on-aws/tree/main/usecases/blea-gov-base-ct)配下にあり、CDKの別スタックとして用意があります

2,3の方法はアカウント払い出しの自動化の際は有用かと思いますが、1の方法が最初は試しやすそうです

参考：<https://github.com/aws-samples/baseline-environment-on-aws?tab=readme-ov-file#governance-baselines>

:::note info
cdkPipelinesは、AWS CDK のスタックを複数環境に展開する為に利用可能なパイプラインです
詳しくはユーザーガイドの[CDK Pipelinesを使用した継続的な統合と配信(CI/CDK)](https://docs.aws.amazon.com/ja_jp/cdk/v2/guide/cdk_pipeline.html)を参照ください
:::

:::note info
AFC（Account Factory Customization）は Control Tower 上から新規及び既存の AWS アカウントをカスタマイズ可能です
詳しくはユーザーガイドの[Account Factory Customization (AFC) を使用したアカウントのカスタマイズ](https://docs.aws.amazon.com/ja_jp/controltower/latest/userguide/af-customization-page.html)を参照ください
:::

## Deployment flow

- ランタイムや導入手順が紹介されていますが、CDK を使える状態（ex. Node.jsのインストールやawscliの認証設定）が最低限必要です
- また`git secrets --scan`がコミット前に実行されるよう設定されているため、コミットをする場合はこちらの[セットアップ](https://github.com/aws-samples/baseline-environment-on-aws/blob/main/doc/HowTo.md#git-pre-commit-hook-setup)が必要です
- プロジェクトルートの[REAME.md](https://github.com/aws-samples/baseline-environment-on-aws/blob/main/README.md)にはスタンドアロンの場合の導入方法が記載されており、ControlTowerへの導入手順は[doc/DeployToControlTower.md](https://github.com/aws-samples/baseline-environment-on-aws/blob/main/doc/DeployToControlTower.md)を参照してください

参考：<https://github.com/aws-samples/baseline-environment-on-aws?tab=readme-ov-file#deployment-flow>

:::note info
git-secretsを利用する事で、Gitリポジトリにシークレットが追加されるのを防ぐ設定が可能です
詳しくは[awslabs / git-secrets](https://github.com/awslabs/git-secrets)を参照ください
:::

## DeployToControlTower

- ありがたいことに[英語版](https://github.com/aws-samples/baseline-environment-on-aws/blob/main/doc/DeployToControlTower.md)と[日本語版](https://github.com/aws-samples/baseline-environment-on-aws/blob/main/doc/DeployToControlTower_ja.md)のREADMEが存在します
- また、[HowTo_ja.md](https://github.com/aws-samples/baseline-environment-on-aws/blob/main/doc/HowTo_ja.md)にはVSCodeのセットアップの他、CloudShellによるデプロイについての解説もあります

## Control Tower 配下への導入手順

- 手順1： Control Tower の追加のサービス設定手順
- 手順2：ゲストアカウントの Control Tower での作成手順
- 手順3~6： パッケージインストール、認証情報の設定、ゲストアカウントへのデプロイ、アプリケーションサンプルのデプロイの手順

手順4のゲストアカウントのガバナンスベースのデプロイにより以下設定がされます

> - デフォルトセキュリティグループの閉塞 （逸脱した場合自動修復）
> - AWS Health イベントの通知
> - セキュリティに影響する変更操作の通知（一部）
> - セキュリティイベントを通知する SNS トピック (SecurityAlarmTopic) の作成
> - 上記 SNS トピックを経由した、メールの送信と Slack のセキュリティチャネルへの通知

引用：<https://github.com/aws-samples/baseline-environment-on-aws/blob/main/doc/DeployToControlTower_ja.md#%E5%B0%8E%E5%85%A5%E6%89%8B%E9%A0%86>

## おわりに

- 今回は実際に設定する所までは試しませんでしたが、 Control Towerを用いて払い出したアカウントに対して、BLEA を用いる事でよりセキュアな環境の実現が可能という事が分かりました
- また、CDK の実装例・コミット時の`git-secrets`の設定・プルリクチェックの仕組み等、CDK を用いた開発全般のリポジトリ設定の手本としても、 BLEA のリポジトリは参考になりそうです

## 参考文献

[slideshare - 20211109 bleaの使い方（基本編）](https://www.slideshare.net/AmazonWebServicesJapan/20211109-blea)
