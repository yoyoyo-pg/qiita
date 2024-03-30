---
title: CodePipelineからGitHubリポジトリを参照する際に「リポジトリが見つからない」場合のトラブルシューティング
tags:
  - CodePipeline
  - CodeStar
private: false
updated_at: '2024-03-30T19:59:38+09:00'
id: 91ae1c67d43fb3313e0d
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

CodePipelineからCodeStarを利用しGitHub接続をする検証中に、ハマった点について紹介します。

## 前提

- CodePipelineでは、CodeStarSourceConnection（デベロッパー用ツール > 設定 > 接続）を用いて、各種プロバイダーと接続が可能です。
- 接続設定がされていると、CodePipeline上の「リポジトリ名」「ブランチ名」の指定を元に、ソースコード取得が可能です。
- 今回の記事では、CodeStarSourceConnectionの接続設定自体についての解説は行いません。

参考：[CodeStarSourceConnection for Bitbucket Cloud, GitHub, GitHub Enterprise Server, GitLab.com, and GitLab self-managed actions](https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/action-reference-CodestarConnectionSource.html)

## 発生した事象

- GitHub上にリポジトリやブランチが実在するにも関わらず、CodePipeline上で`FullRepositoryName`が見つけられないと表示される

> [GitHub] No Branch [main] found for FullRepositoryName [yoyoyo-pg/gradle-publish-sample]

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/e097e735-a328-f052-5d34-58f97e1e2c33.png)

## 原因

- 以前、GitHub側にインストールしていたAWS Connectorを使いまわしており、今回のパイプラインの対象リポジトリに対するアクセス権限が存在しませんでした。
- こちらの設定に関しては、`GitHubのユーザーページ` > `settings` > `Integrations` > `Applications` から確認可能です。

![github-auth.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/0a5155bd-b434-839d-0909-413e4f744bae.png)

## 対処法

- 対象のリポジトリを新たに加え、`save`します。

![select-repo.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/9759f1e6-f031-5ae0-8f8b-f41d143a8b45.png)

- パイプラインを再実行し、無事GitHubからのソース取得に成功しました！

![cicd.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/af373688-b309-0c73-45b1-fc73dfe9b237.png)

## ついでに検証

- [CodeStarSourceConnectionについてのドキュメント](https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/action-reference-CodestarConnectionSource.html)に記載がありますが、リポジトリ名に関しては大文字小文字を正しく設定する必要があるようです。
- 今回だと`yoyoyo-pg/gradle-publish-sample`という指定ですが、あえて`YOYOYO-PG/Gradle-Publish-Sample`という指定で記載し、パイプラインを動かしてみました。
- ただし、こちらに関してはエラーとなる事無くソース取得に成功しました。
