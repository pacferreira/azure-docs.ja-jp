---
title: Azure Event Grid Service Bus のイベント スキーマ
description: Azure Event Grid で Service Bus イベント用に提供されているプロパティについて説明します
services: event-grid
author: banisadr
manager: darosa
ms.service: event-grid
ms.topic: reference
ms.date: 01/17/2019
ms.author: babanisa
ms.openlocfilehash: f44d2c1c5be6ac895b6f5ea9feca29c0f8ed09f3
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/13/2019
ms.locfileid: "60561763"
---
# <a name="azure-event-grid-event-schema-for-service-bus"></a>Service Bus 用の Azure Event Grid イベント スキーマ

この記事では、Service Bus イベントのプロパティとスキーマについて説明します。 イベント スキーマの概要については、「[Azure Event Grid イベント スキーマ](event-schema.md)」を参照してください。

サンプル スクリプトとチュートリアルの一覧については、[Service Bus のイベント ソース](event-sources.md#service-bus)に関する記事を参照してください。

## <a name="available-event-types"></a>使用可能なイベントの種類

Service Bus から出力されるイベントの種類は次のとおりです。

| イベントの種類 | 説明 |
| ---------- | ----------- |
| Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners | キューまたはサブスクリプションにアクティブなメッセージがあり、リッスンしているレシーバーがない場合に生成されます。 |
| Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener | 配信不能キューにアクティブなメッセージがあり、アクティブなレシーバーがない場合に生成されます。 |

## <a name="example-event"></a>イベントの例

次の例では、リスナー イベントがないアクティブなメッセージのスキーマを示します。

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourcegroups/{your-rg}/providers/Microsoft.ServiceBus/namespaces/{your-service-bus-namespace}",
  "subject": "topics/{your-service-bus-topic}/subscriptions/{your-service-bus-subscription}",
  "eventType": "Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://{your-service-bus-namespace}.servicebus.windows.net/{your-topic}/subscriptions/{your-service-bus-subscription}/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

配信不能キュー イベントのスキーマも同様です。

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourcegroups/{your-rg}/providers/Microsoft.ServiceBus/namespaces/{your-service-bus-namespace}",
  "subject": "topics/{your-service-bus-topic}/subscriptions/{your-service-bus-subscription}",
  "eventType": "Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://{your-service-bus-namespace}.servicebus.windows.net/{your-topic}/subscriptions/{your-service-bus-subscription}/$deadletterqueue/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

## <a name="event-properties"></a>イベントのプロパティ

イベントのトップレベルのデータを次に示します。

| プロパティ | Type | 説明 |
| -------- | ---- | ----------- |
| topic | string | イベント ソースの完全なリソース パス。 このフィールドは書き込み可能ではありません。 この値は Event Grid によって指定されます。 |
| subject | string | 発行元が定義したイベントの対象のパス。 |
| eventType | string | このイベント ソース用に登録されたイベントの種類のいずれか。 |
| eventTime | string | プロバイダーの UTC 時刻に基づくイベントの生成時刻。 |
| id | string | イベントの一意識別子。 |
| data | object | Blob Storage イベントのデータ。 |
| dataVersion | string | データ オブジェクトのスキーマ バージョン。 スキーマ バージョンは発行元によって定義されます。 |
| metadataVersion | string | イベント メタデータのスキーマ バージョン。 最上位プロパティのスキーマは Event Grid によって定義されます。 この値は Event Grid によって指定されます。 |

データ オブジェクトには、次のプロパティがあります。

| プロパティ | Type | 説明 |
| -------- | ---- | ----------- |
| namespaceName | string | リソースが存在する Service Bus 名前空間。 |
| requestUri | string | イベントを生成している特定のキューまたはサブスクリプションの URI。 |
| entityType | string | イベントを生成している Service Bus エンティティの種類 (キューまたはサブスクリプション)。 |
| queueName | string | キューをサブスクライブしている場合、アクティブなメッセージのあるキュー。 トピック/サブスクリプションを使っている場合は値 null。 |
| topicName | string | アクティブなメッセージのある Service Bus サブスクリプションが属しているトピック。 キューを使っている場合は値 null。 |
| subscriptionName | string | アクティブなメッセージのある Service Bus サブスクリプション。 キューを使っている場合は値 null。 |

## <a name="next-steps"></a>次の手順

* Azure Event Grid の概要については、[Event Grid の紹介](overview.md)に関する記事を参照してください。
* Azure Event Grid サブスクリプションの作成の詳細については、[Event Grid サブスクリプション スキーマ](subscription-creation-schema.md)に関する記事を参照してください。
* Service Bus での Azure Event Grid の使用に関して詳しくは、「[Azure Service Bus と Event Grid の統合の概要](../service-bus-messaging/service-bus-to-event-grid-integration-concept.md)」をご覧ください。
* [Functions または Logic Apps で Service Bus イベントを受信](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json)してみてください。
