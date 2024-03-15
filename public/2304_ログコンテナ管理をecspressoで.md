---
title: ログコンテナのバージョン管理の為に、ecspresso v2のSSMパラメータストア参照を利用してみた
tags:
  - ECS
  - SSM
  - ecspresso
private: false
updated_at: '2024-03-16T08:45:40+09:00'
id: 1b562124e6b5d62f57d4
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

コンテナデプロイツールである`ecspresso`ですが、ecspresso v2が昨年末にリリースされました！

ecspresso handbookでも紹介されている**ecspresso v1とv2の変更点**の中でも、今回はSSMパラメータストア参照を利用しました。

<https://zenn.dev/fujiwara/books/ecspresso-handbook-v2/viewer/v1-v2>

## 何のために利用したか？

ECSのログコンテナ用fluent-bitのイメージですが、

- 検証環境で`stable`タグ指定で運用し、ログ転送に問題が無いかをテスト
  - 具体的には`public.ecr.aws/aws-observability/aws-for-fluent-bit:stable`の形で指定
- ログ転送に問題が無い場合、動作環境用のSSMパラメータストアの値を`stable`タグのイメージURIに変更
- ecspressoで動作環境にデプロイする際に、SSMパラメータストアのfluent-bitの新規イメージURIを取得し、デプロイ

といった形式をとる事で、**お手軽に実現可能なfluent-bitのバージョンアップの仕組み**を構築しました。
ecspressoを元々デプロイ時に利用している事もあり、今回のアップデートは非常に助かりました！

## パラメータの作成

- 以下の形でパラメータを作成します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/203a0b48-c171-1b7a-f59e-8c33dc4b123f.png)

## ecspressoの設定ファイルにSSMを利用する為の設定を追記

- `ecspresso.yml`にssmについての設定を追記します。

```ecspresso.yml
region: ap-northeast-1
cluster: cluster-name
service: service-name
service_definition: ecs-service-def.json
task_definition: ecs-task-def.json
timeout: "10m0s"
plugins:
  - name: ssm
```

- `ecs-taskdef.json`のlogrouterコンテナのimage指定をSSMパラメータストア参照に変更

```ecs-taskdef.json
"image": "{​​{​​ ssm `/ecs/log-agent/fluent-bit-image-uri` }​​}​​"
```

## デプロイ実行用のIAMロールの記載

- 以下ポリシーをecspressoでのデプロイ実行用IAMロールへと付与します。

```json:Policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:Get*"
            ],
            "Resource": "arn:aws:ssm:XXXXXXXXXX:XXXXXXXXXX:parameter/ecs/log-agent/fluent-bit-image-uri"
        }
    ]
}
```

## おわりに

- SSMパラメータストア参照＆ecspressoのおかげで、便利にログコンテナのバージョン管理が出来そうです。
