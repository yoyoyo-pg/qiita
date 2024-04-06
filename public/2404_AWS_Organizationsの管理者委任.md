---
title: AWS Organizationsの委任管理者の登録で詰まった箇所
tags:
  - AWS
  - Organizations
private: false
updated_at: '2024-04-06T17:37:44+09:00'
id: 1b052664f9d9fb9498ff
organization_url_name: null
slide: false
ignorePublish: false
---
## この記事について

- AWS Control Tower の検証中に、管理者の委任作業で詰まった点があったので、その紹介記事です

## 詰まった箇所

- 具体的には、Control Tower の Account Factory Customization (AFC) の検証中に、Service Catalog の管理者を委任する場面です

### Service Catalog 管理者委任のコマンド実行

- [DevelopersIOのAFCの記事](https://dev.classmethod.jp/articles/control-tower-account-factory-customization/)を参考に以下コマンドを CloudShell で実行しました
- AWS公式ドキュメントの[register-delegated-administrator](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/organizations/register-delegated-administrator.html)に詳細の記載があります

```bash
aws organizations register-delegated-administrator \
    --account-id 委任先アカウントID \
    --service-principal servicecatalog.amazonaws.com
```

:::note info
Organizations の管理アカウントからメンバーアカウントに、ポリシー管理の委任が可能です
:::

- 実行した所、以下のエラーが発生しました

```bash
[cloudshell-user@ip-10-130-48-71 ~]$ aws organizations register-delegated-administrator --account-id XXXXXXXXXXX --service-principal servicecatalog.amazonaws.com

An error occurred (ConstraintViolationException) when calling the RegisterDelegatedAdministrator operation: You must enable service access before you delegate an administrator for this service. Call the AWS API EnableAWSServiceAccess first.
```

- エラー内にも記載がありますが、先に「サービスへのアクセスを有効化する」必要がありました

## コマンド再実行

- `register-delegated-administrator`の前に[enable-aws-service-access](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/organizations/enable-aws-service-access.html)を実行します

```bash
[cloudshell-user@ip-10-134-22-228 ~]$ aws organizations enable-aws-service-access --service-principal servicecatalog.amazonaws.com
[cloudshell-user@ip-10-134-22-228 ~]$ aws organizations register-delegated-administrator --account-id XXXXXXXXXXX --service-principal servicecatalog.amazonaws.com
```

### 結果確認

- [list-delegated-administrators](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/organizations/list-delegated-administrators.html)で詳細を確認します
- コマンド実行前は以下の形で表示

```bash
[cloudshell-user@ip-10-134-22-228 ~]$ aws organizations list-delegated-administrators
{
    "DelegatedAdministrators": [
    ]
}
```

- コマンド実行後は以下の形で表示

```bash
[cloudshell-user@ip-10-134-22-228 ~]$ aws organizations list-delegated-administrators
{
    "DelegatedAdministrators": [
        {
            "Id": "XXXXXXXXXXX",
            "Arn": "arn:aws:organizations::XXXXXXXXXXX:account/o-r1dheg1ess/XXXXXXXXXXX",
            "Email": "sample@example.com",
            "Name": "ServiceCatalogAdmin",
            "Status": "ACTIVE",
            "JoinedMethod": "CREATED",
            "JoinedTimestamp": "2024-04-04T13:57:48.830000+00:00",
            "DelegationEnabledDate": "2024-04-06T03:01:39.347000+00:00"
        }
    ]
}
```

### AWSマネジメントコンソール上からの確認

- Organizations 管理者アカウントの`AWS Organizations` > `サービス`からも、Service Catalog へのアクセスが有効になっている事が確認できます

![enable-service.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/a25d080c-0a2f-3a7a-3a4d-06beea3da57f.png)

### 補足

Organizations のサービスアクセスの許可や管理者委任のコマンドについては、[こちらの記事](https://dev.classmethod.jp/articles/manage-aws-organizations-service-integration-and-delegation-from-cli/)が参考になりました

## おわりに

- サービスアクセス許可や管理者委任の概念の理解が足りておらず、設定に手間取ってしまいました
- Organizations の知識を前提として、AWS Control Tower を利用する必要があると改めて感じました
