---
title: AWSマネージドを利用し、はじめてのWEB API開発をしてみた話
tags:
  - AWS
  - WebAPI
  - DynamoDB
  - lambda
  - APIGateway
private: false
updated_at: '2024-02-11T12:07:36+09:00'
id: 39c6100646b4526be60d
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

最近WEB APIやRESTについて学習中なので、AWSのマネージドを利用してWEB APIの開発をしてみました。  

構成の概要や、開発中に参考になった記事、便利だなと思ったツールやデバックについて紹介したいと思います。

AWSマネージドでのWEB API開発に興味のある方に読んでもらえると嬉しいな、と思っています。

---

### 採用技術

- API Gateway
- Lambda
- DynamoDB

「まずは動くAPIを！」という事で、サーバーサイドの実装のみ行っています。
LambdaはPythonで実装しています。

---

### その他

- VSCode拡張機能
  - REST Client（テスト用）
  - Swagger Viewer（ドキュメント生成用）
- CloudWatch（デバック用）

「VSCodeの拡張機能を色々お試ししたい！」という目的もあったので、テストやドキュメント生成用としてVSCodeの拡張機能を利用しています。

---

### 構成

今回はシンプルなCRUD機能の実装という事で、辞書アプリケーションのAPIの形式をとりました。

- GETメソッド：辞書情報の一覧取得、個別取得
- PUTメソッド：辞書情報の登録、更新
- DELETEメソッド：辞書情報の削除

といった形式で、APIを実装しています。

---

#### DynamoDB

dictionaryテーブルを用意しました。
パーティションキーはid（数値型）としています。

データ構造は以下を定義しました

- id：パーティションキー
- word：辞書として登録する用語
- explanation：用語の説明

![2022-06-25-22-54-48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/4a0a18c6-bc59-f5b3-c284-26137d160a5a.png)

---

#### API Gateway

glossary-sampleという名前のAPIをタイプ「HTTP API」で用意しました。

![2022-06-25-22-51-56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/444a83ab-de2a-4cc4-f342-4d6b085aca71.png)

以下の操作を想定し、設計しました。

|  メソッド  |  パス  |  説明  |
| ---- | ---- | ---- |
|  PUT  |  /items  |  追加、更新  |
|  GET  |  /items |  一覧取得  |
|  GET  |  /items/{id} |  一件取得  |
|  DELETE  |  /items/{id} |  一件削除  |

![2022-06-25-22-45-11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/e635bc9a-27d1-e350-808a-59879ffb7ae4.png)

Lambdaとの統合に関しては、全て同一Lambdaを呼び出すようにしています。

---

#### Lambda

実装自体に時間をかけたくなかった事もあり、Lambdaを選択しました。
詳しくは省略しますが、Lambdaの実行ロール（IAM）にDynamoDBへのアクセス権を付与しています。

```python:lambda_function.py
import json
import boto3
from boto3.dynamodb.conditions import Key, Attr


def lambda_handler(event, context):
    
    # DynamoDBのテーブルアクセスクラスを生成    
    dynamoDB = boto3.resource('dynamodb')
    table= dynamoDB.Table('dictionary')
    
    # return用メッセージ
    dict = {"returnMsg": "null"}
    
    routeKey = event['routeKey']
    
    # 複数件取得
    if routeKey == 'GET /items':
        response = table.scan()
        return response['Items']
    
    # 1件取得    
    elif routeKey == 'GET /items/{id}':
        id = event['pathParameters']['id']
        response = table.get_item(
            Key={
                'id': int(id)
            }
        )
        return response['Item']
    
    # 1件追加 or 更新
    elif routeKey == 'PUT /items':
        
        data = json.loads(event.get("body"))
        print(data)
        
        table.put_item(
            Item = {
                "id": int(data["id"]),
                "word": data["word"],
                "explanation": data["explanation"]
            }
        )
        dict["returnMsg"] = "PUT complete!"
        return json.dumps(dict)
    
    # 1件削除    
    elif routeKey == 'DELETE /items/{id}':
        id = event['pathParameters']['id']
        table.delete_item(
            Key={
                'id': int(id)
                
            }
        )
        dict["returnMsg"] = "DELETE complete!"
        return json.dumps(dict)
    
    # それ以外
    else:
        dict["returnMsg"] = "not match"
        return json.dumps(dict)
```

HTTPメソッド毎に条件分岐をすることで、一つのLambdaで全ての処理を行えるよう実装しました。
DynamoDBへの検索パターン（scan,getItem,query）の使い分けについては、[こちら](https://qiita.com/UpAllNight/items/a15367ca883ad4588c05)の記事を参考にしました。

---

##### REST Client

APIの動作確認として、VSCodeの拡張機能「REST Client」を使って、curlを投げる方法を取りました。
使い方については[こちら](https://qiita.com/toshi0607/items/c4440d3fbfa72eac840c)の記事を参考にしました。

画面左側がテスト用の記述(curl.http)で、右側が一番上の「一覧取得」のリクエストを送信した際の結果です。

テスト用の記述を書くと、自動的に「Send Request」というボタンが表示されます。「Send Request」を押すだけでリクエストを飛ばすことができるので大変便利です。

![2022-06-25-23-21-20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/325131a6-b723-f7fb-2f04-ac6b9f773fcc.png)

##### CloudWatch

開発時のデバッグは、主にCloudWatchに吐かれているLambdaの実行ログを基に行いました。
普段Pythonを書かないので慣れていない部分もありますが、エラー文を都度確認して解消を繰り返し、実装に漕ぎ着けました。

---

#### Swagger Viewer

こちらもVSCodeの拡張機能です。
API設計書としてSwaggerを使ってみたい、と思っていたので利用してみました。

API Gatewayには「OpenAPI」仕様でapiの定義をエクスポートでき、またはインポートすることで、同じAPIを素早く作成できるそうです。

なので今回は、YAML形式でAPI定義をエクスポートしています。

![2022-06-25-23-35-51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/19642341-b535-1555-cd09-26fdc3e9a448.png)


swagger-viewerをVSCodeにインストールした状態で「エクスポートしたファイル」を開き、VSCode上で「Preview Swagger」コマンドを実行することで、APIの定義を閲覧することが可能です。

![2022-06-25-23-38-19.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/f2b4888b-ed9f-5998-52db-3e5090ced3fa.png)

マネジメントコンソール上だとAPIの概要が一目で分かりにくいので、こういった形でドキュンメント化しておくと管理がしやすそうだと思います。

---

### まとめ

個別の要素についての深堀りはしませんでしたが、AWS上でAPIを実装する際の大枠を掴むことができました。
「一旦バックエンドだけ実装しよう！」と割り切れたおかげで、API実装の部分をピンポイントに学習できたのは良かったのかな、と思います。
