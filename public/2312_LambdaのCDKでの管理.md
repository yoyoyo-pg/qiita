---
title: Lambdaの管理をCDKに移行している話
tags:
  - lambda
  - CDK
private: false
updated_at: '2024-03-16T08:45:40+09:00'
id: aa7cdaa678d74c6842f9
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

今回の話は、筆者の私が SRE として Lambda、DynamoDB、API-Gateway 等を用いた、よくある構成の自社内のサーバレスなアプリケーションを、より開発・管理しやすい形への変更を目指している、という話になります。

## 対象となるアプリについて

- 「通知系の基盤」「ある一機能で用いられている処理基盤」を Lambda、DynamoDB、API-Gateway 等を用いて構築しているアプリが、自社内に複数個ある状態です。（事前検証やインフラの構築はSREチームが担当し、Lambda コードの開発は開発チームが担当）
- ただ、どのアプリケーションも、一度構築してからは予期しないエラーやランタイムのバージョンアップ対応を除いては、基本的に「作ってそのまま」な状態でした。

### 運用について

Lambda 関連の運用フローは元々以下の状態でした。

- Lambda の開発方法は、アプリによっては開発用 Lambda のコードをそのまま編集し、動作確認
- ステージング環境や本番環境への展開は、開発用 Lambda のコードをそのままコピペし張り付けるか、一度S3にアーカイブファイルを置いて、その後デプロイする形など、アプリケーションによって異なる状態
- ソースコードは一部アプリを除くと Lambda のコンソール上からしか見れない状態で、Git での管理が出来ていない状態

いくら普段触らない機能とはいえ「ソースコードの管理」があまり出来ておらず、かつ「デプロイされているバージョンやどの環境にどのバージョンのコードがデプロイされているかも分からない状況」からは最低限脱却していこうと思い、今回の改善に踏み切る事となりました。

## 今回の運用改善の目的

一旦は以下をゴールとして定めています。

- どの環境の Lambda に何のバージョンのコードがデプロイされているかが把握できる状態
- 開発チームにとって「コミットが苦とならず、あわよくばCDKが便利と思ってもらえるような」構成とする事
  - その為にも、CDK 化しても出来るだけ開発チームが見ないといけない部分は減らす

正直な所、Lambda 自体の改良や調整の頻度が高い訳では無いので「 Lambda のコードをリポジトリに置くようにし、バージョン管理の為のルールを制定する」という最低限の管理をする為の対応でも良かったのですが、

将来的には Lambda 等、AWS 上のプラットフォームを利用したアプリケーションの開発がより社内で進む可能性もあるのと、現在 CDK によるリソースの IaC 化を SRE チームとして進めている事もあり、ソースコード管理に加えて CDK 化も実施する事としました。

## CDKのスタック構成の紹介

ここからは、今回用意した CDK の詳細を記載します。

Lambda に限らずリソースを扱う為に`serveless`リポジトリとしています。

以下は CDK プロジェクトの全体像です。（`cdk.json`、`package.json`等のファイルは割愛）typescript で`CDKv2`を利用しています。

```powershell
serverless
├─ bin
│　└─ serverless.ts
├─ lambda
│  ├─ archive
│  │  ├─ appA
│  │  │　　└─ appA-1.0.0.jar
│  │  └─ appB
│  │   　　└─ appB-1.0.0.zip
│  ├─ code
│  │   ├─ appA
│  │   └─ appB
│  └─ release-status.ts
├─lib
│  ├─ appA.ts
│  ├─ appB.ts
│  └─ stack-lambda-function.ts
├─ parameter
│  └─ parameter.ts
```

- `bin`、`lib`はお馴染みの通り
- `parameter.ts`に環境別パラメータを記載

という形ですが、今回の工夫としては

- `lambda`ディレクトリは開発チーム
- それ以外のディレクトリは SRE チーム

 が触るという想定で、ソースコードのあるディレクトリと Lambda の周辺コードの存在するディレクトリを分割する事で、お互いがソースを触りやすいようにしています。

## Lambdaディレクトリの説明

`lambda`ディレクトリについては、先に概要を紹介します。

- `lambda/code`配下に各Lambdaのソースコードがあります。
- `lambda/archive`配下にLambda用のアーカイブ（`zip`、`jar`）があります。
- `lambda/release-status.ts`にLambdaのソースコードのバージョンが記載されています。

### release-status

こちらのファイルに、各環境の各機能のリリース状況を記載する形としています。  
今回の仕組み上どのように使われるかについては後述します。

```lambda/release-status.ts
/* 各環境のCDK管理でのLambdaリリース状況 */

// dev
export const devLambdaSource = {
  appA: 'appA-1.0.0.jar',
  appB: 'appB-1.0.0.zip',
}

// test
export const testLambdaSource = {
  appA: 'appA-1.0.0.jar',
  appB: 'appB-1.0.0.zip',
}

// prod
export const prodLambdaSource = {
  appA: 'appA-1.0.0.jar',
  appB: 'appB-1.0.0.zip',
}
```

## bin、lib配下のディレクトリの説明

### CDKアプリケーションの宣言

`bin/serverless.ts`に環境別のスタックを記載しています。  
今回は開発・ステージング・本番環境を想定したスタック構成としています。

```bin/serverless.ts
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import { LambdaFunctionStack } from '../lib/stack-lambda-function';
import { devParam, stgParam, prodParam } from '../parameter/parameter';

/** CDKアプリケーションの宣言 */
const app = new cdk.App();

// dev
new LambdaFunctionStack(app, 'DevLambdaFunction', devParam);
// stg
new LambdaFunctionStack(app, 'StgLambdaFunction', stgParam);
// prod
new LambdaFunctionStack(app, 'ProdLambdaFunction', prodParam);
```

### スタック定義

`lib/stack-lambda-function.ts`にスタック定義を記載しています。  
`parameter.ts`から値を取得し、カスタム Construct の`appA`、`appB`を宣言しています。

```lib/stack-lambda-function.ts
import { Stack } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import { appA } from './appA';
import { appB } from './appB';
import { LambdaStackProps } from '../parameter/parameter';

/** 
 * AWSマネージドを用いた機能を定義するスタックです。
 **/
export class LambdaFunctionStack extends Stack {
  constructor(scope: Construct, id: string, props: LambdaStackProps) {
    super(scope, id, props);
    // appA
    new appA(this, 'appA', props);
    // appB
    new appB(this, 'appB', props);
  }
}
```

### パラメータ

`parameter.ts`に、各種リソースを定義しています。  

ここで登場するのが`release-status`という概念です。

こちらは「主に開発チームが触る事を想定した」`lambda`ディレクトリ配下の`release-status.ts`のパラメータを各環境別に`lambdaSource: string`として渡す形としています。

```parameter/parameter.ts
import { devLambdaSource, stgLambdaSource, prodLambdaSource } from "../lambda/release-status"

// インターフェース
export interface LambdaStackProps {
  env: { account: string,  region: string  },
  envName: string, // 環境名
  lambdaSource: string, // Lambda用ソース指定
}

// dev
export const devParam: LambdaStackProps = {
  env: { account: 'XXXXXXXXXXXX', region: 'ap-northeast-1' },
  envName: 'dev'
  lambdaSource: devLambdaSource,
}

// stg
export const stgParam: LambdaStackProps = {
  env: { account: 'XXXXXXXXXXXX', region: 'ap-northeast-1' },
  envName: 'stg',
  lambdaSource: stgLambdaSource,
}
// prod
export const prodParam: LambdaStackProps = {
  env: { account: 'XXXXXXXXXXXX', region: 'ap-northeast-1' },
  envName: 'prod',
  lambdaSource: prodLambdaSource,
}
```

### カスタムConstruct

`lib/appX-construct.ts`という形で各種 Construct を宣言しています。

アプリケーション別のリソース・権限付与をひとまとめにして宣言しています。

ここでの注目ポイントは先程の`release-status.ts`を参照し渡って来た`lambdaSource`です。  
例えば`appB-1.0.0.zip`という形で、アーカイブファイル名の指定がされる形です。

```lib/appB-construct.ts
import * as codebuild from 'aws-cdk-lib/aws-codebuild';
import { Construct } from "constructs";
import { Function, Code, Runtime } from 'aws-cdk-lib/aws-lambda';

/** 入力インターフェース */
export interface constructProps {
  envName: string
  lambdaSource: { appB: string }
}

export class appB extends Construct {
  constructor(scope: Construct, id: string, props: constructProps) {
    super(scope, id);
    /* API-Gatewayの定義 */
    /* DynamoDBの定義 */

    /* Lambdaの定義 */
    const fn = new Function(this, 'Function', {
      runtime: Runtime.PYTHON_3_11,
      handler: 'index.handler'
      code: Code.fromAsset('\lambda/archive/appB/' + props.lambdaSource.appB);
    });
    
    /* 権限付与など */
    
  }
}
```

## 開発時の大まかな流れ

ここまでソースを読んで頂くと大体お気づき頂けるかと思いますが、今回の想定では

- 開発時に、`lambda/code/appX`配下のアプリケーションコードを触って頂く
- 各環境への展開時には、`lambda/archive/appX`配下に、アプリ`appX`のアーカイブファイル（`zip`、`jar`）のバージョンを採番した上で置いてもらう
- 改良後、`archive`配下に配置したファイルを`release-status.ts`に記載する
- 後は`cdk diff`を取り、問題なければ`cdk deploy`する

といった開発フローを想定しています。

## 補足

- 「そもそも`release-status`って必要ですか？開発→ステージング→本番環境の順に、単にデプロイすれば良くないですか？」という声もあると思います。
  - 実は今回CDK化する前に手動、もしくはCloudFormationで構築されたアプリケーション群が大半を占めており、例えばアプリによっては「開発環境のCDK化は済んだが、ステージング・本番環境のCDK化は済んでいない」等、アプリ × 環境 によって状況が様々となっています。
  - 早めに「全環境 × 全アプリ」のデプロイ状況、IaC管理の状況の足並みを揃えたい所ですが、他タスクとの兼ね合いもあり、時間があれば実施、となっているのが現状です。
  - その為、`release-status`で一覧化しておく事で、ついでに現在の IaC による管理状況の把握（見える化）事態も目的として、リポジトリ化したという事情もあります。

## まとめ

開発フローの改善により、

- 開発チームで`cdk deploy`まで実施できそうであれば依頼
- 難しそうであれば`lambda`ディレクトリのアーカイブ作成まで依頼し、デプロイ作業は SRE チームで実施する

等、ディレクトリ上のお作法に従えば、リリース作業と状態把握まで出来る状態となりました。  

ただし、複数の言語や複数の管理状態のLambdaが存在するので「統一」ルールを作成して運用の認知負荷を下げる事を優先した結果、「アーカイブ作成」「release-statusに記載」等、管理の為の手作業が発生してしまっている部分もあります。

今後は CDK 化を進める共に、この辺りの手間をより改善する事が出来ればと考えていますし、将来的にはパイプラインの導入や、開発チームにも CDK を通して AWS 上で気軽に検証環境等の構築を行って頂ける状態を目指せると良いな、と考えています。
