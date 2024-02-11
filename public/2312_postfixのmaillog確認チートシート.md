---
title: Postfixのmaillogからメール配信数を集計するチートシート
tags:
  - mail
  - postfix
private: false
updated_at: '2024-02-11T16:42:19+09:00'
id: 1c08eb4045408bf8f5b4
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

Postfixの`maillog`の配信数を集計する為のチートシートです。

## 集計対象を取得

まずは`xxxx@xxx.xxxxx.com`からのメールを絞り込み

```bash
cat maillog | grep "from=<xxxx@xxx.xxxxx.com>" > filter-maillog
```

`filter-maillog`の一例としては以下

```bash:filter-maillog
Nov 26 13:01:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 26 13:02:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 26 13:02:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 26 13:03:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 26 13:03:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 26 13:03:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 26 13:04:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 26 13:04:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 26 13:04:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 26 13:04:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 27 13:00:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 28 13:00:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 30 13:00:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 30 14:00:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 30 15:00:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 30 16:00:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
Nov 30 17:00:00 XXXXX postfix/qmgr[XXXX]: XXXXXXXXXXXX: from=<xxxx@xxx.xxxxxx.com>, size=xxxx, nrcpt=1 (queue active)
```

## 配信数の集計

### 月別

```bash
$ awk '{print $1}' filter-maillog | uniq -c
     17 Nov
```

### 日別

```bash
$ awk '{print $1, $2}' filter-maillog | uniq -c
     10 Nov 26
      1 Nov 27
      1 Nov 28
      5 Nov 30
```

### 時間別

```bash
$ awk '{print $1, $2, substr($3, 1, 2)}' filter-maillog | uniq -c
     10 Nov 26 13
      1 Nov 27 13
      1 Nov 28 13
      1 Nov 30 13
      1 Nov 30 14
      1 Nov 30 15
      1 Nov 30 16
      1 Nov 30 17
```

### 10分毎

```bash
$ awk '{print $1, $2, substr($3, 1, 4)}' filter-maillog | uniq -c
     10 Nov 26 13:0
      1 Nov 27 13:0
      1 Nov 28 13:0
      1 Nov 30 13:0
      1 Nov 30 14:0
      1 Nov 30 15:0
      1 Nov 30 16:0
      1 Nov 30 17:0
      1 Oct 30 17:0
```

### 分別

```bash
$ awk '{print $1, $2, substr($3, 1, 5)}' filter-maillog | uniq -c
      1 Nov 26 13:01
      2 Nov 26 13:02
      3 Nov 26 13:03
      4 Nov 26 13:04
      1 Nov 27 13:00
      1 Nov 28 13:00
      1 Nov 30 13:00
      1 Nov 30 14:00
      1 Nov 30 15:00
      1 Nov 30 16:00
      1 Nov 30 17:00
```

## 参考文献

<http://unyouchan.blog.jp/archives/1492098.html>
