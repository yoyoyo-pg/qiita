---
title: FargateのエフェメラルストレージをDatadogで監視する
tags:
  - Datadog
  - ECS
private: false
updated_at: '2024-03-30T19:59:45+09:00'
id: ae7c577c00ceca47c2a3
organization_url_name: null
slide: false
ignorePublish: false
---
## Fargate タスクエフェメラルストレージについて

- ECS FargateのLinuxプラットフォーム1.4.0以降のECSタスクに対して、デフォルトで20GiBのエフェメラルストレージが割り当てられます
- `ephemeralStorage`パラメータを設定する事で、最大200GiBまで増やす事が可能です

> プラットフォームバージョン 1.4.0 以降を使用している Fargate でホストされている Amazon ECS タスクは、デフォルトで、最低 20 GiB のエフェメラルストレージを受け取ります。エフェメラルストレージの総量は、最大 200 GiB まで増やすことができます。これを行うには、タスク定義内で ephemeralStorage パラメータを設定します。

参考：[Fargate タスクエフェメラルストレージ](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/fargate-task-storage.html#fargate-task-storage-linux-pv)

## エフェメラルストレージのモニタリング

また、ECS FargateのLinuxプラットフォーム1.4.0以降から、ストレージ利用のモニタリングが可能となっています

参考：[AWS Fargate、ストレージ利用のモニタリングのサポートを追加](https://aws.amazon.com/jp/about-aws/whats-new/2022/11/aws-fargate-monitoring-storage-utilization/)

## Datadogでのモニタリング

Fargateのエフェメラルストレージについては

- 予約値
  - `ecs.fargate.ephemeral_storage.reserved`
- 使用量
  - `ecs.fargate.ephemeral_storage.utilized`

をDatadogのダッシュボード内で指定することで、モニタリングが可能です

[Amazon ECS on AWS Fargate - メトリクス](https://docs.datadoghq.com/ja/integrations/ecs_fargate/?tab=webui#%E3%83%A1%E3%83%88%E3%83%AA%E3%82%AF%E3%82%B9)

なお、Datadog以外にもContainer InsightsやFargateコンテナ内から確認が可能です

[AWS Fargateがストレージ容量を監視できるようになりました](https://dev.classmethod.jp/articles/aws-fargate-monitoring-storage-utilization/)
