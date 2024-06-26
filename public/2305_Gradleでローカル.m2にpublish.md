---
title: GradleでMavenローカルリポジトリにpublishをする
tags:
  - Maven
  - gradle
private: false
updated_at: '2024-03-30T20:00:03+09:00'
id: 61ea8dc2e4e434f53f99
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

普段Mavenを使ってJavaのプロジェクトをビルドしていますが、新規ライブラリを作成するにあたり

- Gradleを使いビルド
- 既存のMavenプロジェクトからGradleでビルドしたJarを参照

を試してみました。

初めてGradleを触ったので、Mavenしか触ったことない方が新たにGradleを利用する際の参考になれば幸いです。

## 動作環境

Windows 11
Amazon Corretto 20.0.1
Gradle 8.1
Apache Maven 3.9.1

## Gradleプロジェクトの作成

`gradle init`でプロジェクトを作成します。[Gradle公式](https://docs.gradle.org/current/samples/sample_building_java_libraries.html)のJavaライブラリの構築例を参考にしています。  

※今回は`gradle-sample`ディレクトリにプロジェクトを作成しました。

```powershell
PS C:\Users\yoyoyo-pg\workspace\gradle-sample> gradle init

Welcome to Gradle 8.1!

Here are the highlights of this release:
 - Stable configuration cache
 - Experimental Kotlin DSL assignment syntax
 - Building with Java 20

For more details see https://docs.gradle.org/8.1/release-notes.html

Starting a Gradle Daemon (subsequent builds will be faster)
```

projectのtypeを聞かれるので、`3.library`を選択します。  

```powershell
Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4] 3
```

languageは`3.Java`とします。

```powershell
Select implementation language:
  1: C++
  2: Groovy
  3: Java
  4: Kotlin
  5: Scala
  6: Swift
Enter selection (default: Java) [1..6] 3
```

DSLは`1.Grooby`とします。

```powershell
Select build script DSL:
  1: Groovy                                                                        
  2: Kotlin
Enter selection (default: Groovy) [1..2] 1
```

その他の設定をします。

```powershell
Select test framework:
  1: JUnit 4                                                                                        
  2: TestNG
  3: Spock
  4: JUnit Jupiter
Enter selection (default: JUnit Jupiter) [1..4] 1

Project name (default: gradle-sample):
Source package (default: gradle.sample):               
Enter target version of Java (min. 7) (default: 20): 
Generate build using new APIs and behavior (some features may change in the next minor release)? (default: no) [yes, no]
```

`gradle init`が完了！

```powershell
> Task :init
Get more help with your project: https://docs.gradle.org/8.1/samples/sample_building_java_libraries.html

BUILD SUCCESSFUL in 38s
2 actionable tasks: 2 executed
```

## build.gradle

最終的な`build.gradle`は以下です。

```build.gradle
plugins {
    id 'java-library'
    id 'maven-publish'
}

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'junit:junit:4.13.2'
    api 'org.apache.commons:commons-math3:3.6.1'
    implementation 'com.google.guava:guava:31.1-jre'
}

publishing {
    publications {
        maven(MavenPublication) {
            groupId = 'com.sample.yoyoyo-pg'
            artifactId = 'jarSample'
            version = '1.0.0'
            from components.java
        }
    }
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(20)
    }
}

```

## build.gradleの調整内容

### プラグインの調整

`maven-publish`を新たに追加します。

こちらのプラグインを追加する事で、ビルドアーティファクトをMavenリポジトリに公開する事が可能となります。  

```build.gradle
plugins {
    id 'java-library'
    id 'maven-publish'
}
```

:::note warn
こちらのプラグインの追加が無い状態だと、`publish`時に以下エラーで失敗します。
:::

```powershell
PS C:\Users\yoyoyo-pg\workspace\gradle-sample> .\gradlew publishToMavenLocal

FAILURE: Build failed with an exception.

* Where:
Build file 'C:\Users\yoyoyo-pg\workspace\gradle-sample\lib\build.gradle' line: 30

* What went wrong:
A problem occurred evaluating project ':lib'.
> Could not find method publishing() for arguments [build_45h53m0hs22jrtku865mdmsu1$_run_closure3@3934a26a] on project ':lib' of type org.gradle.api.Project.
```

:::note warn
また、以前は`maven`プラグインの`uploadArchives`というタスクを利用して`publish`が出来たようですが、現在のGradle 8.X系では廃止されているので注意が必要です。
:::

[Upgrading your build from Gradle 5.x to 6.0](https://docs.gradle.org/current/userguide/upgrading_version_5.html)

> Legacy publication system is deprecated and replaced with the *-publish plugins
The uploadArchives task and the maven plugin are deprecated.

[Upgrading your build from Gradle 6.x to 7.0](https://docs.gradle.org/current/userguide/upgrading_version_6.html)

> Removal of the uploadArchives task
The uploadArchives task was used in combination with the legacy Ivy or Maven publishing mechanisms. It has been removed in Gradle 7. You should migrate to the maven-publish or ivy-publish plugin instead.

### build.gradleのpublish内容の調整

以下内容を`build.gradle`に追加します。

ここでの`groupId`、`artifactId`、`version`はJarを参照するプロジェクトの`pom.xml`の指定と対応しています。

```build.gradle
publishing {
    publications {
        maven(MavenPublication) {
            groupId = 'com.sample.yoyoyo-pg'
            artifactId = 'jarSample'
            version = '1.0.0'
            from components.java
        }
    }
}
```

## Mavenのローカルリポジトリにpublish

`gradle-sample`ディレクトリ上で `.\gradlew publishToMavenLocal`を実行します。

[Publishing to Maven Local](https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven:install)

```powershell
PS C:\Users\yoyoyo-pg\workspace\gradle-sample> .\gradlew publishToMavenLocal                                                            
Path for java installation 'C:\Program Files\Amazon Corretto\jdk20.0.0_33' (Windows Registry) does not contain a java executable

BUILD SUCCESSFUL in 765ms
5 actionable tasks: 3 executed, 2 up-to-date
```

この実行により、私の環境の場合は`C:\Users\yoyoyo-pg\.m2\repository\com\sample\yoyoyo-pg\jarSample\1.0.0`配下に`jarSample-1.0.0.jar`が作成されます。

## Maven側のプロジェクトからJarの参照

既存Mavenプロジェクトの`pom.xml`に以下を追加します。

```pom.xml
<dependency>
    <groupId>com.sample.yoyoyo-pg</groupId>
    <artifactId>jarSample</artifactId>
    <version>1.0.0</version>
</dependency>
```

これにて、MavenプロジェクトからGradleのプロジェクトの参照が可能となります。

## おわりに

はじめてのGradleでしたが、Mavenを多少触っているお陰でそこまで手間取ることなくライブラリの準備を行う事が出来ました。

今回の構築内容のGitHubリポジトリは[こちら](https://github.com/yoyoyo-pg/gradle-publish-local-sample)です。

:::note info
2024年3月追記
ローカルMavenリポジトリへの`publish`を継続して行いたい場合は`-t`オプションの活用が便利ですので、[こちらの記事](https://qiita.com/yoyoyo_pg/items/d9ec078365167b89c6c1)にまとめています。
:::

## 参考資料

[Gradle - Releases](https://gradle.org/releases/)
[Gradle - Building Java Libraries Sample](https://docs.gradle.org/current/samples/sample_building_java_libraries.html)
[Gradle - Using Gradle Plugins](https://docs.gradle.org/current/userguide/plugins.html)
[Gradle - The Java Library Plugin](https://docs.gradle.org/current/userguide/java_library_plugin.html#java_library_plugin)
[Gradle - Maven Publish Plugin]( https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven)
