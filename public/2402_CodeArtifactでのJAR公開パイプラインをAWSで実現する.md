---
title: CodeArtifactでのJAR公開パイプラインをAWSで実現する
tags:
  - gradle
  - CodePipeline
  - CodeBuild
  - CDK
  - CodeArtifact
private: false
updated_at: '2024-03-15T12:39:27+09:00'
id: 1647d65f5b4ae4ae4270
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

- CodeArtifact上のMavenプライベートリポジトリに対して、GitHubへのPushにより、JARファイルの公開が可能なパイプラインを今回構築しました。
- アーキテクチャの紹介とインフラコードの紹介を行います。

## 構築したもの

![gradle-publish-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/734927cb-034b-fd54-53f0-dd19b254dffa.png)

- インフラ構成(AWS CDK)とアプリケーションコード(Gradleプロジェクト)をセットでGitHub上のリポジトリで保持しています。
  - リポジトリは[こちら](https://github.com/yoyoyo-pg/gradle-publish-sample)
- リポジトリの`main`ブランチにソースコードがプッシュされると、パイプラインが動いてJARがCodeArtifact上のリポジトリに`publish`されます。
- 今回構築している、CodeArtifact・Codestar connections・CodePipeline・CodeBuildの4つのリソースにより、上記の機能が実現されています。

## CodeArtifact

> フルマネージドのアーティファクトリポジトリサービスである AWS CodeArtifact を使用すると、組織はアプリケーション開発に使用するソフトウェアパッケージを安全に保存および共有できます。CodeArtifactは、NuGet CLI、Maven、Gradle、npm、yarn、pip、twine などの一般的なビルドツールやパッケージマネージャーで使用できます。

[参考](https://docs.aws.amazon.com/ja_jp/codeartifact/latest/ug/welcome.html)

- 普段だとローカルのPCで保持するようなパッケージ（npmやMavenなど）を、共有のパッケージリポジトリとして公開する事が可能です。
- リポジトリのエンドポイントを指定する事で、リポジトリに対してパッケージの公開や、リポジトリ内のパッケージの利用が可能です。
- ドメインと呼ばれるグループの中に、パッケージを保持するリポジトリ作成する形です。
- 使用例として、チーム開発や個人開発を想定し、プライベートなJarライブラリとしての利用する事が可能です。

### 今回構築したCodeArtifactのリポジトリ

- `yoyoyo-pg`ドメインの中に`gradle-publish-sample`リポジトリを用意しています。
- 今回構築しているJAR公開のパイプラインを通して、こちらのリポジトリにJARが公開されます。

![codeartifact1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/7dc84800-64c2-a458-8094-940b0f25e9c5.png)

## Codestar connections

> デベロッパーツールコンソールの接続昨日を使用して、AWS CodePipeline などの AWS リソースを外部コードリポジトリに接続します。
・・・
例えば、CodePipeline で接続を追加して、サードパーティーのコードリポジトリでコードが変更されたときにパイプラインをトリガーできるようになります。

[参考](https://docs.aws.amazon.com/ja_jp/dtconsole/latest/userguide/welcome-connections.html)

- サードパーティーのコードリポジトリとの接続設定を行う事が出来ます。
- コードリポジトリ内の変更があった際に、連携しているCodePipelineをトリガー出来るようになります。

### 今回構築した接続設定

- 今回は`yoyoyo-pg-connection`という名前で接続設定を作成しています。
- こちらの接続設定を用いて、GitHubリポジトリ上のソースコードをCodePipelineで取得しています。
- GitHubの場合は、一度接続設定を作成した後に「GitHub側」で許可の操作をする必要があります。
  - この操作については後の構築手順で解説します。

![codestar-connections.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/2b6b4db5-cc3b-bd90-cc2d-42ee4133d390.png)

## CodePipeline

> AWS CodePipeline は、ソフトウェアをリリースするために必要なステップのモデル化、視覚化、および自動化に使用できる継続的な配信サービスです。

- ソースの取得、プロジェクトのビルドやテスト、実行環境へのデプロイを取りまとめる事が可能です。
- 今回利用するCodeBuildや、他にもCodeDeployなどAWSマネージドのサービスと連携する事が可能です。

[参考](https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/welcome.html)

### 今回構築したパイプライン

- GitHubからのソース取得を`Source`ステージで行い、CodeBuildの呼び出しを`Build`ステージで行います。
- `Source`ステージ
  - 先程の接続設定を用いて、GitHubからソースコードの取得を行います。
- `Build`ステージ
  - 後述するCodeBuildを呼び出し、Gradleプロジェクトとして構成されているソースコードのビルドとJARの公開処理を行います。

![codepipeline.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/5acadb15-861b-0d78-0743-6f975ca812f2.png)

## CodeBuild

> AWS CodeBuild はクラウドで動作する、完全マネージド型のビルドサービスです。CodeBuild はソースコードをコンパイルし、単体テストを実行して、すぐにデプロイできるアーティファクトを生成します。

- サーバーの用意が必要なく、オンデマンドかつビルドスクリプトさえ用意出来ればすぐに使える点が利点とされています。
- `Buildspec`を用意する事で、そこに記載のあるコマンドをCodeBuild上で実行してくれます。

[参考](https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/welcome.html)

## 今回構築したビルド処理

- CodeBuildとしては主に以下二つの処理をしています。
  - CodeArtifactのトークンを取得
  - その後CodeArtifactに対してJARを公開
- `Buildspec`の内容は以下です。

```yaml:buildspec.yml
version: 0.2
phases:
  install:
    runtime-versions:
      java: corretto17
  pre_build:
    on-failure: ABORT
    commands:
      - export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain yoyoyo-pg --domain-owner $AWS_ACCOUNT_ID --region ap-northeast-1 --query authorizationToken --output text`
      - export CODEARTIFACT_REPO_URL=$CODEARTIFACT_REPO_URL
  build:
    on-failure: ABORT
    commands:
      - cd $CODEBUILD_SRC_DIR/gradle-sample
      - gradle publishAllPublicationsToMavenRepository
```

- CodeBuildの他の構築内容については後述します。

---

## CDKリポジトリの紹介

上記構成についての初期構築、初期パイプラインの起動時の手順を紹介する前に、今回用意したCDKコードの紹介を進めていきます。

### ディレクトリの構成

- 先にも紹介した通り、インフラ構成(AWS CDK)とアプリケーションコード(Gradleプロジェクト)をセットでGitHub上のリポジトリで保持しています。
- 通常のAWS CDKのプロジェクトに加え、`gradle-sample`配下がGradleプロジェクトとなっております。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/6a0d53ba-5f7d-daaf-ca02-383ba59a4241.png)

## CDKのスタックとコンストラクトの構成

- `GradlePublishSampleStack`の中で、今回構築する4つのリソースをカスタムコンストラクトとして用意しています。

```typescript:bin/gradle-publish-sample.ts
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import { GradlePublishSampleStack } from '../lib/gradle-publish-sample-stack';

const app = new cdk.App();
new GradlePublishSampleStack(app, 'GradlePublishSampleStack', {
  env: { account: process.env.CDK_DEFAULT_ACCOUNT, region: process.env.CDK_DEFAULT_REGION },
});
```

- カスタムコンストラクトは`CodeStar`、`CodeArtifact`、`CodeBuild`、`CodePipeline`の4つです。

```typescript:lib/gradle-publish-sample-stack.ts
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import { Codebuild } from './codebuild';
import { CodePipeline } from './codepipeline';
import { CodeArtifact } from './codeartifact';
import { CodeStar } from './codestar';

export class GradlePublishSampleStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id);
    // Connection
    const codeStar = new CodeStar(this, 'CodeStar', {})
    // CodeArtifact
    new CodeArtifact(this, 'CodeArtifact', {});
    // CodeBuild
    const codeBuild = new Codebuild(this, 'CodeBuild', {});
    // CodePipeline
    new CodePipeline(this, 'CodePipeline', { 
      buildProject: codeBuild.buildProject,
      CfnConnection : codeStar.CfnConnection
    });
  }
}
```

### 各コンストラクトの記載

#### codestar.ts

- L1コンストラクト`CfnConnection`を利用し、接続設定を作成しています。
- 接続設定としてCodePipelineで利用する為、`CfnConnection`として外部から参照できるようにしています。

```typescript
// 外部からの参照用リソース
readonly CfnConnection : CfnConnection
...
this.CfnConnection = codeStarConnection;
```

```typescript:codestar.ts
import { StackProps } from "aws-cdk-lib";
import { Construct } from "constructs";
import { CfnConnection } from 'aws-cdk-lib/aws-codestarconnections';

/** CodeStar */
export class CodeStar extends Construct {
  // 外部からの参照用リソース
  readonly CfnConnection : CfnConnection

  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id);

    // Resources
    const codeStarConnection = new CfnConnection(this, 'Connection', {
      connectionName: 'yoyoyo-pg-connection',
      providerType: 'GitHub',
    });

    this.CfnConnection = codeStarConnection;
  }
}
```

#### codeartifact.ts

- L1コンストラクト`CfnDomain`と`CfnRepository`を利用し、CodeArtifactのドメインとリポジトリを定義しています。

```typescript:codeartifact.ts
import * as codeartifact from 'aws-cdk-lib/aws-codeartifact';
import { Construct } from "constructs";
import { StackProps } from 'aws-cdk-lib';

/** CodeArtifact */
export class CodeArtifact extends Construct {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id);
    // CodeArtifact domain
    const codeArtifactDomain = new codeartifact.CfnDomain(this, 'Domain', {domainName: 'yoyoyo-pg'});
    // Maven Repo
    const MavenPrivateRepo = new codeartifact.CfnRepository(this, 'GradleSample', {
      repositoryName: 'gradle-publish-sample',
      description: 'gradle publish repository',
      domainName: codeArtifactDomain.attrName,
    });
  }
}
```

#### codebuild.ts

- L2コンストラクト`PipelineProject`を利用し、CodeBuildのビルドプロジェクトを定義しています。
- CodeArtifactに対するトークン発行と公開も行う為、実行IAMロールに対する権限付与も`attachInlinePolicy`で定義しています。

```typescript
buildRole.attachInlinePolicy(codeArtifactAccessPolicy);
```

```typescript:codebuild.ts
import * as codebuild from 'aws-cdk-lib/aws-codebuild';
import { Construct } from "constructs";
import { IProject } from 'aws-cdk-lib/aws-codebuild';
import { Policy, PolicyStatement, Role, ServicePrincipal } from 'aws-cdk-lib/aws-iam';
import { buildSpecObject } from './buildspec';
import { StackProps } from 'aws-cdk-lib';

/** Codebuild */
export class Codebuild extends Construct {
  // 外部からの参照用リソース
  readonly buildProject : IProject

  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id);
    // Role
    const buildRole = new Role(this, 'Role', { assumedBy: new ServicePrincipal('codebuild.amazonaws.com') });
    // BuildProject
    const buildProject = new codebuild.PipelineProject(this, 'CodeBuild', {
      buildSpec: codebuild.BuildSpec.fromObjectToYaml(buildSpecObject),
      role: buildRole,
      environment: {
        buildImage: codebuild.LinuxBuildImage.fromCodeBuildImageId('aws/codebuild/amazonlinux2-x86_64-standard:4.0'),
        privileged: false,
        environmentVariables: {
          AWS_ACCOUNT_ID: { 
            value: process.env.CDK_DEFAULT_ACCOUNT
          },
          CODEARTIFACT_REPO_URL: {
            value: `https://yoyoyo-pg-${process.env.CDK_DEFAULT_ACCOUNT}.d.codeartifact.ap-northeast-1.amazonaws.com/maven/gradle-publish-sample/`
          }
        }
      },
      description: 'gradle-sample'
    });
    // CodeArtifactAccessPolicy
    const codeArtifactAccessPolicy = new Policy(this, 'CodeArtifactAccessPolicy', { 
        policyName: 'codeArtifactAccessPolicy',
        statements: [
          new PolicyStatement({
            actions: [
              "codeartifact:GetAuthorizationToken",
              "codeartifact:GetRepositoryEndpoin",
              "codeartifact:ReadFromRepository",
              "codeartifact:PublishPackageVersion",
              "codeartifact:PutPackageMetadata",
            ],
            resources: ["*"]
          }),
          new PolicyStatement({
            actions: ["sts:GetServiceBearerToken"],
            resources: ["*"],
            conditions: {
              'StringEquals': {
                'sts:AWSServiceName': 'codeartifact.amazonaws.com',
              },
            },
          })
        ] 
    });
    buildRole.attachInlinePolicy(codeArtifactAccessPolicy);

    this.buildProject = buildProject;
  }
}
```

#### codepipeline.ts

- L2コンストラクト`Pipeline`を利用し、接続設定を利用する事でGitHubからのソース取得を可能としている`Source`ステージと、CodeBuildを呼び出す為の`Build`ステージを定義しています。
- `props.CfnConnection.attrConnectionArn`という形で接続設定のArnを取得する事で、ソースコード上にAWSアカウントIDをべた書きしなくて良いようにしています。

```typescript:codepipeline.ts
import { Construct } from "constructs";
import { CfnConnection } from 'aws-cdk-lib/aws-codestarconnections';
import { CodeStarConnectionsSourceAction, CodeBuildAction, CodeBuildActionType } from 'aws-cdk-lib/aws-codepipeline-actions';
import { IProject } from 'aws-cdk-lib/aws-codebuild';
import { Artifact, Pipeline } from "aws-cdk-lib/aws-codepipeline";

/** 入力インターフェース */
export interface constructProps  {
  buildProject : IProject;
  CfnConnection : CfnConnection
}

/** Codepipeline */
export class CodePipeline extends Construct {
  constructor(scope: Construct, id: string, props: constructProps) {
    super(scope, id);
    // アーティファクト定義
    const sourceOutput = new Artifact(); // SourceAction
    const buildOutput = new Artifact(); // BuildAction
    // CodePipeline
    new Pipeline(this, 'Pipeline', {
      crossAccountKeys: false,
      stages: [
        {
          stageName: 'Source',
          actions: [
            new CodeStarConnectionsSourceAction({
              actionName: 'Source',
              owner: 'yoyoyo-pg',
              repo: 'gradle-publish-sample',
              branch: 'main',
              output: sourceOutput,
              connectionArn: props.CfnConnection.attrConnectionArn,
              triggerOnPush: true,
            }),
          ],
        },
        {
          stageName: 'Build',
          actions: [
            new CodeBuildAction({
              actionName: 'Build',
              input: sourceOutput,
              project: props.buildProject,
              type: CodeBuildActionType.BUILD,
              outputs: [buildOutput]
            }),
          ],
        }
      ]
    });
  }
}
```

## Gradleプロジェクト内の設定内容

### Gradleプロジェクトのディレクトリ

- `lib`配下の`build.gradle`が、Gradleでのビルドに関する主なファイルとなっています。
- `lib/src`配下にJavaクラスファイルが記載されています。

![gradle-dir.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/b2c1f0c2-df38-fd71-0300-87449ae66317.png)

### CodeBuild側からの呼び出し内容

- CodeBuildの`Buildspec`では、以下の形でディレクトリ`gradle-sample`へと移動し、JAR公開のコマンド`publishAllPublicationsToMavenRepository`を実行しています。

```yaml:buildspec.yml
    commands:
      - cd $CODEBUILD_SRC_DIR/gradle-sample
      - gradle publishAllPublicationsToMavenRepository
```

### build.gradleの設定

CodeArtifactへの公開処理が行われる際の設定として用いられる`build.gradle`について紹介します。

- `publications`では、Mavenリポジトリに対する公開処理なので、`groupId`、`artifactId`、`version`を指定しています。
  - この内容で公開されるので、逆にこのJARを利用する場合にも同じ指定をする必要があります。
- `repositories`では、対象のCodeArtifactのURLとトークンを指定しています。
  - どちらもCodeBuild上から値を取得できるような工夫をしています。

```build.gradle
publishing {
  // Used publishToMavenLocal and publish to CodeArtifact
  publications {
      maven(MavenPublication) {
          groupId = 'com.sample.yoyoyo-pg'
          artifactId = 'jarSample'
          version = '1.0.0-SNAPSHOT'
          from components.java
      }
  }
  // Used only publish to CodeArtifact
  repositories {
      maven {
          url System.env.CODEARTIFACT_REPO_URL
          credentials {
              username "aws"
              password System.env.CODEARTIFACT_AUTH_TOKEN
          }
      }
  }
}
```

## リソースの構築とJARの公開手順

いよいよ上記リポジトリを用いたリソースの構築に入ります。  
リポジトリの詳細については、[GitHub上のパブリックリポジトリ](https://github.com/yoyoyo-pg/gradle-publish-sample)として公開しています。

### CDKのスタックデプロイ

- CDKのスタックデプロイとして、`cdk deploy GradlePublishSampleStack`を実行します。

```powershell
PS C:\Users\git\gradle-publish-sample> cdk deploy GradlePublishSampleStack

✨  Synthesis time: 4.75s

（中略）

 ✅  GradlePublishSampleStack

✨  Deployment time: 67.98s

Stack ARN:
arn:aws:cloudformation:ap-northeast-1:XXXXXXXXXXX:stack/GradlePublishSampleStack/7f4f0d80-c9a8-11ee-bc49-0abe591c2227

✨  Total time: 72.73s

```

- 以下スタックが構築されます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/f8e7f343-68cd-8388-8f67-9f87b1040d1d.png)

- この時点で、上記で紹介した一通りのAWSリソースが構築されている事が分かります。

### GitHubへの接続設定の完了

- 接続設定はGitHub側での許可が必要なため、`cdk deploy`後に「接続中」の表示となります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/1cd2ef9b-2fcf-0e05-a65f-1b86e9be8e36.png)

- 「接続中」の接続を選択し、「保留中の接続を更新」をクリックします。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/9dd1e362-1b28-c997-cf8f-2bf9f32dae6e.png)

- 別ウィンドウが出てくるので、そちらで接続をします。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/11e9f7db-bf2c-deb3-6653-1e0697f2b740.png)

- 「新しいアプリをインストールする」を選択した場合は、以下の様なGitHub側の画面から、接続許可するリポジトリを選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/6a2db515-c808-d80e-a89d-cb72305af087.png)

- ここまで来たら、接続には成功です。

![codestar-connection.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/6f651c75-6799-88d0-3854-4de71f3474eb.png)

### JARの公開

- 接続設定が済んだ時点で、CodeArtifactへのJAR公開のパイプラインは完成です。
- 後は`main`ブランチに対してコミットをするか、手動でパイプラインを動かすと、自動的にCodeArtifactに対してJARが公開されるようになります。

![codepipeline.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/5acadb15-861b-0d78-0743-6f975ca812f2.png)

- パイプラインが起動するとBuildステージまで成功し、CodeArtifact上にも`JarSample`パッケージとしてJARが反映されている事が分かります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/92f91766-02ec-4cfe-be6d-7cc4e47f1a38.png)

## 参考文献

[CodeArtifactユーザーガイド - GradleでCodeArtifactを使用する](https://docs.aws.amazon.com/ja_jp/codeartifact/latest/ug/maven-gradle.html)
[Qiita - GradleでMavenローカルリポジトリにpublishをする](https://qiita.com/yoyoyo_pg/items/61ea8dc2e4e434f53f99)
