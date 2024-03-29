---
title: VSCode拡張機能Community Server Connectorsを試して、sample.warをデプロイしてみた
tags:
  - Java
  - Tomcat
  - VSCode
private: false
updated_at: '2024-03-16T08:45:40+09:00'
id: 8d340eea5a71627c3c31
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

- VSCodeでTomcatにデプロイするためのサンプルのJavaアプリ開発でもしてみようと思い、Tomcat for Javaを見てみたらなんと、非推奨となっておりました。。

![2022-06-27-23-14-16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/80ca559f-af34-f6cf-11a0-65b2841cd889.png)

代わりに推奨されている拡張機能である[Community Server Connectors](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-community-server-connector&ssr=false#overview)を利用し、tomcatのサーバを立ててsample.warをデプロイしたので、その紹介記事となります。

---

### Community Server Connectorsとは

[VSCodeのマーケットプレイス](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-community-server-connector&ssr=false#overview)には以下の記載がありました。

>このVSCode拡張機能は、リモートサーバープロトコルベースのサーバーコネクタを提供します。このコネクタは、コミュニティランタイムと、Apache Felix、Karaf、Tomcatなどのサーバーを開始、停止、公開、または制御できます。

RedHatが提供している拡張機能となっております。

こちらの拡張機能をインストールすると、Remote Server Protocol UIが同時にインストールされます。

![2022-06-27-23-41-50.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/79c79ee1-5a01-3dba-67fa-0387b49a592d.png)
![2022-06-27-23-43-37.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/1289607a-10d5-f642-c78b-e49417d26e30.png)
拡張機能をインストールすると、エクスプローラにSERVERSとしてCommunity Server Connectorが表示されます。
![2022-06-27-23-55-28.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/f13f33a3-78f8-54b5-fb77-a41c062ec945.png)

---

### セットアップの概要

:::note info
2022.09.08追記：最新バージョンのCommunity Server Connectorsのバージョン`v0.25.6`、Remote Server Protocol UIのバージョン`v0.23.13`の場合、セットアップ操作が不要の可能性があります。
:::

拡張機能インストール後のセットアップ方法については[公式リポジトリ](https://github.com/redhat-developer/rsp-server-community)に記載があるので、それ通りに進めてみることにしました。

実行するコマンドは以下です。

```.bash
git clone https://github.com/redhat-developer/rsp-server-community
cd rsp-server-community/rsp
mvn clean install
```

筆者の場合は手元のPCにJavaやMavenを入れておりませんでしたので

- Mavenのインストール + 環境変数Pathの登録
- JDKのインストール + 環境構築Path,JAVA_HOMEの登録

を済ませてから`mvn clean install`を実行しました。

※JDKは特にこだわりがなかったので[Java18](https://www.oracle.com/java/technologies/downloads/#java18)をインストールしました。

---

### セットアップ詳細

`mvn clean install`を実行
![2022-06-27-22-55-52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/5ad402e1-8ba7-34c1-5b32-adbe2f5d9dd9.png)

ビルド完了時
![2022-06-27-23-01-23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/7aa50a03-015e-9bd0-78bb-913a167f85e0.png)

ビルドが完了すると、Community Server Connectorが起動できるようになります。
![2022-06-27-23-01-53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/4f9464b5-d2b8-4cec-3d2a-8b8622de05b1.png)

---

### Tomcat起動

次に、tomcatを起動させます。
Create New Serverを選択
![2022-06-27-23-02-34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/a669b631-1bce-c161-e53b-519222b3de0e.png)
確認ダイアログで「Yes」を選択
![2022-06-27-23-02-49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/e4914081-952d-469c-b3e8-2132789d941b.png)
起動させるサーバを選択します。
![2022-06-27-23-03-15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/fcf144a1-7316-2e21-e12b-efc3b0bb46b0.png)

起動したいサーバを選択すると、ライセンスへの同意を求められるので、先に進みます。
![2022-06-27-23-07-46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/b68499af-7a0e-64e6-4da0-be0428b825b0.png)
![2022-06-27-23-07-55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/c1bb3223-2071-c0a8-6a91-a9028462cc35.png)
選択後、Community Server Connectorの下に先程追加したサーバが表示されます。
![2022-06-28-00-00-34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/43397ed5-9703-4860-82e7-b30c8ebb37a1.png)

Start Serverをクリックすると...
![2022-06-27-23-11-33.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/acb6aa5f-526e-cb86-1f4a-62859f8a87ce.png)

おなじみの起動ログが表示されます。
![2022-06-27-23-11-56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/7cfd2faa-e793-84dd-f7a6-92ac05e471d6.png)

`localhost:8080`で接続すると、ウェルカムページが表示されます！
![2022-06-27-23-12-22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/c61c59cd-c9a2-e76c-ca0b-862c86e52fc1.png)

---

### warのデプロイ

`Add Deployment`を選択して、サンプルのwarをデプロイします。
サンプルのwarは[tomcat.apache.org](https://tomcat.apache.org/tomcat-9.0-doc/appdev/sample/)からダウンロードしました。
![2022-06-27-23-23-34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/c381f7ce-d7f9-9536-36f5-7b3b0e06e8a1.png)

起動時のパラメータは特に指定せずに進みます
![2022-06-27-23-24-08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/548cd771-f51c-69b7-bb50-8b3a9e6484dd.png)

デプロイ完了のログ
![2022-06-27-23-27-21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/ec7d17ff-9f6e-8b79-7ce2-9d384409eaf7.png)

`localhost:8080/sample/`で動作確認完了！
![2022-06-27-23-26-22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/40f62a50-8fa4-af9b-9329-39ea2b08d33d.png)
