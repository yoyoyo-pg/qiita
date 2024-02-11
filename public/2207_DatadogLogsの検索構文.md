---
title: Datadog Logsのよく使うログ検索構文をまとめてみた
tags:
  - Datadog
private: false
updated_at: '2024-02-11T12:20:04+09:00'
id: 91dadbf250917238b1e7
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

Datadog Logsのログの検索方法についてまとめてみた記事です。
特定のログを抽出するうえで、よく使う検索条件をまとめました。

## Datadog Logsとは

その名の通り、ログの収集、処理、監視等が出来るDatadogの機能の一つです。
「Live Tail」を使えばリアルタイムにログ監視をすることが可能ですし、ログを基にメトリクスを作成したり、ログの各項目に対する分析を行うことが可能です。

<https://docs.datadoghq.com/ja/logs/>

## 検索構文の例

公式ドキュメント内、[ログ検索構文](https://docs.datadoghq.com/ja/logs/explorer/search_syntax/)を参考にしています。

### 仮定

スロークエリログを分析するというシーンを想定し、以下二つの属性を仮定しています。

- スロークエリが発生したSQL文：`sql_text`
- クエリの実行秒数：`query_time`

## 文字に対する検索

- 完全一致で特定のクエリを検索する場合

`@sql_text:"検索ワード"`

- 前方一致、後方一致で特定のクエリを検索する場合、ワイルドカード「*」を利用

`@sql_text:*検索ワード`
`@sql_text:検索ワード*`

- 特定の文字を除外する場合、「-」を付ける

`-@sql_text:*除外したいワード*`

## 数値に対する検索

- 5秒から10秒のクエリを検索する場合

`@query_time:[5 TO 10]`

- 10秒以上のクエリを検索する場合

`@query_time:>10`

## 積の確認

- 10秒以上かつ`UPDATE*`のクエリの場合

`@query_time:>10 AND @sql_text:UPDATE*`

## 和の確認

- `UPDATE`か`DELETE`のクエリの場合

`@sql_text:UPDATE* OR @sql_text:DELETE*`
