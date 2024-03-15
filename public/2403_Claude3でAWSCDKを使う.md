---
title: Claude 3とAWS CDKを使い爆速でAWSの検証が出来る環境を手に入れよう！
tags:
  - AWS
  - SNS
  - lambda
  - CDK
  - claude3
private: false
updated_at: '2024-03-15T23:41:17+09:00'
id: dc3c0e5f4c9af9be9214
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

- 今回は、今話題の [Claude 3](https://claude.ai/) で[AWS CDK](https://aws.amazon.com/jp/cdk/)というAWSインフラのプロビジョニングツールを使い、簡易的なメール通知の仕組みを作成しました。
- 基本的にはClaude 3に聞いて作っておりAWS CDKの最初のセットアップ以外はコードを全く書いていませんので、基本的にどなたにもお試し頂ける構成だと思います。

## 技術要素の紹介

### Claude 3

2024年3月4日に発表されたAnthropic社の最新モデルの生成AIです。

https://qiita.com/minorun365/items/e6f3aa71f5e1bdf21139

- 特に驚いたのは「マルチモーダル」対応という事で、画像やPDFの分析もしてくれます。
- [anthropic.com](https://www.anthropic.com/)に登録すると、Claude 3 Sonnetをお試し頂けます。
  - 今回はこちらを利用し、AWS CDKのコードを出力しています。

### AWS CDK（AWS Cloud Development Kit）

- 一言で表すと、プログラム言語（ex. TypeScirpt、Python、Java、Go）でAWSリソースを定義できるツールで、IaC（Infrastructure as Code）を実現する手段として用いられます。
- [2019年7月にv1](https://docs.aws.amazon.com/ja_jp/cdk/v1/guide/doc-history.html)が、[2021年12月にv2](https://docs.aws.amazon.com/ja_jp/cdk/v2/guide/doc-history.html)がリリースされています。
- CDKを実行すると、裏側ではAWS CloudFormationを用いたリソースデプロイが実行されます。
- CDKを用いる事で、CloudFormationよりもより少ないコード量かつ抽象化した記述が可能です。

https://qiita.com/Brutus/items/6c8d9bfaab7af53d154a

https://qiita.com/yoyoyo_pg/items/e7e85839341d16006b87

### Claude 3 で使われる AWS CDK について

- 私はCDK v2になってから使い始めたのですが、以前利用していたChatGPT（GPT-3.5）は、CDK v2リリースから日が浅いのか、CDK v2のコードを出力するには少し物足りない印象がありました。
- 今回は、学習データも溜まっていそうだという期待を込め、Claude 3にCDKの出力を依頼してみた背景もあります。
  - なお、Claude 3に直接尋ねてみると、2023年8月時点までの情報を利用しているとの返答があります。
  - [github - aws/aws-cdkのrelease](https://github.com/aws/aws-cdk/releases)を見る限りだと、2023年の8月には既にv2.90.0がリリースされている為、少なくともCDK v2の情報を保持している事が分かります。

### AWS Lambda

- サーバレスなコンピューティングサービスです。
- 今回は、Lambdaの中でSNSを呼び出す処理を、CDKを用いてデプロイしています。

https://aws.amazon.com/jp/lambda/

### AWS SNS

- フルマネージド型のメッセージング・モバイル通知サービスです。
- SNSトピックを設定し、そこに対してサブスクライブという形で今回はメールアドレスを登録します。
- 今回は、上記の設定をCDKを用いてデプロイしています。

https://aws.amazon.com/jp/sns/

## 構成

今回の構成は以下です。

- AWS SNSトピックを作成
- トピックへのサブスクライブの作成
  - 送信先メールアドレスの登録
- Lambdaの作成
  - トピックに対して`publish`を行い、SNSでメール送信

![application-composer.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/910ad731-76d3-fd44-2617-76f41ff16b6f.png)

### 構成図の出力について

- 今回の構成図はVSCodeのAWS Toolkitの拡張機能を使い、Application Composerを用いる事で、CloudeFormationのテンプレートを可視化しています。
- リソースの関連が一目で分かるので、便利です。

https://www.d-make.co.jp/blog/2023/12/19/aws-application-composer-in-vscode/

## CDKコードの準備

### 下準備

- CDKプロジェクト用のディレクトリ作成と、`cdk init`を済ませます。
- こちらに関しては、以下ドキュメントが参考になります。

https://docs.aws.amazon.com/ja_jp/cdk/v2/guide/hello_world.html

なお、今回構築したサンプルコードはGitHubリポジトリ上に公開しています。

https://github.com/yoyoyo-pg/sns-sample

### CDKコードの追記

- SNSとLambdaのCDKコードです。
- 命名以外は、基本的にそのままClaude 3から吐き出された物を利用しています。

```sns-sample-stack.ts
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as sns from 'aws-cdk-lib/aws-sns';
import * as subscriptions from 'aws-cdk-lib/aws-sns-subscriptions';
import { NodejsFunction } from 'aws-cdk-lib/aws-lambda-nodejs';

export class SnsSampleStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    // SNSトピックを作成
    const topic = new sns.Topic(this, 'Topic', {
      topicName: 'sampleTopic',
    });
    // メール送信先の設定
    topic.addSubscription(new subscriptions.EmailSubscription('yoyoyo-pg@gmail.com'));
    // ARNをアウトプット
    new cdk.CfnOutput(this, 'TopicArn', {
      value: topic.topicArn,
      description: 'Topic ARN',
      exportName: 'TopicArn',
    });
    // Lambdaを作成してSNSトピックにメッセージを送信
    const sendEmailFunction = new NodejsFunction(this, 'SendEmailFunction', {
      entry: 'lambda/sendEmail.ts',
      handler: 'handler',
      environment: {
        TOPIC_ARN: topic.topicArn,
      },
    });
    // Lambdaにトピックへの公開権限を付与
    topic.grantPublish(sendEmailFunction);
  }
}
```

- Lambda内のコードです。
- 今回のLambdaランタイムはNode.jsを利用しています。
- こちらに関しては、Claudeから出力されたものに対して、一切手を加えていません。

```sendEmaill.ts
import { SNSClient, PublishCommand } from '@aws-sdk/client-sns';

const snsClient = new SNSClient({});

export const handler = async (event: any) => {
  const topicArn = process.env.TOPIC_ARN;

  const params = {
    Message: 'Hello, this is a notification from Lambda!',
    TopicArn: topicArn,
  };

  try {
    const data = await snsClient.send(new PublishCommand(params));
    console.log('Message sent successfully:', data);
  } catch (err) {
    console.error('Error sending message:', err);
  }
};
```

## CDKを用いたデプロイ

### cdk deployの実施

- `cdk deploy スタック名`というコマンドを叩く事で、CloudFormationのスタックデプロイを実行できます。

:::note info
今回は`NodejsFunction`というCDKのコンストラクトを利用しており、その関係でローカル環境にdockerが必要です。
:::

![cdk-deploy.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/81037d7d-c449-d616-a334-057518b79d92.png)

- デプロイ完了時

![deploy-complete.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/62e7078b-9f4b-5861-545c-fc4bb1c3b95e.png)

- この時点で、上記に示した構成がCloudFormationのスタックとしてデプロイされています。
- 実はこれもCDKの凄い所なのですが、明示していないLambdaの実行ロールも自動生成してくれます。

![cfn.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/41798f45-5f1b-4199-7583-41365340dde1.png)

### SNSサブスクライブの確認

- なお、SNSのトピックを見てみると、メールアドレスが保留中の状態となっています。

![topic-notauth.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/2ac01401-14e1-70e2-43b9-173e9418fbaf.png)

- 実際に対象のメールアドレスに確認のメールが届くので、`Confirm subscription`を選択します。

![confirm-subscription.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/c33eee28-baf2-1fbf-8915-a54f93d28ef8.png)

- `Confirm subscription`を選択すると、サブスクライブが成功します。

![subscription-complete.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/37898b5f-34bc-c7c8-2f6b-3b02e5095e56.png)

![topic-auth.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/f1176aaf-edf8-b838-d460-f2c95376ea61.png)

- この状態で、Lambda関数の実行をすると、メール送信が可能となります。

![send-success.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/9e2523b4-7244-2cd2-f89f-734cc9d6b684.png)

## おわりに

- 今回、AWS SNSも、Lambdaの`NodejsFunction`も初めて使ってみたのですが、背景知識さえあればCDKのドキュメントを読むことなく実装を完了させることが出来ました。
- CDK自体も、ちょっとしたサービスの検証をする上で既に便利なのですが、生成AIの力を借りる事で、より構築のスピードを早めていく事が可能だと思います。
- これを機に、AWS CDKがより広まると嬉しいなと思います。

## 参考文献

https://claude.ai/

https://aws.amazon.com/jp/cdk/

https://qiita.com/Brutus/items/6c8d9bfaab7af53d154a

https://qiita.com/yoyoyo_pg/items/e7e85839341d16006b87

https://github.com/aws/aws-cdk/

https://aws.amazon.com/jp/lambda/

https://aws.amazon.com/jp/sns/

https://www.d-make.co.jp/blog/2023/12/19/aws-application-composer-in-vscode/

https://docs.aws.amazon.com/ja_jp/cdk/v2/guide/hello_world.html

https://github.com/yoyoyo-pg/sns-sample
