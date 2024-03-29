---
title: VSCode拡張機能Community Server Connectorsのデプロイ時の挙動について調べてみた
tags:
  - Tomcat
  - VSCode
private: false
updated_at: '2024-03-16T08:45:40+09:00'
id: 2e8719e9a649aa93a617
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

前回の記事の続きです。

https://qiita.com/yoyoyo_pg/items/8d340eea5a71627c3c31

今回は、Community Server Connectorsの詳細のディレクトリ構成や、デプロイ時の操作による挙動を確認してみました。

## 環境

- VSCodeインストール済（Windows版）
- 拡張機能Community Server Connectors`v0.25.6`がインストール済
- 拡張機能Remote Server Protocol UI`v0.23.13`がインストール済
- 今回は検証としてTomcat8.5を立てます。

## インストール時

拡張機能Community Server Connectorsが起動した段階では、以下のディレクトリが存在しています。

- `C:\Users\ユーザー名\`配下に隠しフォルダ`.rsp`が生成
- `.rsp`フォルダ内には`redhat-community-server-connector`フォルダが生成
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/e7408a87-6824-e688-2375-17baf4df574a.png)

## サーバーの作成

次に、`Create New Server`より`Apache Tomcat 8.5.50`を選択しダウンロード

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/19553233-b271-62f0-5958-656a0692c46a.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/40ff94b2-e3b8-42fe-9451-e3fcd695fe99.png)

この段階では、ディレクトリ内が以下状態となっていました。

- `C:\Users\ユーザー名\.rsp\redhat-community-server-connector\runtimes\downloads`ディレクトリが生成
  - apache-tomcat-8.5.50.zipがダウンロード
- `C:\Users\ユーザー名\.rsp\redhat-community-server-connector\runtimes\installations`ディレクトリが生成
  - フォルダ`tomcat-8.5.50`が展開
- `C:\Users\akita\.rsp\redhat-community-server-connector\servers`ディレクトリが生成
  - apache-tomcat-8.5.50というファイルが生成

```json:apache-tomcat-8.5.50
{
  "args.override.boolean": "false",
  "args.program.override.string": "start",
  "args.vm.override.string": "-Djava.util.logging.config.file=/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dcatalina.base= -Dcatalina.home=C:\\Users\\ユーザー名\\.rsp\\redhat-community-server-connector\\runtimes\\installations\\tomcat-8.5.50\\apache-tomcat-8.5.50 -Djava.io.tmpdir=/temp",
  "id": "apache-tomcat-8.5.50",
  "id-set": "true",
  "org.jboss.tools.rsp.server.typeId": "org.jboss.ide.eclipse.as.server.tomcat.85",
  "server.base.dir": "C:\\Users\\ユーザー名\\.rsp\\redhat-community-server-connector\\runtimes\\installations\\tomcat-8.5.50\\apache-tomcat-8.5.50",
  "server.deploy.dir": "${server.base.dir}/webapps/",
  "server.home.dir": "C:\\Users\\ユーザー名\\.rsp\\redhat-community-server-connector\\runtimes\\installations\\tomcat-8.5.50\\apache-tomcat-8.5.50",
  "server.http.port": "8080"
}
```

GUIから`Edit Server`を選択すると、`C:\Users\ユーザ名\AppData\Local\Temp`内にある`tmpServerConnector-apache-tomcat-8.5.50~~~.json`といったファイルが開かれますが、上記ファイル`apache-tomcat-8.5.50`とリンクしているようです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/8efaf104-1cd5-4500-7652-47934d3ab8d0.png)

※こちらについては`C:\Users\ユーザー名\.rsp\redhat-community-server-connector\runtimes`配下の`URLTransportCache.cacheIndex.properties`内に記載がありました。

## Add Deployment（File）

ここからは、war展開時のディレクトリ構造や挙動を見てみます。

まずは`Add Deployment`を選択
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/29b240cd-88bf-9d58-8001-1fdd4b0df22f.png)
`File`か`Exploded`を選べますが、ここでは`File`を選択（sample.war）
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/210070e9-e722-007d-d56c-b3f0c4177496.png)
オプションのパラメータは`No`を選択
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/93ceccdb-4a94-e28c-cf2a-9f623967efb2.png)
先程の`apache-tomcat-8.5.50`ファイル内の`server.deploy.dir`として指定のある`webapps`配下に`sample.war`と`sample`が展開されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/9cd2b35b-cdd1-c967-e1b5-156f79dde53c.png)
`localhost:8080/sample/`でsampleを表示
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/2f209136-e319-7e48-4004-84cc8ec70ca2.png)

## Add Deployment（Exploded）

Add Deploymentとして、Explodedを選択した場合の挙動を確認します。
※先ほど展開された`sample`フォルダを使います。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/e918adc6-22b0-f53f-0f35-0b36562e95b5.png)
すると、sampleフォルダのみ`webapps`配下に展開されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/53f0f0d4-d161-0849-22fd-6652b8d38b8a.png)

ちなみに、**元のファイルを変更した際、自動的にwebapps内のファイルも更新されるか**が気になったので、元の指定ファイル`index.html`を編集してみました。

すると、`publish`が自動で走りファイル間の同期が行われます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/6aabda8d-9267-ff8f-d057-a0c90f4f1c95.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/5e8f7b20-b15c-f5de-a491-5a854df049f1.png)

`localhost:8080/sample/`で見てみると、`webapps`内にファイルの変更が反映されていることが分かります。

### それ以外に調べた事

- 同一バージョンの複数のtomcatを登録する事も可能のようです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/3b3fa135-3a51-114e-cde6-7743fc6a3175.png)

### まとめ

- 気軽にサーバーを構築・破棄出来る点は非常だと感じました。
- また、ファイル変更時に自動的に`publish`が走る点も便利そうです。
