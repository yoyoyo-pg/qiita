---
title: AWS CDKのgrantメソッドが便利すぎた件
tags:
  - AWS
  - ECR
  - aws-cdk
  - CDK
private: false
updated_at: '2024-02-11T16:21:56+09:00'
id: 2384b57c03b28de864e0
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

最近、CDKのベストプラクティスを久しぶりに読み返していたのですが、`grant`メソッドなる存在を知りました。

<https://aws.amazon.com/jp/blogs/news/best-practices-for-developing-cloud-applications-with-aws-cdk/>

こちらのメソッドを利用する事で、例えばCDKのリソース定義時に

- インラインポリシーを記載
- リソースにアタッチ

をしなくとも、IAMのインラインポリシーやアクセスポリシーの実装が可能です。

今回はこちらの`grant`を利用し、記述量の削減をした例をご紹介します。

## 対象リソース

今回構築するリソースは以下。ECSの周辺リソースを対象としています。

- ECRリポジトリA
- ロググループB
- ECSのタスク実行ロール
  - リポジトリAのPULL権限を付与
  - ロググループBに対するログ吐き出しの権限を付与

## リソース定義

まずは権限以外の部分を構築します。  
最低限必要なのは以下記述です。

```typescript:sample-stack.ts
export class sampleStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id);
    // ECR
    const repository = new Repository(this, 'Repository', {});
    // ロググループ
    const logGroup = new LogGroup(this, 'LogGroup', {});
    // タスク実行ロール
    const taskExecRole = new Role(this, 'TaskExecRole', { assumedBy: new ServicePrincipal('ecs-tasks.amazonaws.com') });
  }
}
```

## 付与したい権限

最終的に、タスク実行ロールに対して以下ポリシーがアタッチされている状態を目指します。

```json:Policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:BatchGetImage",
                "ecr:GetDownloadUrlForLayer"
            ],
            "Resource": "arn:aws:ecr:XXXXXXXXXX:XXXXXXXXXX:repository/XXXXXXXXXX",
            "Effect": "Allow"
        },
        {
            "Action": "ecr:GetAuthorizationToken",
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:XXXXXXXXXX:XXXXXXXXXX:log-group:XXXXXXXXXX:*",
            "Effect": "Allow"
        }
    ]
}
```

## 権限付与の為の関数を用意

今回は権限付与を関数`addPolicyToTaskExecRole`として定義しています。

引数として、先に宣言した`taskExecRole`、`repository`、`logGroup`を渡します。

```typescript:sample-stack.ts
    // 各種ポリシーを追加
    addPolicyToTaskExecRole(taskExecRole, repository, logGroup); 
```

## grantを利用しない場合

各種ポリシー付与の例は以下です。IAMコンソール上から直に値を入力するのと変わらない位の手間がかかります。

```typescript
/** タスク実行ロールに各種権限を付与 */
const addPolicyToTaskExecRole = (taskExecRole : Role, repository : Repository, logGroup : LogGroup) => {
  // ECRのPULL用
  taskExecRole.addToPolicy(
    new PolicyStatement({
      actions: ['ecr:BatchCheckLayerAvailability','ecr:BatchGetImage','ecr:GetDownloadUrlForLayer'],
      resources: [repository.repositoryArn]
    })
  );
  taskExecRole.addToPolicy(
    new PolicyStatement({
      actions: ['ecr:GetAuthorizationToken'],
      resources: ['*']
    })
  );
  // ログ吐き出し用
  taskExecRole.addToPolicy(
    new PolicyStatement({
      actions: ['logs:CreateLogStream','logs:PutLogEvents'],
      resources: [logGroup.logGroupArn]
    })
  );
}
```

## grantを利用する場合

`grant`を利用する場合、なんと上記の記述がたった2行で済みます。

```typescript
/** タスク実行ロールに各種権限を付与 */
export const addPolicyForTaskExecRole = (taskExecRole : Role, repository : Repository, logGroup : LogGroup) => {
  // ECRのPULL用
  repository.grantPull(taskExecRole);
  // ログ吐き出し用
  logGroup.grantWrite(taskExecRole);
}
```

Repositoryコンストラクトには`grantPull(granttee)`、`grantPullPush(granttee)`といったメソッドが用意されています。
<https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ecr.Repository.html>

LogGroupコンストラクトには`grantRead(grantee)`、`grantWrite(grantee)`といったメソッドが用意されています。
<https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_logs.LogGroup.html>

## おわりに

- CDKに権限の生成を任せられる
- コード量の削減に繋がる
- 構築の手間を省ける

等々、メリット満載だと感じた為、今後は積極的に使っていきたい機能です。  

ただし、権限を広めに与えてしまう可能性もあります。細かな制御が必要な場合は`addToPolicy`で対応した方が良い場面もあるかもしれません。
