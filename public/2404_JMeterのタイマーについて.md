---
title: JMeterのタイマーの配置場所には気を付ける
tags:
  - JMeter
private: false
updated_at: '2024-04-13T20:48:31+09:00'
id: 7977d710a9f06efe8c35
organization_url_name: null
slide: false
ignorePublish: false
---
## この記事について

JMeterのタイマーの配置場所が原因で詰まった箇所があった為、その備忘録です。

## 起きた現象

- JMeterによる負荷試験をした際に、想定の半分～三割程度のリクエスト数しか出せない状態となってしまっていた
- ヒープメモリの指定、CPU使用率、ファイルディスクリプタ上限やプロセス数も確認したが、特に問題ない状態

## 原因

- JMeterのタイマーの追加場所が正しくなかった

> Note that timers are processed before each sampler in the scope in which they are found; if there are several timers in the same scope, all the timers will be processed before each sampler.
Timers are only processed in conjunction with a sampler. A timer which is not in the same scope as a sampler will not be processed at all.
To apply a timer to a single sampler, add the timer as a child element of the sampler. The timer will be applied before the sampler is executed. To apply a timer after a sampler, either add it to the next sampler, or add it as the child of a Flow Control Action Sampler.

複数のタイマーが同一スコープ内に存在する場合、全てのタイマーが各サンプラーの前に処理されるとの事でした。

参考：[jmeter.apache.org - 18.6 Timers](https://jmeter.apache.org/usermanual/component_reference.html#timers)

## 結果として

- 本来、サンプラーの子要素としてタイマーを追加すべき所を、サンプラーと並列で複数のタイマーを追加してしまっていた事が問題となっていた
- 結果として、各サンプラーでタイマーが実行される事となり、想定以下のリクエスト数となっていた

