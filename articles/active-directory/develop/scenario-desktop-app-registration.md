---
title: Web API を呼び出すデスクトップ アプリを登録する - Microsoft ID プラットフォーム | Azure
description: Web API を呼び出すデスクトップ アプリ (アプリの登録) をビルドする方法について学習します
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/09/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: dabc96ef669f0c0c61a7bca4a16828294cf404df
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75423851"
---
# <a name="desktop-app-that-calls-web-apis---app-registration"></a>Web API を呼び出すデスクトップ アプリ - アプリの登録

この記事では、デスクトップ アプリケーションのアプリ登録の詳細について説明しています。

## <a name="supported-accounts-types"></a>サポートされているアカウントの種類

デスクトップ アプリケーションでサポートされるアカウントの種類は、利用するエクスペリエンスによって異なります。 この関係により、サポートされるアカウントの種類は、使用するフローによって決まります。

### <a name="audience-for-interactive-token-acquisition"></a>対話型トークン取得の対象ユーザー

デスクトップ アプリケーションで対話型認証を使用する場合は、任意の[アカウントの種類](quickstart-register-app.md#register-a-new-application-using-the-azure-portal)からユーザーをサインインできます。

### <a name="audience-for-desktop-app-silent-flows"></a>デスクトップ アプリのサイレント フローの対象ユーザー

- 統合 Windows 認証またはユーザー名/パスワードを使用する場合は、アプリケーションでは独自のテナント (LOB 開発者) または Azure Active Directory 組織 (ISV シナリオ) のユーザーをサインインする必要があります。 これらの認証フローは、Microsoft の個人用アカウントではサポートされていません。
- デバイス コード フローを使用する場合は、ユーザーの Microsoft の個人アカウントを使用してユーザーをサインインすることはまだできません。
- B2C 機関とポリシーを渡すソーシャル ID を使用してユーザーにサインインする場合は、対話型のユーザー名とパスワード認証のみを使用できます。

## <a name="redirect-uris"></a>リダイレクト URI

デスクトップ アプリケーションで使用するリダイレクト URI は、使用するフローによって決まります。

- **対話型認証**または**デバイス コード フロー**を使っている場合は、`https://login.microsoftonline.com/common/oauth2/nativeclient` を使う必要があります。 アプリケーションの **[認証]** セクションで対応する URL をクリックすることで、この構成が実現します。
  
  > [!IMPORTANT]
  > 現在、MSAL.NET は、Windows で実行されているデスクトップ アプリケーションでは、既定で別のリダイレクト URI を使用しています (`urn:ietf:wg:oauth:2.0:oob`)。 今後、この既定を変更することを予定しているため、`https://login.microsoftonline.com/common/oauth2/nativeclient` を使用することをお勧めします。

- MacOS 用のネイティブ Objective-C アプリまたは Swift アプリを構築している場合は、アプリケーションのバンドル識別子に基づいて redirectUri を次の形式で登録する必要があります: **msauth.<your.app.bundle.id>://auth** (<your.app.bundle.id> をご自分のアプリケーションのバンドル識別子に置き換えます)
- アプリで統合 Windows 認証またはユーザー名/パスワードのみを使用する場合は、アプリケーションのリダイレクト URI を登録する必要はありません。 これらのフローが Microsoft ID プラットフォームの v2.0 エンドポイントへのラウンド トリップを行い、アプリケーションは特定の URI にコールバックされません。
- デバイス コード フロー、統合 Windows 認証、ユーザー名/パスワードを、いずれのリダイレクト URI も持たない機密性の高いクライアント アプリケーション フロー (デーモン アプリケーションで使用されるクライアント資格証明フロー) と区別するには、アプリケーションがパブリック クライアント アプリケーションであることを示す必要があります。 この構成を実現するには、アプリケーションの **[認証]** セクションにアクセスします。 次に、 **[詳細設定]** サブセクションの **[既定のクライアントの種類]** 段落で、 **[アプリケーションは、パブリック クライアントとして扱います]** の質問に対して **[はい]** を選択します。

  ![パブリック クライアントを許可する](media/scenarios/default-client-type.png)

## <a name="api-permissions"></a>API のアクセス許可

デスクトップ アプリケーションは、サインインしたユーザーのために API を呼び出します。 委任されたアクセス許可を要求する必要があります。 ただし、アプリケーションのアクセス許可を要求することはできません。これは[デーモン アプリケーション](scenario-daemon-overview.md)でのみ処理されます。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [デスクトップ アプリ - アプリの構成](scenario-desktop-app-configuration.md)
