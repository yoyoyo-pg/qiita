---
title: Datadog の Log Rehydration にかかる料金をシンプルにまとめる
tags:
  - Datadog
private: false
updated_at: '2025-06-12T23:55:01+09:00'
id: 2b9e6722d5ba0547c82c
organization_url_name: null
slide: false
ignorePublish: false
---
## 結論

「ログアーカイブに対するスキャンサイズ」と「復元したログの保持料金」にお金がかかります。

「スキャンサイズ」は圧縮されたログデータ1GBあたり`$0.10`かかり、「保持料金」は100万ログイベントあたり`$1.70`かかります。　※DatadogサイトがUSの場合かつ15日間の保存期間の場合

![{46EDDCDB-2223-4118-942B-FCBB27FDBF04}.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/22e3d5c2-b4ed-476a-93e5-6086e75ba6fa.png)

## 補足

なお、この「スキャンサイズ」はリハイドレート時の Archive Scan Size で確認可能です。

![aaa.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/581e4e05-c49b-40a1-b9a0-9e3a4fc46ac4.png)


[Datadog - 料金（Log Management）](https://www.datadoghq.com/ja/pricing/?product=log-management&site=us&tab=standard#products)  
[Datadog - アーカイブからのリハイドレート](https://docs.datadoghq.com/ja/logs/log_configuration/rehydrating/?utm_source=chatgpt.com&tab=amazons3)

## どうすれば節約できるのか

- たとえ「復元対象」が少なくとも、選択期間が同じ場合のスキャンサイズは変わりません。なので「出来るだけ少ない範囲を指定したり、少ない回数のリハイドレートにする」ことが必要です
- また、保持料金にもお金がかかるので、「絞り込みにより復元対象を減らす」ことも料金の削減につながります
