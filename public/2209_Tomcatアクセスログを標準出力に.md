---
title: ECS上のTomcatアクセスログをCloudWatch Logsに出力する
tags:
  - Java
  - Tomcat
  - CloudWatch
  - ECS
private: false
updated_at: '2024-03-16T08:45:41+09:00'
id: 725b153fa974206557c3
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

- ECS上で動作するコンテナについては標準出力にログが流れるよう設定する必要がありますが、Tomcatアクセスログはデフォルトで標準出力には流れません。
- 本記事ではTomcatのアクセスログを標準出力とし、CloudWatch Logsに出力する方法を紹介します。

## 環境

- Java11
- tomcat9.0

## 実施内容

方法としては以下です。

- アクセスログを日付毎に出力するのではなく、固定ファイル名とする
- 固定ファイルと標準出力のシンボリックリンクを作成する

::: note info
202402追記： serverl.xmlに directory="/dev" prefix="stdout"と直接標準出力を指定することが可能です。その場合は固定ファイルの作成とシンボリックリンク作成は不要です。
:::

## server.xmlの用意

`server.xml`のAccessLogValveを以下のような設定にする事で、アクセスログを固定ファイル名`access_log.txt`に出力するよう設定します。

```server.xml
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
prefix="access_log" suffix=".txt"
pattern="%h %l %u %t &quot;%r&quot; %s %b"
rotatable="false" fileDateFormat="" />
```

## Dockerfileの用意

`server.xml`の上書きを行い、その上で標準出力と`access_log.txt`のシンボリックリンクを作成します。

```:Dockerfile
FROM tomcat:9.0-jdk11-corretto
RUN yum update -y
COPY server.xml /usr/local/tomcat/conf/server.xml
RUN ln -sf /dev/stdout /usr/local/tomcat/logs/access_log.txt
COPY sample.war /usr/local/tomcat/webapps/sample.war
EXPOSE 8080
```

## Dockerfileをビルドし、ECRにプッシュ

```sh
# ECRの認証
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin アカウントID
.dkr.ecr.ap-northeast-1.amazonaws.com
# Dockerイメージのビルド
docker build -t tomcat-sample .
# イメージのタグ付け
docker tag tomcat-sample:latest アカウントID.dkr.ecr.ap-northeast-1.amazonaws.com/tomcat-sample:latest
# イメージのプッシュ
docker push アカウントID.dkr.ecr.ap-northeast-1.amazonaws.com/tomcat-sample:latest
```

## タスク定義、サービスの作成

ECRにプッシュしたイメージを参考に、tomcatコンテナを起動します。

### tomcatコンテナの設定内容

ECSのタスク定義内のログドライバーはデフォルトの`awslogs`を利用します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/417fd2f8-e501-d2dd-78ab-62b1f32092be.png)

8080番ポートをホストOSの8080番ポートにバインドします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/15e5b993-c1ec-7712-9d93-28804cb6e479.png)

※この際、タスク実行ロールの設定を忘れないようにしましょう

## CloudWatch Logsでアクセスログを確認

`ip:8080`の形でTomcatコンテナにブラウザからアクセスします。
次に、設定したロググループよりログを確認します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/61326375-c4a0-e01a-b7d4-d6cc2514bfe2.png)

無事、アクセスログがCloudWatch上で確認できる状態になりました！
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/6c82d025-8d91-d603-4566-c449a9c0df58.png)

## 参考文献

<https://aws.amazon.com/jp/premiumsupport/knowledge-center/ecs-container-logs-cloudwatch/>

<https://github.com/docker-library/tomcat>
