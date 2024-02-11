---
title: '[CDK pipelines]同一アカウント＆同一リージョンに環境の数だけcdk pipelinesを実装する方法'
tags:
  - AWS
  - TypeScript
  - aws-cdk
  - CDK
private: false
updated_at: '2024-02-11T15:49:15+09:00'
id: ddda3fdb809712deb70d
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

複数人でのCDK開発を進める上でcdk pipelinesは非常に便利ですが、**同一アカウント＆同一リージョン上で環境別のcdk pipelinesを走らせる**といった要件で検討した際に

- gitのブランチ運用
- 環境変数の定義方法
- cdk pipelinesを動的に生成する為のスタック定義方法

など、運用上考慮した点や工夫した点があった為、実際の構築例を紹介する記事です。

## 対象

- 環境別（dev、test等）のcdk pipelinesを構築しようとお考えの方
- CDk内の環境変数を使いまわす方法を模索中の方

## 検証環境

cdk 2.50.0
node 18.12.1
typescript 4.8.4

## ゴール

今回は`test1`、`test2`という環境用のcdk pipelinesを構築する想定です。

▼実際に作成したpipeline
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/244ddc7c-8511-627a-0d90-262943288c28.png)

最終的にCloudFormationとしては2種類のスタックが出来上がります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/5f16fe39-3c61-e025-d991-78511f7c0dee.png)

- cdk pipelinesとして`〇〇(環境名)PipelinesStack`
- cdk pipelines内でデプロイされる`〇〇(環境名)PipelinesDeploy-CdkSampleStack`
※今回はECRリポジトリ一つだけ生成される`CdkSampleStack`をサンプルとして用意しています。

## ブランチの用意

**1環境=1ブランチ**の形でcdk pipelinesのトリガーとなるブランチを用意します。
今回はtest1用として`test1_env`、test2用として`test2_env`ブランチを用意しています。

## CodePipeline で Connection の作成

今回はGitHubとの連携でcdk pipelinesを構成する為、Connectionの作成をしています。
Connectionの作成方法に関してはこちら↓の記事を参考にしています。

<https://qiita.com/sugimount-a/items/540d130fe7cfea15bf13>

## 環境別のパラメータを用意

`cdk init`後に`cdk.json`に各環境に対応するパラメータを用意します。
パラメータ`stage`のデフォルト値を`dev`としています。

ローカルPCからcdkコマンドを実施する場合は、例えば`cdk diff -c stage=test1`といった形で`stage`のパラメータを指定することで、各環境に応じた操作を可能とします。

```json:cdk.json
{
  "app": "npx ts-node --prefer-ts-exts bin/cdk_sample.ts",
  "versionReporting": false,
  "watch": {
    "include": [
      "**"
    ],
    "exclude": [
      "README.md",
      "cdk*.json",
      "**/*.d.ts",
      "**/*.js",
      "tsconfig.json",
      "package*.json",
      "yarn.lock",
      "node_modules",
      "test"
    ]
  },
  "context": {
    "@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId": true,
    "@aws-cdk/core:stackRelativeExports": true,
    "@aws-cdk/aws-rds:lowercaseDbIdentifier": true,
    "@aws-cdk/aws-lambda:recognizeVersionProps": true,
    "@aws-cdk/aws-lambda:recognizeLayerVersion": true,
    "@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021": true,
    "@aws-cdk-containers/ecs-service-extensions:enableDefaultLogDriver": true,
    "@aws-cdk/aws-ec2:uniqueImdsv2TemplateName": true,
    "@aws-cdk/core:checkSecretUsage": true,
    "@aws-cdk/aws-iam:minimizePolicies": true,
    "@aws-cdk/aws-ecs:arnFormatIncludesClusterName": true,
    "@aws-cdk/core:validateSnapshotRemovalPolicy": true,
    "@aws-cdk/aws-codepipeline:crossAccountKeyAliasStackSafeResourceName": true,
    "@aws-cdk/aws-s3:createDefaultLoggingPolicy": true,
    "@aws-cdk/aws-sns-subscriptions:restrictSqsDescryption": true,
    "@aws-cdk/aws-apigateway:disableCloudWatchRole": true,
    "@aws-cdk/core:enablePartitionLiterals": true,
    "@aws-cdk/core:target-partitions": [
      "aws",
      "aws-cn"
    ],
    "stage" : "dev",
    "dev": {
      "envName": "dev",
      "repository": "yoyoyo-pg/cdkSample",
      "branch": "main",
      "connectionArn": "connectionArnを入力"
    },
    "test1": {
      "envName": "test1",
      "repository": "yoyoyo-pg/cdkSample",
      "branch": "test1_env",
      "connectionArn": "connectionArnを入力"
    },
    "test2": {
      "envName": "test2",
      "repository": "yoyoyo-pg/cdkSample",
      "branch": "test2_env",
      "connectionArn": "connectionArnを入力"
    }
  }
}
```

## 実装例

主に以下3ファイルを利用します。

- `cdk_sample.ts`・・・デプロイ対象のスタックの定義、環境変数の受け渡し
- `cdk_pipelines-stack.ts`・・・cdk pipelines用のスタック
- `cdk_sample-stack.ts`・・・cdk pipelines内でデプロイされるアプリケーションのサンプルスタック

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/8f2b822a-7209-0338-ce96-5dc42652478b.png)

## cdk_sample.ts

`stage`パラメータを取得し、`stage`パラメータの値に応じた各環境毎のパラメータを`cdk.json`から取得しています。

```typescript:cdk_sample.ts
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import { PipelinesStack } from '../lib/cdk_pipelines-stack';

const app = new cdk.App();
// cdk.jsonから環境別のパラメータを取得
const stage = app.node.tryGetContext('stage');
const context = app.node.tryGetContext(stage);

// cdk pipelines宣言時に、contextとして各種パラメータを渡す
new PipelinesStack(app, `${stage}PipelinesStack`, context, {
  env: { 
    account: process.env.CDK_DEFAULT_ACCOUNT, 
    region: process.env.CDK_DEFAULT_REGION 
  },
});
```

PipelineStackをnewする際に```new PipelinesStack(app, `${stage}PipelinesStack`, context, {```
と指定することで、`stage=○○`の値を基に、スタック名を動的に生成するようにしています。

これにより、環境別の`test1CdkPipelinesStack`、`test2CdkPipelinesStack`といったスタックが生成できます。

他にも、変数`context`をスタック宣言時に渡す事で、`cdk.json`内の各種パラメータをPipelinesStack内で再利用できるようにしています。

```json:cdk.json
    "test1": {
      "envName": "test1",
      "repository": "yoyoyo-pg/cdkSample",
      "branch": "test1_env",
      "connectionArn": "connectionArnを入力"
    },
```

## cdk_pipelines-stack.ts

各環境別にcdk pipelinesを生成する為のスタックです。
`constructor`として`context`を受け取れるようにしているのが、通常のスタックからの変更点となります。

```typescript:cdk_pipelines-stack.ts
import * as pipelines from "aws-cdk-lib/pipelines";
import { Construct } from 'constructs';
import { Stack, StackProps, Stage, StageProps } from 'aws-cdk-lib';
import { CdkSampleStack } from "./cdk_sample-stack";

// cdk pipelines用のStack
export class PipelinesStack extends Stack {
    constructor(scope: Construct, id: string, context: any, props?: StackProps) {
        super(scope, id, props);
        // cdk pipelines
        const cdkPipelines = new pipelines.CodePipeline(this, 'DevPipeline', {
            // クロスアカウントを利用しない
            crossAccountKeys: false,
            synth: new pipelines.CodeBuildStep('Synth', {
                // cdk.json内の値を利用
                input: pipelines.CodePipelineSource.connection(context.repository, context.branch, {
                    connectionArn: context.connectionArn,
                }),
                commands: ['npm ci', 'npm run build', 'npx cdk synth'],
            }),
        });
        // add stage
        cdkPipelines.addStage(
            new CdkDeploymentStage(this, `${context.envName}PipelinesDeploy`, props)
        );
    }
}

// CdkDeployment stageを定義
export class CdkDeploymentStage extends Stage {
    constructor(scope: Construct, id: string, props?: StageProps) {
        super(scope, id, props);
        const service = new CdkSampleStack(this, "CdkSampleStack");
    }
}
```

`context`として受け取った情報はGithubとcdk pipelinesの紐づけの為の設定値として、
ここでは`repository`、`branch`、`connectionArn`の受け渡しを行っています。

```typescript:cdk_pipelines-stack.ts
// cdk.json内の値を利用
input: pipelines.CodePipelineSource.connection(context.repository, context.branch, {
    connectionArn: context.connectionArn,
}),
```

また、cdk pipelines内からのスタックのデプロイステージを定義していますが、ここでも`context`の情報を利用しています。

```typescript:cdk_pipelines-stack.ts
// add stage
cdkPipelines.addStage(
    new CdkDeploymentStage(this, `${context.envName}PipelinesDeploy`, props)
);
```

ここで`envName + PipelinesDeploy`とすることで、「cdk pipelinesのスタック」と「cdk pipelinesからデプロイされたスタック」をCloudFormation上でセットで確認しやすいようにしています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/78795ad8-a7d2-ffd9-094c-496fb3be9c10.png)

## cdk_sample-stack.ts

cdk pipelines内でデプロイしているスタックです。

今回は特別なことをしている訳ではなく、サンプル用のECRのリポジトリを作成しています。

```typescript:cdk_sample-stack.ts
import * as ecr from 'aws-cdk-lib/aws-ecr';
import { Construct } from 'constructs';
import { Stack, StackProps } from 'aws-cdk-lib';

// アプリケーションサンプルのStack
export class CdkSampleStack extends Stack {
    constructor(scope: Construct, id: string, props?: StackProps) {
        super(scope, id, props);
        // サンプル用リポジトリ
        const sampleRepo = new ecr.Repository(this, `repo`);
    }
}
```

## 環境別のデプロイについて

ここからが今回の肝の部分です。

- まず、一番最初のcdk pipelines構築（手元PCから`cdk deploy`）の際は`stage`に各環境名を指定します。
  - `cdk deploy -c stage=test1`といった形です。
- また、先程用意した`test1_env`、`test2_env`ブランチでは`cdk.json`内の`stage`のデフォルト値をそれぞれ`dev`から`test1`、`test2`に変更してプッシュします。

```json:cdk.json
    "stage" : "dev",
    "dev": {
      "envName": "dev",
      "repository": "yoyoyo-pg/cdkSample",
      "branch": "master",
      "connectionArn": "connectionArnを入力"
    },
```

`stage`を変更しデフォルト値を各環境別にすることで、各ブランチにプッシュが走った際に正常にパイプラインが動作する事となります。

この運用における注意点は、例えば`test1_env`ブランチで作業し、それを`main`にマージといった手順を踏んでしまうと`main`ブランチの`stage`のパラメータも変更される事となってしまう点です。

なので、例としては

- 各環境共通で変更を加えたい場合
  - `main`から新しく`develop`ブランチを切り、変更後に`main`に対してマージ
  - その後、`main`から`test1_env`と`test2_env`に対してマージ
- `test1_env`から`main`や、`test2_env`から`main`に対するマージはしない

等のルールを設ける必要がある点には注意が必要です。

※ちなみに、`stage`のパラメータを環境別に書き換えなかった場合、cdk pipelines内のビルド時に以下エラーが発生してしまいます。

```powershell
No stacks match the name(s) test1PipelinesStack
[14:07:53] Error: No stacks match the name(s) test1PipelinesStack
    at CdkToolkit.validateStacksSelected (/usr/local/lib/node_modules/aws-cdk/lib/cdk-toolkit.ts:708:13)
    at CdkToolkit.selectStacksForDeploy (/usr/local/lib/node_modules/aws-cdk/lib/cdk-toolkit.ts:655:10)
    at CdkToolkit.deploy (/usr/local/lib/node_modules/aws-cdk/lib/cdk-toolkit.ts:143:29)
    at initCommandLine (/usr/local/lib/node_modules/aws-cdk/lib/cli.ts:358:12)
```

## おわりに

最終的なポイントとしては以下です。

- 同一アカウント＆同一リージョンの場合はスタック名識別の為の工夫が必要
  - その為、`cdk.json`に各環境別の値を入れておく
- ただし`cdk.json`に各環境別の値を入れておくだけだと、cdk pipelines上ではデフォルトの環境設定値でパイプラインが走ってしまう
  - 環境別にブランチを用意した上で、ブランチ別にcdk.jsonの`stage`の値を変更しておく

`cdk.json`を上手く活用することで、環境が異なる場合にも再利用しやすい形を実現出来そうです。

今回は「同一アカウント＆リージョン」という事で各ブランチ上でパラメータを分けるやり方としましたが、やはり環境別にアカウントが分けられたり、各環境に対して順にデプロイしていけるパイプラインが構築できると理想だと思います。

## 参考資料

<https://docs.aws.amazon.com/cdk/v2/guide/cdk_pipeline.html>
<https://qiita.com/sugimount-a/items/540d130fe7cfea15bf13>
<https://logmi.jp/tech/articles/326730>
