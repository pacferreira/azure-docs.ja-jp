---
title: C# で REST 呼び出しを使用してモデルを取得する
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/18/2019
ms.author: diberry
ms.openlocfilehash: cc8f01d5608b36521185695cd5f7be531ba7aeaf
ms.sourcegitcommit: c22327552d62f88aeaa321189f9b9a631525027c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/04/2019
ms.locfileid: "73505861"
---
## <a name="prerequisites"></a>前提条件

* スターター キー。
* cognitive-services-language-understanding GitHub リポジトリから [TravelAgent](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/documentation-samples/quickstarts/change-model/TravelAgent.json) アプリをインポートします。
* インポートした TravelAgent アプリ用の LUIS アプリケーション ID。 アプリケーション ID は、アプリケーション ダッシュボードに表示されます。
* 発話を受け取るアプリケーション内のバージョン ID。 既定の ID は "0.1" です。
* [Node.js](https://nodejs.org/) プログラミング言語 
* [Visual Studio Code](https://code.visualstudio.com/)

## <a name="example-utterances-json-file"></a>発話の例の JSON ファイル

[!INCLUDE [Quickstart explanation of example utterance JSON file](get-started-get-model-json-example-utterances.md)]


## <a name="get-luis-key"></a>LUIS キーを取得する

[!INCLUDE [Use authoring key for endpoint](../includes/get-key-quickstart.md)]

## <a name="change-model-programmatically"></a>プログラムを使用してモデルを変更する

Go を使用して、機械学習されたエンティティ [API](https://aka.ms/luis-apim-v3-authoring) をアプリケーションに追加します。 

1. `model.js` という名前で新しいファイルを作成します。 次のコードを追加します。

    ```javascript
    var request = require('request');
    var requestpromise = require('request-promise');
    
    const LUIS_authoringKey = "YOUR-KEY";
    const LUIS_endpoint = "YOUR-ENDPOINT";
    const LUIS_appId = "YOUR-APP-ID";
    const LUIS_versionId = "0.1";
    const addUtterancesURI = `https://${LUIS_endpoint}/luis/authoring/v3.0-preview/apps/${LUIS_appId}/versions/${LUIS_versionId}/examples`;
    const addTrainURI = `https://${LUIS_endpoint}/luis/authoring/v3.0-preview/apps/${LUIS_appId}/versions/${LUIS_versionId}/train`;
    
    const utterances = [
            {
              'text': 'go to Seattle today',
              'intentName': 'BookFlight',
              'entityLabels': [
                {
                  'entityName': 'Location::LocationTo',
                  'startCharIndex': 6,
                  'endCharIndex': 12
                }
              ]
            },
            {
                'text': 'a barking dog is annoying',
                'intentName': 'None',
                'entityLabels': []
            }
          ];
    
    const main = async() =>{
    
    
        await addUtterance();
        await train("POST");
        await trainStatus("GET");
    
    }
    const addUtterance = async () => {
    
        const options = {
            uri: addUtterancesURI,
            method: 'POST',
            headers: {
                'Ocp-Apim-Subscription-Key': LUIS_authoringKey
            },
            json: true,
            body: utterances
        };
    
        const reponse = await requestpromise(options)
        console.log(reponse.body);
    }
    const train = async (verb) => {
    
        const options = {
            uri: addTrainURI,
            method: verb, 
            headers: {
                'Ocp-Apim-Subscription-Key': LUIS_authoringKey
            },
            json: true,
            body: null // The body can be empty for a training request
        };
    
        const reponse = await requestpromise(options)
        console.log(reponse.body);
    }
    
    // MAIN
    main().then(() => console.log("done")).catch((err)=> console.log(err returned));
    ```
1. 次の値を置き換えます。

    * `YOUR-KEY` をご利用のスターター キーに
    * `YOUR-ENDPOINT` をご利用のエンドポイントに (例: `westus2.api.cognitive.microsoft.com`)
    * `YOUR-APP-ID` をご利用のアプリの ID に

1. このファイルを作成したときと同じディレクトリからコマンド プロンプトで次のコマンドを入力し、ファイルを実行します。

    ```console
    node model.js
    ```  

## <a name="luis-keys"></a>LUIS キー

[!INCLUDE [Use authoring key for endpoint](../includes/starter-key-explanation.md)]

## <a name="clean-up-resources"></a>リソースのクリーンアップ

このクイックスタートを完了したら、ファイル システムからファイルを削除します。 

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [アプリのベスト プラクティス](../luis-concept-best-practices.md)