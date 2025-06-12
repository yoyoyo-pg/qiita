---
title: Data Lifecycle Manager が予定通りの時刻に実行されずに詰まった点
tags:
  - ebs
  - DLM
private: false
updated_at: '2025-06-12T23:10:58+09:00'
id: bf22dca18e71cef7f07e
organization_url_name: null
slide: false
ignorePublish: false
---
## 事象

- AWS CDK を用いて AWS の Data Lifecycle Manager のライフサイクルポリシーを設定していて、実行時に IAM Role 関連のエラーが発生していた
- IAM Role の調整をして、次回の実行を待っていても実行が行われなかった

## 原因

- ライフサイクルポリシーのステータスが「エラー」状態になっており、ポリシーのステータスを「有効」に選択し直す必要があった

マネジメントコンソール上でライフサイクルポリシーを変更しようとすると、以下の表示が出ていました。

> The policy is in an error state. Select enable or disable in policy status and then select modify to save the changes.

- 前提として、ポリシーの状態は「有効」「無効」「エラー」の3種類ある

## 補足

その他にも、実行タイミングについては以下の点に注意が必要

- ライフサイクルポリシーで指定出来るスケジュールは UTC 形式である点

> For Starting at, specify the time at which the policy runs are scheduled to start. The first policy run starts within an hour after the scheduled time. The time must be entered in the hh:mm UTC format.

https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-ami-policy.html?icmpid=docs_ec2_console#create-snap-policy

- スケジュールされた時刻について、即時実行ではない点（1時間以内）

> The first snapshot creation operation starts within one hour after the specified start time. Subsequent snapshot creation operations start within one hour of their scheduled time.


https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-ami-policy.html?icmpid=docs_ec2_console#snapshot-considerations

## 感想

AWS CDK で構築しており、マネジメントコンソール上での表示をあまり見ていなかったのが良くなかったな...という反省でした。
