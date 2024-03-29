---
title: GitHub CodespacesでAWS CDK + ecspressoのコンテナ構築用環境を整備
tags:
  - CDK
  - ecspresso
  - devcontainer
  - Codespaces
private: false
updated_at: '2024-03-16T08:45:40+09:00'
id: 0daeca3a799d3f7e133e
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

以前書いたこちらAWS CDKとecspressoを利用してコンテナを簡単に構築する為の記事ですが、

https://qiita.com/yoyoyo_pg/items/5921801e7f674f4e1023

AWS CDKを触った事が無い方や、ecspressoを触った事が無い方にピンと来にくいかもしれないと思い、ハンズオン形式で`cdk deploy`や`ecspresso deploy`が実行できるように、リポジトリを公開しました。

https://github.com/yoyoyo-pg/cdk-ecspresso

単に環境構築手順を`README.md`に記載しても良かったのですが、以前から気になっていた**GitHub Codespaces**を利用し、ある程度環境構築を自動化しましたので、その紹介記事です。

## 用語の再整理

### ecspresso

**ECSサービス、タスクに関わる最小限のリソースをコード管理**する事ができるツールです。

https://github.com/kayac/ecspresso

https://zenn.dev/fujiwara/books/ecspresso-handbook-v2

### GitHub Codespaces

>codespace は、クラウドでホストされている開発環境です。 構成ファイルをリポジトリにコミットすることで、GitHub Codespaces のプロジェクトをカスタマイズできます (コードとしての構成とよく呼ばれます)。これにより、プロジェクトのすべてのユーザーに対して繰り返し可能な codespace 構成が作成されます。

https://docs.github.com/ja/codespaces/overview

以前はベータ版でしたが、現在は一般ユーザーも無料で使えるようになった為、今回初めて利用しました。

開発環境としてコンテナが利用される為、今回は`devcontainer.json`と`Dockerfile`を定義しています。

## 構築自動化の為に用意した内容

プロジェクトの`ルートディレクトリ/.devcontainer/`配下に以下2ファイルを用意しました。

```powershell
cdk-ecspresso-project/
├── .devcontainer
│   ├── Dockerfile
│   └── devcontainer.json
```

```Dockerfile:Dockerfile
ARG VARIANT="18"
FROM mcr.microsoft.com/vscode/devcontainers/typescript-node:0-${VARIANT}
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
  && unzip awscliv2.zip \
  && ./aws/install \
  && rm -rf ./aws awscliv2.zip \
  && npm install -g typescript \
  && npm install -g aws-cdk \
  && npm install -g npm-check-updates
RUN curl -sL -O https://github.com/kayac/ecspresso/releases/download/v2.0.0/ecspresso_2.0.0_linux_amd64.tar.gz \
  && tar -zxvf ecspresso_2.0.0_linux_amd64.tar.gz \
  && sudo install ecspresso /usr/local/bin/ecspresso \
  && rm -f LICENSE README.md ecspresso ecspresso_2.0.0_linux_amd64.tar.gz
```

```json:devcontainer.json
{
  "name": "Node.js & TypeScript",
  "build": {
    "dockerfile": "Dockerfile"
  },
    "postCreateCommand": "npm install"
}
```

## コンテナ構築用環境の構築

**Code**タブからCodespacesを立ち上げるだけで、`.devcontainer`配下の内容を基に構築が始まります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/ca04d821-2e31-dce7-851f-c7360cc1759f.png)

- Codespacesが立ち上がってきた状態で既に`cdk`コマンドも`ecspresso`コマンドもインストールされています。  
- 最後に、`aws configure`だけ設定すれば直ぐにお手持ちのAWSアカウントにインフラリソースを構築できる状態となります。
- Codespaces内からAWSアカウントに対しての詳細のコンテナデプロイ手順はリポジトリ内の`README.md`に記載してあります。

<https://github.com/yoyoyo-pg/cdk-ecspresso/tree/master/.devcontainer>

## おわりに

- GitHub CodespacesはローカルPC上のVSCodeと遜色なく使えて、今回初めて使いましたが非常に便利だと感じました。
- 環境構築の備忘録として残しておく事も出来る為、devcontainer自体も今後活用していきたいです。
- 上記環境であれば気軽に`cdk deploy`と`ecspresso deploy`が試せるので、便利さを実感頂けると嬉しいなと思います。

## 参考文献

https://docs.github.com/ja/codespaces

https://github.com/kayac/ecspresso

https://zenn.dev/fujiwara/books/ecspresso-handbook-v2
