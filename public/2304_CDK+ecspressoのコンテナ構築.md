---
title: AWS CDK + ecspressoでAWSコンテナリソースの管理をラクにしよう！
tags:
  - ECS
  - aws-cdk
  - CDK
  - ecspresso
private: false
updated_at: '2024-03-16T08:45:41+09:00'
id: 5921801e7f674f4e1023
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

最近、AWS CDKでコンテナ関連のリソース構築をしておりますが、**コンテナリソースをどこまでCDKで実装するべきか**が大きな悩みの種でした。

今回、コンテナ周辺のリソースをCDK、コンテナ本体のリソースをecspressoで構築した為、構築内容の紹介となっています。

## ecspressoとは

**ECSサービス、タスクに関わる最小限のリソースをコード管理**する事ができるツールです。

https://github.com/kayac/ecspresso

CDKを使う中で「CDKでサービス定義、タスク定義をするのは運用上厳しい」と感じた為、

- CDKでサービス定義、タスク定義以外のリソースを構築
- ecspressoでサービス定義、タスク定義、デプロイにも利用
  - ecspressoの外部参照機能（後述）を利用し、CDKで定義したリソースを取り込み

といった構築の仕方をしています。

## 動作環境

cdk 2.59.0
ecspresso v2.1.0

## 構築内容

今回はサンプルアプリケーションとして

- パブリックサブネットにALB
- プライベートサブネットにnginxコンテナ

を構築していきます。

## 構築手順

大まかな構築手順は以下です。

- CDKで関連リソースを定義し、デプロイ
- ecspressoでサービス定義、タスク定義をし、デプロイ

## スタックの作成

まずはCDKのスタックを記載します。  
各種ファイルの記載は以下の通りです。

```typescript:bin/cdk-sample.ts
import { CdkEcspressoStack } from '../lib/cdk-ecspresso-stack';

const app = new cdk.App();
new CdkEcspressoStack(app,'CdkEcspressoStack');
```

スタックの宣言内容は以下です。

```typescript:lib/cdk-ecspresso-stack.ts
import { Stack, StackProps } from "aws-cdk-lib";
import { Role, ServicePrincipal } from "aws-cdk-lib/aws-iam";
import { Vpc } from 'aws-cdk-lib/aws-ec2'
import { Construct } from "constructs";
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as ssm from 'aws-cdk-lib/aws-ssm';
import * as elbv2 from 'aws-cdk-lib/aws-elasticloadbalancingv2';
import { Cluster } from "aws-cdk-lib/aws-ecs";
import { Repository } from "aws-cdk-lib/aws-ecr";

// CDK + ecspressoで構築するコンテナ関連リソース
export class CdkEcspressoStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id);
    // VPC
    const vpc = new Vpc(this, "Vpc", { maxAzs: 2 });
    const subnetIdList = vpc.privateSubnets.map(obj => obj.subnetId);
    // SG
    const albSg = new ec2.SecurityGroup(this, 'AlbSg', { vpc, allowAllOutbound: false });
    const containerSg = new ec2.SecurityGroup(this, 'ContainerSg', { vpc });
    albSg.addIngressRule(ec2.Peer.ipv4('0.0.0.0/0') , ec2.Port.tcp(8080)); // インバウンドを許可
    albSg.connections.allowTo(containerSg, ec2.Port.tcp(80));  // ALB ⇔ コンテナ間の通信を許可
    // ALB    
    const alb = new elbv2.ApplicationLoadBalancer(this, 'Alb', { vpc, internetFacing: true, securityGroup: albSg });
    // TG
    const containerTg = new elbv2.ApplicationTargetGroup(this, 'ContainerTg', { targetType: elbv2.TargetType.IP, port: 80, vpc });
    // ALBリスナー
    alb.addListener('Listener', { defaultTargetGroups: [containerTg], open: true, port: 8080 }); // 作成したTGをALBに紐づけ
    // ECSクラスタ
    const cluster = new Cluster(this, 'EcsCluster', { vpc, clusterName: 'cdk-ecspresso' });
    // タスクロール
    const taskRole = new Role(this, 'TaskRole', { assumedBy: new ServicePrincipal('ecs-tasks.amazonaws.com'), });
    // タスク実行ロール
    const taskExecRole = new Role(this, 'TaskExecRole', { assumedBy: new ServicePrincipal('ecs-tasks.amazonaws.com'), });
    // ロググループ
    const logGroup = new LogGroup(this, 'logGroup', {});
    // ECR
    const repository = new Repository(this, 'Repository', {});
    // タスク実行ロールに権限付与
    repository.grantPull(taskExecRole); // ECRのPULL権限
    logGroup.grantWrite(taskExecRole); // ログ吐き出し権限

    // SSMパラメータの設定
    new ssm.StringParameter(this, 'TaskRoleParam', { parameterName: '/ecs/cdk-ecspresso/task-role', stringValue: taskRole.roleArn });
    new ssm.StringParameter(this, 'TaskExecRoleParam', { parameterName: '/ecs/cdk-ecspresso/task-exec-role', stringValue: taskExecRole.roleArn });
    new ssm.StringParameter(this, 'ContainerSubnet1Param', { parameterName: '/ecs/cdk-ecspresso/subnet-id-a', stringValue: subnetIdList[0] });
    new ssm.StringParameter(this, 'ContainerSubnet2Param', { parameterName: '/ecs/cdk-ecspresso/subnet-id-c', stringValue: subnetIdList[1] });
    new ssm.StringParameter(this, 'ContainerSgParam', { parameterName: '/ecs/cdk-ecspresso/sg-id', stringValue: containerSg.securityGroupId });
    new ssm.StringParameter(this, 'ContainerTgParam', { parameterName: '/ecs/cdk-ecspresso/tg-arn', stringValue: containerTg.targetGroupArn });
    new ssm.StringParameter(this, 'LogGroupParam', { parameterName: '/ecs/cdk-ecspresso/log-group-name', stringValue: logGroup.logGroupName });
  }
}
```

**CDKではコンテナ周辺リソースを定義**という事で、VPC・サブネット・ALB・セキュリティグループやターゲットグループだけでなく、ECSクラスターや各種ロールも定義します。

ECRへのpush・pullやログ吐き出し等の権限付与を考えると、各種ロールやロググループはCDK側で明示的に生成すると便利だと考えています。

### CDKで宣言したリソースをecspressoで利用する為の工夫

AWS CDKの[ベストプラクティス](https://aws.amazon.com/jp/blogs/news/best-practices-for-developing-cloud-applications-with-aws-cdk/)の一つとして、**自動で生成されるリソース名を使用し、物理的な名前を使用しない**というものがあります。  

リソース名を明示する方が便利な場面もありますが、明示する事でリソースのUPDATE時に失敗してしまう事もあります。なので、ベストプラクティスにもある通り、可能ならば自動生成名で良いのかなと思います。

ただ**外部処理からの参照が必要な際に、決め打ちでリソース名・IDを指定できなくなる**点には注意が必要です。それこそスタックやコンストラクトといった、大きな単位での変更を加えた際に自動生成名が変わり、外部処理からの参照箇所も修正となると面倒なので、今回はecspresso v2からの新機能である「SSMパラメータストア参照」機能を利用しています。

その為、ecspressoでデプロイ時に使うパラメータをSSMパラメータとして保存しています。パラメータストアへの保存処理により、CDKによる自動生成の名称が変更になった時の差異を吸収する事が可能です。

```typescript
// SSMパラメータの設定
new ssm.StringParameter(this, 'TaskRoleParam', { parameterName: '/ecs/cdk-ecspresso/task-role', stringValue: taskRole.roleArn });
new ssm.StringParameter(this, 'TaskExecRoleParam', { parameterName: '/ecs/cdk-ecspresso/task-exec-role', stringValue: taskExecRole.roleArn });
new ssm.StringParameter(this, 'ContainerSubnet1Param', { parameterName: '/ecs/cdk-ecspresso/subnet-id-a', stringValue: subnetIdList[0] });
new ssm.StringParameter(this, 'ContainerSubnet2Param', { parameterName: '/ecs/cdk-ecspresso/subnet-id-c', stringValue: subnetIdList[1] });
new ssm.StringParameter(this, 'ContainerSgParam', { parameterName: '/ecs/cdk-ecspresso/sg-id', stringValue: containerSg.securityGroupId });
new ssm.StringParameter(this, 'ContainerTgParam', { parameterName: '/ecs/cdk-ecspresso/tg-arn', stringValue: containerTg.targetGroupArn });
new ssm.StringParameter(this, 'LogGroupParam', { parameterName: '/ecs/cdk-ecspresso/log-group-name', stringValue: logGroup.logGroupName });
```

## スタックの構築

まずは`cdk deploy`でスタックを構築します。

```powershell
cdk deploy CdkEcspressoStack 
```

VSCode上では以下の形でログが出ます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/a5bd9d84-0b83-2644-5809-b22e9594b3ea.png)

CloudFormation上の表示は以下。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/2aa920a0-32ae-8e20-c21a-7e7b7cd812de.png)

パラメータストアにも値が保管されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/c826e8d9-2281-43ea-3890-01b88afa3b2c.png)

## ecspresso用のファイルを準備

デプロイの為に、以下3つのファイルを用意します。

```yaml:ecspresso.yml
region: ap-northeast-1
cluster: cdk-ecspresso
service: nginx
service_definition: ecs-service-def.json
task_definition: ecs-task-def.json
timeout: "10m0s"
plugins:
  - name: ssm
```

以下はサービス定義設定ファイルです。

ターゲットグループARN、セキュリティグループID、サブネットIDはパラメータストアから取得しています。

```ecs-service-def.json
{
  "deploymentConfiguration": {
    "deploymentCircuitBreaker": {
      "enable": false,
      "rollback": false
    },
    "maximumPercent": 200,
    "minimumHealthyPercent": 100
  },
  "desiredCount": 1,
  "enableECSManagedTags": false,
  "healthCheckGracePeriodSeconds": 0,
  "launchType": "FARGATE",
  "loadBalancers": [
    {
      "containerName": "nginx",
      "containerPort": 80,
      "targetGroupArn": "{{ ssm `/ecs/cdk-ecspresso/tg-arn` }}"
    }
  ],
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "securityGroups": [
        "{{ ssm `/ecs/cdk-ecspresso/sg-id` }}"
      ],
      "subnets": [
        "{{ ssm `/ecs/cdk-ecspresso/subnet-id-a` }}",
        "{{ ssm `/ecs/cdk-ecspresso/subnet-id-c` }}"
      ]
    }
  },
  "placementConstraints": [],
  "placementStrategy": [],
  "platformVersion": "LATEST",
  "schedulingStrategy": "REPLICA",
  "serviceRegistries": []
}
```

以下はタスク定義設定ファイルです。

ロググループ名、タスク実行ロールARN、タスクロールARNはパラメータストアから取得しています。

```ecs-task-def.json
{
  "containerDefinitions": [
    {
      "name": "nginx",
      "image": "nginx:latest",
      "environment": [
        {
          "name": "AWS_REGION",
          "value": "ap-northeast-1"
        },
        {
          "name": "TZ",
          "value": "Asia/Tokyo"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "{{ ssm `/ecs/cdk-ecspresso/log-group-name` }}",
          "awslogs-region": "ap-northeast-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80,
          "protocol": "tcp"
        }
      ]
    }
  ],
  "cpu": "256",
  "executionRoleArn": "{{ ssm `/ecs/cdk-ecspresso/task-exec-role` }}",
  "family": "cdk-ecspresso",
  "memory": "512",
  "networkMode": "awsvpc",
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "taskRoleArn": "{{ ssm `/ecs/cdk-ecspresso/task-role` }}"
}
```

```"{​​{​​ ssm `/ecs/cdk-ecspresso/sg-id` }​​}​​"```といった形でSSMパラメータストア参照を利用しています。

ecspressoでのSSMパラメータストア参照については、以下記事にIAMの設定内容など記載しましたので、宜しければご覧ください。

https://qiita.com/yoyoyo_pg/items/1b562124e6b5d62f57d4

## ecspressoでサービス定義、タスク定義をデプロイ

最後に`ecspresso deploy`をして、ターミナル上で実行結果を見守ります。

`Service is stable now. Completed!`となり、デプロイ成功となります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/c1b55789-64f8-63a5-5b0a-46bee332aae2.png)

ブラウザ上からアクセスし確認
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/a78ac86a-bfc5-8cbd-6781-0eda6447a253.png)

## さいごに

- 「CDKやCloudFormationでECSのサービス定義、タスク定義まで構築しているが、どうも運用しにくい...」と感じている方や、既存リソースがある程度定義されていて「コンテナの最低限のリソースをコード化＆デプロイに使いたい」となった際に、ecspressoは非常に便利なツールです。
- SSMのパラメータストア参照の他、TerraformやCloudFormationのパラメータを参照する事も可能な為、既存のIaCに組み込みやすい点も、ecspressoの良い点だと感じています。

## 20230503追記

上記デプロイをハンズオン形式で簡単にお試し頂けるように、GitHubリポジトリを公開しました。
<https://github.com/yoyoyo-pg/cdk-ecspresso>

## 参考文献

<https://github.com/kayac/ecspresso>
<https://zenn.dev/fujiwara/books/ecspresso-handbook-v2>
