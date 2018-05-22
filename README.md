
# kinesis-commerce-system-sample-documentation

Kinesis 受発注・在庫システム サンプル 構成情報などのドキュメント

## アプリ

マルチプロジェクト構成にはしていない。

| 名前                                              | パッケージ                                                 | 説明                                                     |
|:--------------------------------------------------|:-----------------------------------------------------------|:---------------------------------------------------------|
| kinesis-commerce-system-sample-common             | com.example.kinesiscommercesystemsample.common             | Kinesis 受発注・在庫システム サンプル 共通ライブラリ     |
| kinesis-commerce-system-sample-order-api          | com.example.kinesiscommercesystemsample.order.api          | Kinesis 受発注・在庫システム サンプル 受注API            |
| kinesis-commerce-system-sample-order-consumer     | com.example.kinesiscommercesystemsample.order.consumer     | Kinesis 受発注・在庫システム サンプル 受注コンシューマ   |
| kinesis-commerce-system-sample-purchase-api       | com.example.kinesiscommercesystemsample.purchase.api       | Kinesis 受発注・在庫システム サンプル 仕入れAPI          |
| kinesis-commerce-system-sample-purchase-consumer  | com.example.kinesiscommercesystemsample.purchase.consumer  | Kinesis 受発注・在庫システム サンプル 仕入れコンシューマ |
| kinesis-commerce-system-sample-inventory-api      | com.example.kinesiscommercesystemsample.inventory.api      | Kinesis 受発注・在庫システム サンプル 在庫API            |
| kinesis-commerce-system-sample-inventory-consumer | com.example.kinesiscommercesystemsample.inventory.consumer | Kinesis 受発注・在庫システム サンプル 在庫コンシューマ   |

## コンシューマで受け付けるメッセージ

| コンシューマ                                      | メッセージ                    | 説明                                                      |
|:--------------------------------------------------|:------------------------------|:----------------------------------------------------------|
| kinesis-commerce-system-sample-order-consumer     | OrderRegistMessage            | 受注登録                                                  |
| kinesis-commerce-system-sample-order-consumer     | OrderChangeStatusMessage      | 受注ステータス変更                                        |
| kinesis-commerce-system-sample-purchase-consumer  | PurchaseRegistMessage         | 仕入れ登録                                                |
| kinesis-commerce-system-sample-purchase-consumer  | PurchaseStatusChangeMessage   | 仕入れステータス変更                                      |
| kinesis-commerce-system-sample-inventory-consumer | InventoryInboundMessage       | 入庫 (仕入れに対して在庫を充当する)                       |
| kinesis-commerce-system-sample-inventory-consumer | InventoryOutboundMessage      | 出庫 (受注に対して在庫を引き当てる)                       |


## API呼び出し

* 受注登録

リクエスト
```bash
curl -XPOST -H 'Content-Type:application/json' localhost:8080/order/regist -d '{"item_id": "item_1", "quantity": 1}'
```
レスポンス
```json
{
    "request_id": "5ce5a130-57ff-11e8-adea-acde48001122",
    "data": [
        {
            "message_id": "5ce68b90-57ff-11e8-adea-acde48001122"
        }
    ],
    "message": "Successful completion."
}
```

---

* 仕入れ登録

リクエスト
```bash
curl -XPOST -H 'Content-Type:application/json' localhost:8081/purchase/regist -d '{"item_id": "item_1", "quantity": 10}'
```
レスポンス
```json
{
    "request_id": "5ce5a130-57ff-11e8-adea-acde48001122",
    "data": [
        {
            "message_id": "5ce68b90-57ff-11e8-adea-acde48001122"
        }
    ],
    "message": "Successful completion."
}
```

---

* 入庫 (どの仕入れに紐付いて入庫したのか、リクエストに含める)

リクエスト
```bash
curl -XPUT -H 'Content-Type:application/json' localhost:8082/inventory/inbound -d '{"item_id": "item_1", "quantity": 10, "purchase_id": "5ce68b91-57ff-11e8-adea-acde48001122"}'
```
レスポンス
```json
{
    "request_id": "5ce5a130-57ff-11e8-adea-acde48001122",
    "data": [
        {
            "message_id": "5ce68b90-57ff-11e8-adea-acde48001122"
        }
    ],
    "message": "Successful completion."
}
```

---

* 出庫 (どの受注に紐付いて出庫したのか、リクエストに含める)

リクエスト
```bash
curl -XPUT -H 'Content-Type:application/json' localhost:8082/inventory/outbound -d '{"item_id": "item_1", "quantity": 10, "order_id": "5ce68b91-57ff-11e8-adea-acde48001122"}'
```
レスポンス
```json
{
    "request_id": "5ce5a130-57ff-11e8-adea-acde48001122",
    "data": [
        {
            "message_id": "5ce68b90-57ff-11e8-adea-acde48001122"
        }
    ],
    "message": "Successful completion."
}
```


---


## AWS

* Kinesis Data Streams

1シャードにつき1コンシューマの構成。

| 名前                                              | シャード数 | サーバー側の暗号化 | データ保持期間 |
|:--                                                |:--         |:--                 |:--             |
| kinesis-commerce-system-sample-order-stream       | 1          | 無効               | 24時間         |
| kinesis-commerce-system-sample-purchase-stream    | 1          | 無効               | 24時間         |
| kinesis-commerce-system-sample-inventory-stream   | 1          | 無効               | 24時間         |

* DynamoDB

役割については後述。

| 名前                                                  |
|:--                                                    |
| kinesis-commerce-system-sample-order-consumer-v1      |
| kinesis-commerce-system-sample-purchase-consumer-v1   |
| kinesis-commerce-system-sample-inventory-consumer-v1  |


## ローカルで動かす

```bash
 export AWS_ACCESS_KEY_ID=
 export AWS_SECRET_ACCESS_KEY=
 export AWS_REGION=ap-northeast-1
./gradlew bootRun

```



## Kinesisについて

### KCL設定

### DynamoDBテーブル

KCLにより自動生成されるテーブル。
KCLで設定したアプリケーション名単位でのチェックポイント管理に利用される。
テーブル名はKCLで設定したアプリケーション名と一致する。(無関係なテーブルが上書きされないように注意が必要)

* kinesis-commerce-system-sample-order-consumer-v1
    * Write Capacity : 10
    * Read Capacity : 10


| フィールド名                  | プライマリパーティションキー  | 概要                      |
|:--                            |:--                            |:--                        |
| checkpoint                    | -                             |                           |
| checkpointSubSequenceNumber   | -                             |                           |
| leaseCounter                  | -                             |                           |
| leaseKey                      | ◯                            | シャードIDが入る。        |
| leaseOwner                    | -                             |                           |
| ownerSwitchesSinceCheckpoint  | -                             | -                         |


### レイテンシ簡易計測

1. KCLチューニング前 (Producer-Consumer 同一AZ)
    * 実行ログ
        * (1) 2018-03-07T06:19:51.659Z プロデューサ側のレコード送信 直前
        * (2) 2018-03-07T06:19:51.675Z プロデューサ側のレコード送信 直後
        * (3) 2018-03-07T06:19:52.578Z コンシューマ側のレコード受信 直後
    * レイテンシ
        * 919ms
1. KCLチューニング後 (ポーリング間隔 250ms) (Producer-Consumer 同一AZ)
    * 実行ログ
        * (1) 2018-03-07T07:59:09.310Z プロデューサ側のレコード送信 直前
        * (2) 2018-03-07T07:59:09.325Z プロデューサ側のレコード送信 直後
        * (3) 2018-03-07T07:59:09.544Z コンシューマ側のレコード受信 直後
    * レイテンシ
        * 234ms
1. KCLチューニング後 (ポーリング間隔 250ms) (Producer-Consumer 別のAZ)
    * 実行ログ
        * (1) 2018-03-07T09:34:44.536Z プロデューサ側のレコード送信 直前
        * (2) 2018-03-07T09:34:44.550Z プロデューサ側のレコード送信 直後
        * (3) 2018-03-07T09:34:44.620Z コンシューマ側のレコード受信 直後
    * レイテンシ
        * 84ms


イベントドリブン・アーキテクチャにするとKinesisのGetRecords呼び出し回数の制限に引っかかる可能性がある。（コンシューマの種類数の増加に伴い）

