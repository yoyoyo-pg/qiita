---
title: AWS ToolKit For Eclipseを利用してDynamoDBのテーブルを作成する
tags:
  - AWS
  - Eclipse
private: false
updated_at: '2024-02-11T11:46:07+09:00'
id: 9c6b6de2cd86e61b7b99
organization_url_name: null
slide: false
ignorePublish: false
---

## もくじ

1. IAMユーザの作成+ロールの作成
2. EclipseマーケットプレイスでAWS ToolKit For Eclipseをインストール
3. プロジェクトを作成してjar実行
4. おわりに

## 1. IAMユーザの作成+ロールの作成

IAMユーザ、またはロールに`AmazonDynamoDBFullAccess`を付与します。

## 2. EclipseマーケットプレイスでAWS ToolKit For Eclipseをインストール

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/2eaf5824-72db-07c7-bd74-e34cbc0ce761.png)

こちらをインストールします。

ここでCredential情報を聴かれるので、IAMユーザーの「アクセスキーID」と「秘密アクセスキー」を入力します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/aebc5d03-133a-c108-1e4f-4719a0df4434.png)

## 3. プロジェクトを作成してJarの実行

新規AWS Javaプロジェクトより、DynamoDBSampleを選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/fb63706e-dfa2-2f7f-a4e8-9daa690ca285.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/5d68173d-1549-7351-8c15-fd2ffb4ddc7c.png)

すると以下のプロジェクトが自動生成されます。
この中の`AmazonDynamoDBSample.java`を実行します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/0af09cc4-b1bc-26b3-ac36-41db5fb1554c.png)

## おわりに

リージョンを明記していない場合、オレゴンのDynamoDBに`my-favorite-movies-table`テーブルが作成されます。
Javaの記述と作成されたテーブルを見比べながら、SDKの学習していくのに使えそうです。
