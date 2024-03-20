---
title: Gradleの`-m`、`--dry-run`オプションでタスクの実行計画を確認する
tags:
  - gradle
private: false
updated_at: '2024-03-20T20:37:19+09:00'
id: df91f0f395e4599ded0c
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

- 既存のGradleのビルドパイプライン内で`publishAllPublicationsToMavenRepository`を実行する前に、新たに`test`タスクを追加したい要件がありました

## 結論

- `-m`または`--dry-run`オプションを付ける事で、実行されるタスクの一覧が確認可能です

> Run Gradle with all task actions disabled. Use this to show which task would have executed.

参考：[Command-Line Interface Reference - Execution options](https://docs.gradle.org/current/userguide/command_line_interface.html#sec:command_line_execution_options)

## 確認したこと

`publishAllPublicationsToMavenRepository`タスク単体での実行

```powershell
PS C:\Users\yoyoyo-pg\git\gradle-sample> .\gradlew publishAllPublicationsToMavenRepository --dry-run                                                        
:lib:compileJava SKIPPED
:lib:processResources SKIPPED
:lib:classes SKIPPED
:lib:jar SKIPPED
:lib:generateMetadataFileForMavenPublication SKIPPED
:lib:generatePomFileForMavenPublication SKIPPED
:lib:publishMavenPublicationToMavenRepository SKIPPED
:lib:publishAllPublicationsToMavenRepository SKIPPED

BUILD SUCCESSFUL in 617ms
```

`publishAllPublicationsToMavenRepository`の前に`test`タスクを実行

```powershell
PS C:\Users\yoyoyo-pg\git\gradle-sample> .\gradlew test publishAllPublicationsToMavenRepository --dry-run
:lib:compileJava SKIPPED
:lib:processResources SKIPPED
:lib:classes SKIPPED
:lib:jar SKIPPED
:lib:compileTestJava SKIPPED
:lib:processTestResources SKIPPED
:lib:testClasses SKIPPED
:lib:test SKIPPED
:lib:generateMetadataFileForMavenPublication SKIPPED
:lib:generatePomFileForMavenPublication SKIPPED
:lib:publishMavenPublicationToMavenRepository SKIPPED
:lib:publishAllPublicationsToMavenRepository SKIPPED
```

`diff`の内容

```diff
:lib:compileJava SKIPPED
:lib:processResources SKIPPED
:lib:classes SKIPPED
:lib:jar SKIPPED
+ :lib:compileTestJava SKIPPED
+ :lib:processTestResources SKIPPED
+ :lib:testClasses SKIPPED
+ :lib:test SKIPPED
:lib:generateMetadataFileForMavenPublication SKIPPED
:lib:generatePomFileForMavenPublication SKIPPED
:lib:publishMavenPublicationToMavenRepository SKIPPED
:lib:publishAllPublicationsToMavenRepository SKIPPED
```
