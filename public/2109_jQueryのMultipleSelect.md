---
title: jQueryプラグイン【Multiple Select】を使って複数選択可能なselectタグを生成してみよう
tags:
  - JavaScript
  - jQuery
  - multiple-select
private: false
updated_at: '2024-03-16T08:45:40+09:00'
id: 4859dfbbc512ef653828
organization_url_name: null
slide: false
ignorePublish: false
---
- 最近、jQueryプラグインである[Multiple Select](https://multiple-select.wenzhixin.net.cn/)を用いてselectタグをカスタマイズする機会がありました。
- Multiple Selectとは、チェックボックスで複数選択が可能なプルダウンを簡単に作れるjQueryプラグインです。

## 作成例

以下の様なプルダウンを作成可能です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/7b128be9-8b01-673d-5353-b8ffea5fa25a.png)　![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/d017bcfb-e405-da95-d9d1-6204020ffacc.png)　![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/3393ad52-1ce3-10e0-7584-5d9889714ebc.png)

## コード

公式のサンプルを少し弄ったものになります。
最低限利用するために必要なのは以下3点です。

1. 各種JS,CSSのインポート
2. selectタグ内に`multiple="multiple"`の指定
3. 初期化処理の`$('select').multipleSelect()`

```index.html
<!doctype html>
<html lang="jp">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <script src="https://cdn.jsdelivr.net/npm/jquery/dist/jquery.min.js"></script>
    <!-- Latest compiled and minified JavaScript -->
    <script src="https://unpkg.com/multiple-select@1.5.2/dist/multiple-select.min.js"></script>
    <!-- Latest compiled and minified CSS -->
    <link rel="stylesheet" href="https://unpkg.com/multiple-select@1.5.2/dist/multiple-select.min.css">
    <title>Multiple Select Sample</title>
  </head>
  <body>
    <!-- Multiple Select -->
    <select multiple="multiple">
      <option value="1">1月</option>
      <option value="2">2月</option>
      <option value="3">3月</option>
      <option value="4">4月</option>
      <option value="5">5月</option>
      <option value="6">6月</option>
      <option value="7">7月</option>
      <option value="8">8月</option>
      <option value="9">9月</option>
      <option value="10">10月</option>
      <option value="11">11月</option>
      <option value="12">12月</option>
    </select>
    <script>
    $(function () {
        $('select').multipleSelect({
            width: 200,
            formatSelectAll: function() {
                return 'すべて';
            },
            formatAllSelected: function() {
                return '全て選択されています';
            }
        });
    });
    </script>
  </body>
</html>
```

## 解説

### 1.各種JS,CSSのインポート

お好きな方法で構いませんが、今回はCDNを利用しています。

### 2.selectタグ内にmultiple="multiple"の指定

選択肢の複数選択を可能にするために、multiple属性を指定します。

```html
<select multiple="multiple">
</select>
```

### 3.初期化処理の$('select').multipleSelect()

`$('select').multipleSelect()`内に、オプションを指定する形となっています。
今回は以下の3つのオプションを指定してあります。

```js
   $('select').multipleSelect({
        width: 200,
        formatSelectAll: function() {
            return 'すべて';
        },
        formatAllSelected: function() {
            return '全て選択されています';
        }
   });
```

#### width: 200

Multipe Selectのオプションの一つです。
生成時の`$('select').multipleSelect()`内に記載する事で、
生成された選択エリアに`style="width: 200px"`の指定をしてくれます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/5a8601f5-8f30-a203-3bb0-128eff019672.png)

#### formatSelectAll／formatAllSelected

公式ドキュメントにはローカライズ用APIとして紹介されています。

- `formatSelectAll`
  - 「すべて選択」オプションの為のチェックボックスのメッセージを調整できます。
  - 初期値は[Select all]です。
- `formatAllSelected`
  - 「すべて選択」時の表示メッセージを調整できます。
  - 初期値は[All selected]です。

#### 使い方

`return`の値を任意の値に変更する事で、表示させたい値を変更できます。
今回は以下の値にすることで、ローカライゼーションを行っております。

```js
        formatSelectAll: function() {
            return 'すべて';
        },
        formatAllSelected: function() {
            return '全て選択されています';
        }
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/057bc560-bbd3-e7b9-a46d-6addcd103566.png)　　![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/d3184547-475e-3da1-c277-6a61739e76f4.png)

### その他オプション等

イベントリスナをセットするオプションも豊富にあります。
セレクトボックスの開閉や、全選択、全選択を解除した際など、幅広い場面で利用できます。

```js
    onOpen: function() {
        alert("セレクトボックスを開いた際のalert");
    },
    onCheckAll: function() {
        alert("全選択した際のalert");
    },
```

## おわりに

リッチなUIを簡単に実現できるので、非常に便利です。

## 参考文献

[Multiple Select 公式サイト](https://multiple-select.wenzhixin.net.cn/)
[Github - wenzhixin / multiple-select](https://github.com/wenzhixin/multiple-select)
