---
title: GradleのContainuous Buildsを使い、タスクの自動実行を実現する
tags:
  - gradle
private: false
updated_at: '2024-03-27T22:01:49+09:00'
id: d9ec078365167b89c6c1
organization_url_name: null
slide: false
ignorePublish: false
---
## Containuous Buildを使うと、タスクの自動的な再起動が可能

普段実行しているGradleコマンドに`-t`または`--continuous`を付ける事で、ファイルが変更された時のタスクの再実行が可能です。

## ローカルMavenリポジトリへの反映

最近は以下の記事のようなGradleを用いたJARライブラリのビルド環境を整えているのですが、

- [Qiita - GradleでMavenローカルリポジトリにpublishをする](https://qiita.com/yoyoyo_pg/items/61ea8dc2e4e434f53f99)
- [Qiita - CodeArtifactでのJAR公開パイプラインをAWSで実現する](https://qiita.com/yoyoyo_pg/items/1647d65f5b4ae4ae4270)

手元の環境でJARライブラリを開発する時に、`.m2`への反映の自動実行として`Containuous Build`が上手く利用できそうでした。

## 試してみる

- 試しに`publishToMavenLocal`を実行します
  - こちらは[Maven Publish Plugin](https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven)のローカルMavenリポジトリへの反映のコマンドです
- `maven-publish`プラグインを利用し、以下の`publish`の設定を記載します
  - 詳細は[こちら](https://qiita.com/yoyoyo_pg/items/61ea8dc2e4e434f53f99)の`publishToMavenLocal`の紹介記事の設定を参照ください

```build.gradle
plugins {
    id 'java-library'
    id 'maven-publish'
}

...

publishing {
    publications {
        maven(MavenPublication) {
            groupId = 'com.sample.yoyoyo-pg'
            artifactId = 'jarSample'
            version = '1.0.0-SNAPSHOT'
            from components.java
        }
    }
}
```

- `./gradlew publishToMavenLocal -t`を実行
- 最初の実行後、`IDLE`状態となります

```powershell
PS C:\Users\yoyoyo-pg\git\gradle-sample> ./gradlew publishToMavenLocal -t
Starting a Gradle Daemon, 9 incompatible and 4 stopped Daemons could not be reused, use --status for details

BUILD SUCCESSFUL in 4s
5 actionable tasks: 3 executed, 2 up-to-date

Waiting for changes to input files... (ctrl-d then enter to exit)
<-------------> 0% WAITING
> IDLE

```

- 最初の`publish`が成功している事が分かります

![first-publish.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/866e2c74-f219-018e-6127-b0ecfee3c5e3.png)

- 適当な`src/main`配下のJavaファイルを変更すると、変更を検知して再度タスクが走ります

```powershell
modified: C:\Users\yoyoyo-pg\git\gradle-sample\lib\src\main\java\gradle\sample\Library.java
Change detected, executing build...


BUILD SUCCESSFUL in 650ms
5 actionable tasks: 5 executed

Waiting for changes to input files... (ctrl-d then enter to exit)
<=============> 100% EXECUTING [10m 13s]
> IDLE

```

- ファイルが更新されている事が分かります

![second-publish.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/765420b1-b012-f9e5-0219-9529821dbfd4.png)
