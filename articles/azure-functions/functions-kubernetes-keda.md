---
title: KEDA を使用した Kubernetes での Azure Functions
description: クラウドまたはオンプレミスで KEDA (Kubernetes ベースのイベント ドリブン自動スケーリング) を使用して Kubernetes で Azure Functions を実行する方法について説明します。
author: jeffhollan
ms.topic: conceptual
ms.date: 11/18/2019
ms.author: jehollan
ms.openlocfilehash: 9978bd567b1b07e8dd0e22e1f02834626281a5dd
ms.sourcegitcommit: f34165bdfd27982bdae836d79b7290831a518f12
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/13/2020
ms.locfileid: "75920677"
---
# <a name="azure-functions-on-kubernetes-with-keda"></a>KEDA を使用した Kubernetes での Azure Functions

Azure Functions ランタイムにより、必要な場所と方法でのホスティングにおける柔軟性が提供されます。  [KEDA](https://keda.sh) (Kubernetes ベースのイベント ドリブン自動スケーリング) は、Azure Functions ランタイムおよびツールにシームレスに組み合わされ、Kubernetes でのイベント ドリブンな自動スケーリングを提供します。

## <a name="how-kubernetes-based-functions-work"></a>Kubernetes ベースの関数の動作

Azure Functions サービスは 2 つの主要コンポーネントで構成されています。ランタイムとスケール コントローラーです。  Functions ランタイムでは、ご自分のコードを実行します。  ランタイムには、関数の実行をトリガー、ログ、および管理する方法のロジックが含まれています。  Azure Functions ランタイムは、*どこでも*実行できます。  もう 1 つのコンポーネントは、スケール コントローラーです。  スケール コントローラーによって、関数をターゲットにしているイベントの割合が監視され、アプリを実行しているインスタンスの数がプロアクティブにスケーリングされます。  詳細については、「[Azure Functions のスケールとホスティング](functions-scale.md)」を参照してください。

Kubernetes ベースの Functions では、KEDA によるイベント ドリブン スケーリングを使用して、[Docker コンテナー](functions-create-function-linux-custom-image.md)内に Functions ランタイムが提供されます。  KEDA では、0 インスタンスまでのスケールダウン (インスタンスが発生していないとき) と *n* インスタンスまでのスケールアップが可能です。 これは、Kubernetes 自動スケーラー (ポッドの水平自動スケーラー) 用のカスタム メトリックを公開することによって行われます。  KEDA で Functions のコンテナーを使用すると、任意の Kubernetes クラスターにおいてサーバーレス関数の機能をレプリケートできるようになります。  これらの関数は、サーバーレス インフラストラクチャ用の [Azure Kubernetes Services (AKS) 仮想ノード](../aks/virtual-nodes-cli.md)機能を使用してデプロイすることもできます。

## <a name="managing-keda-and-functions-in-kubernetes"></a>Kubernetes での KEDA と関数の管理

Kubernetes クラスター上で Functions を実行するには、KEDA コンポーネントをインストールする必要があります。 このコンポーネントは、[Azure Functions Core Tools](functions-run-local.md) を使用してインストールできます。

### <a name="installing-with-the-azure-functions-core-tools"></a>Azure Functions Core Tools によるインストール

既定では、Core Tools によって KEDA と Osiris の両方のコンポーネントがインストールされます。これらは、それぞれイベント ドリブンと HTTP スケーリングをサポートするコンポーネントです。  インストールでは、現在のコンテキストで実行されている `kubectl` が使用されます。

次のインストール コマンドを実行して、ご自分のクラスターに KEDA をインストールします。

```cli
func kubernetes install --namespace keda
```

## <a name="deploying-a-function-app-to-kubernetes"></a>Kubernetes への関数アプリのデプロイ

KEDA を実行する Kubernetes クラスターには、あらゆる関数アプリをデプロイできます。  関数は Docker コンテナー内で実行されるため、プロジェクトには `Dockerfile` が必要です。  まだそれがない場合は、Functions プロジェクトのルートで次のコマンドを実行して、Dockerfile を追加できます。

```cli
func init --docker-only
```

イメージをビルドして関数を Kubernetes にデプロイするには、次のコマンドを実行します。

> [!NOTE]
> Core Tools では、イメージのビルドと発行に Docker CLI が活用されます。 あらかじめ Docker がインストールされ、ご利用のアカウントに `docker login` で接続されていることを確認してください。

```cli
func kubernetes deploy --name <name-of-function-deployment> --registry <container-registry-username>
```

> `<name-of-function-deployment>` をお使いの関数アプリの名前に置き換えます。

これにより、Kubernetes `Deployment` リソース、`ScaledObject` リソース、`local.settings.json` からインポートされる環境変数を含む `Secrets` が作成されます。

### <a name="deploying-a-function-app-from-a-private-registry"></a>プライベート レジストリから関数アプリをデプロイする

前述のフローは、プライベート レジストリにも使用できます。  コンテナー イメージをプライベート レジストリからプルする場合は、`func kubernetes deploy` を実行する際に `--pull-secret` フラグを指定して、プライベート レジストリの資格情報を保持する Kubernetes シークレットを参照します。

## <a name="removing-a-function-app-from-kubernetes"></a>Kubernetes からの関数アプリの削除

デプロイ後は、作成された関連する `Deployment`、`ScaledObject`、`Secrets` を削除することによって、関数を削除できます。

```cli
kubectl delete deploy <name-of-function-deployment>
kubectl delete ScaledObject <name-of-function-deployment>
kubectl delete secret <name-of-function-deployment>
```

## <a name="uninstalling-keda-from-kubernetes"></a>Kubernetes からの KEDA のアンインストール

次のコア ツール コマンドを実行して KEDA を Kubernetes クラスターから削除できます。

```cli
func kubernetes remove --namespace keda
```

## <a name="supported-triggers-in-keda"></a>KEDA でサポートされているトリガー

KEDA は、次の Azure Function トリガーをサポートしています。

* [Azure Storage キュー](functions-bindings-storage-queue.md)
* [Azure Service Bus キュー](functions-bindings-service-bus.md)
* [Azure Event/IoT Hubs](functions-bindings-event-hubs.md)
* [Apache Kafka](https://github.com/azure/azure-functions-kafka-extension)
* [RabbitMQ キュー](https://github.com/azure/azure-functions-rabbitmq-extension)

### <a name="http-trigger-support"></a>HTTP トリガーのサポート

HTTP トリガーを公開する Azure Functions は使用することはできますが、KEDA では直接管理されません。  Azure Functions Core Tools は、HTTP エンドポイントの 0 から 1 へのスケーリングを可能にする関連するプロジェクト Osiris をインストールします。  1 から *n* へのスケーリングは、従来の Kubernetes スケーリング ポリシーに依存します。

## <a name="next-steps"></a>次の手順
詳細については、次のリソースを参照してください。

* [カスタム イメージを使用して関数を作成する](functions-create-function-linux-custom-image.md)
* [Azure Functions をローカルでコーディングしてテストする](functions-develop-local.md)
* [Azure Functions の従量課金プランのしくみ](functions-scale.md)