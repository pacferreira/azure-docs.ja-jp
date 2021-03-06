---
title: Azure Batch プール サイズ変更開始イベント
description: Batch のプール サイズ変更開始イベントのリファレンスです。 次の例では、プールのサイズを 0 ノードから 2 ノードへ手動でサイズ変更する場合のプール サイズ変更開始イベントの本文を示します。
services: batch
author: ju-shim
manager: gwallace
ms.assetid: ''
ms.service: batch
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 04/20/2017
ms.author: jushiman
ms.openlocfilehash: 8e2df1c2264392ab79e2dcbb7e58977f341b14d9
ms.sourcegitcommit: dbcc4569fde1bebb9df0a3ab6d4d3ff7f806d486
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/15/2020
ms.locfileid: "76026619"
---
# <a name="pool-resize-start-event"></a>プールのサイズ変更の開始イベント

 このイベントは、プールのサイズ変更が開始されたときに発生します。 プール サイズ変更は非同期イベントであるため、サイズ変更操作が完了するとプール サイズ変更の完了イベントが発生することが想定されます。

 次の例では、プールのサイズを 0 ノードから 2 ノードへ手動でサイズ変更する場合のプール サイズ変更開始イベントの本文を示します。

```
{
    "id": "myPool1",
    "nodeDeallocationOption": "Invalid",
    "currentDedicatedNodes": 0,
    "targetDedicatedNodes": 2,
    "currentLowPriorityNodes": 0,
    "targetLowPriorityNodes": 2,
    "enableAutoScale": false,
    "isAutoPool": false
}
```

|要素|種類|メモ|
|-------------|----------|-----------|
|`id`|String|プールの ID。|
|`nodeDeallocationOption`|String|プールからノードが削除されるとき、プールのサイズが減少するかを指定します。<br /><br /> 次のいずれかの値になります。<br /><br /> **requeue** – 実行中のタスクを終了して、再度キューに入れます。 このタスクは、ジョブが有効になると再び実行されます。 タスクが終了するとすぐにノードを削除します。<br /><br /> **terminate** – 実行中のタスクを終了します。 タスクは再び実行されることがありません。 タスクが終了するとすぐにノードを削除します。<br /><br /> **taskcompletion** – 現在実行中のタスクが完了することを許可します。 待機中に新しいタスクをスケジュールしません。 すべてのタスクが完了するとノードを削除します。<br /><br /> **Retaineddata** - 現在実行中のタスクを完了できるようにしてから、すべてのタスク データ保有期間が終了するまで待機します。 待機中に新しいタスクをスケジュールしません。 すべてのタスク保有期間が終了したとき、ノードを削除します。<br /><br /> 既定値は requeue です。<br /><br /> プールのサイズが増加している場合、値は**無効**に設定されます。|
|`currentDedicatedNodes`|Int32|プールに現在割り当てられているコンピューティング ノードの数。|
|`targetDedicatedNodes`|Int32|プールに要求されたコンピューティング ノード数。|
|`currentLowPriorityNodes`|Int32|プールに現在割り当てられているコンピューティング ノードの数。|
|`targetLowPriorityNodes`|Int32|プールに要求されたコンピューティング ノード数。|
|`enableAutoScale`|Bool|プールのサイズを自動的に調整し続けるかどうかを指定します。|
|`isAutoPool`|Bool|プールがジョブの AutoPool メカニズムを介して作成されたかどうかを指定します。|
