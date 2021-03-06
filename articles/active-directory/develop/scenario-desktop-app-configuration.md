---
title: Web API を呼び出すデスクトップ アプリを構成する - Microsoft ID プラットフォーム | Azure
description: Web API を呼び出すデスクトップ アプリのコードを構成する方法について説明します
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/30/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 33b1724c25ef2d85aa8f838811864104b49576a3
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75423893"
---
# <a name="desktop-app-that-calls-web-apis---code-configuration"></a>Web API を呼び出すデスクトップ アプリ - コードの構成

お使いのアプリケーションを作成したところで、アプリケーションの座標を使用してコードを構成する方法について説明します。

## <a name="msal-libraries"></a>MSAL ライブラリ

デスクトップ アプリケーションをサポートしている Microsoft ライブラリは次のとおりです。

  MSAL ライブラリ | [説明]
  ------------ | ----------
  ![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/> MSAL.NET  | 複数のプラットフォーム (Linux、Windows、MacOS) でのデスクトップ アプリケーションの作成をサポートします。
  ![Python](media/sample-v2-code/logo_python.png) <br/> MSAL Python | 複数のプラットフォームでのデスクトップ アプリケーションの作成をサポートします。
  ![Java](media/sample-v2-code/logo_java.png) <br/> MSAL Java | 複数のプラットフォームでのデスクトップ アプリケーションの作成をサポートします。
  ![MSAL iOS](media/sample-v2-code/logo_iOS.png) <br/> MSAL iOS | macOS 上でのみ実行されるデスクトップアプリケーションがサポートされています。

## <a name="public-client-application"></a>パブリック クライアント アプリケーション

コードの観点から、デスクトップ アプリケーションは、パブリック クライアント アプリケーションです。 対話型認証を使用するかどうかによって、構成は少し異なります。

# <a name="nettabdotnet"></a>[.NET](#tab/dotnet)

MSAL.NET `IPublicClientApplication` を作成して操作する必要があります。

![IPublicClientApplication](media/scenarios/public-client-application.png)

### <a name="exclusively-by-code"></a>コードで排他的に

以下のコードでは、パブリック クライアント アプリケーションをインスタンス化して、職場および学校のアカウントまたは個人の Microsoft アカウントを使用して、Microsoft Azure のパブリック クラウドにユーザーをサインインさせます。

```csharp
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId)
    .Build();
```

前述のように、対話型認証またはデバイス コード フローを使用する場合は、`.WithRedirectUri` 修飾子を使用することが必要になる可能性があります。

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithDefaultRedirectUri()
        .Build();
```

### <a name="using-configuration-files"></a>構成ファイルの使用

次のコードは、構成オブジェクトからパブリック クライアント アプリケーションをインスタンス化します。構成オブジェクトは、プログラムで入力することも、構成ファイルから読み取ることもできます

```csharp
PublicClientApplicationOptions options = GetOptions(); // your own method
IPublicClientApplication app = PublicClientApplicationBuilder.CreateWithApplicationOptions(options)
        .WithDefaultRedirectUri()
        .Build();
```

### <a name="more-elaborated-configuration"></a>より詳細な構成

多数の修飾子を追加して、アプリケーションの構築を細かく設定できます。 たとえば、お使いのアプリケーションを国内クラウド (ここでは米国政府) のマルチテナント アプリケーションにする場合、次のように記述できます。

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithDefaultRedirectUri()
        .WithAadAuthority(AzureCloudInstance.AzureUsGovernment,
                         AadAuthorityAudience.AzureAdMultipleOrgs)
        .Build();
```

MSAL.NET には、ADFS 2019 用の修飾子も含まれています。

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAdfsAuthority("https://consoso.com/adfs")
        .Build();
```

最後に、Azure AD B2C テナントのトークンを取得する場合、次のコード スニペットで示されているとおり、お使いのテナントを指定できます。

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithB2CAuthority("https://fabrikamb2c.b2clogin.com/tfp/{tenant}/{PolicySignInSignUp}")
        .Build();
```

### <a name="learn-more"></a>詳細情報

MSAL.NET デスクトップ アプリケーションを構成する方法の詳細については、以下のとおりです。

- `PublicClientApplicationBuilder` で使用可能なすべての修飾子の一覧については、リファレンス ドキュメント [PublicClientApplicationBuilder](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.publicclientapplicationbuilder#methods) を参照してください
- `PublicClientApplicationOptions` に公開されているすべてのオプションの説明については、リファレンス ドキュメント内の [PublicClientApplicationOptions](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.publicclientapplicationoptions) を参照してください

### <a name="complete-example-with-configuration-options"></a>構成オプション付きの詳細な例

以下の `appsettings.json` 構成ファイルが含まれている .NET Core コンソール アプリケーションを想像してください。

```JSon
{
  "Authentication": {
    "AzureCloudInstance": "AzurePublic",
    "AadAuthorityAudience": "AzureAdMultipleOrgs",
    "ClientId": "ebe2ab4d-12b3-4446-8480-5c3828d04c50"
  },

  "WebAPI": {
    "MicrosoftGraphBaseEndpoint": "https://graph.microsoft.com"
  }
}
```

.NET で提供される構成フレームワークを使用してこのファイルを読み込むコードはほとんどありません。

```csharp
public class SampleConfiguration
{
 /// <summary>
 /// Authentication options
 /// </summary>
 public PublicClientApplicationOptions PublicClientApplicationOptions { get; set; }

 /// <summary>
 /// Base URL for Microsoft Graph (it varies depending on whether the application is ran
 /// in Microsoft Azure public clouds or national / sovereign clouds
 /// </summary>
 public string MicrosoftGraphBaseEndpoint { get; set; }

 /// <summary>
 /// Reads the configuration from a json file
 /// </summary>
 /// <param name="path">Path to the configuration json file</param>
 /// <returns>SampleConfiguration as read from the json file</returns>
 public static SampleConfiguration ReadFromJsonFile(string path)
 {
  // .NET configuration
  IConfigurationRoot Configuration;
  var builder = new ConfigurationBuilder()
                    .SetBasePath(Directory.GetCurrentDirectory())
                    .AddJsonFile(path);
  Configuration = builder.Build();

  // Read the auth and graph endpoint config
  SampleConfiguration config = new SampleConfiguration()
  {
   PublicClientApplicationOptions = new PublicClientApplicationOptions()
  };
  Configuration.Bind("Authentication", config.PublicClientApplicationOptions);
  config.MicrosoftGraphBaseEndpoint =
  Configuration.GetValue<string>("WebAPI:MicrosoftGraphBaseEndpoint");
  return config;
 }
}
```

アプリケーションを作成するには、以下のコードの記述するだけで済みます。

```csharp
SampleConfiguration config = SampleConfiguration.ReadFromJsonFile("appsettings.json");
var app = PublicClientApplicationBuilder.CreateWithApplicationOptions(config.PublicClientApplicationOptions)
           .WithDefaultRedirectUri()
           .Build();
```

また、`.Build()` メソッドを呼び出す前に、前述のように `.WithXXX` メソッドを呼び出すことで構成を上書きできます。

# <a name="javatabjava"></a>[Java](#tab/java)

次に、MSAL Java dev サンプルでその構成に使用されているクラスを紹介します。[TestData](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/TestData.java)。

```Java
PublicClientApplication app = PublicClientApplication.builder(TestData.PUBLIC_CLIENT_ID)
        .authority(TestData.AUTHORITY_COMMON)
        .build();
```

# <a name="pythontabpython"></a>[Python](#tab/python)

```Python
config = json.load(open(sys.argv[1]))

app = msal.PublicClientApplication(
    config["client_id"], authority=config["authority"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )
```

# <a name="macostabmacos"></a>[MacOS](#tab/macOS)

以下のコードでは、パブリック クライアント アプリケーションをインスタンス化して、職場および学校のアカウントまたは個人の Microsoft アカウントを使用して、Microsoft Azure のパブリック クラウドにユーザーをサインインさせます。

### <a name="quick-configuration"></a>クイック構成

Objective-C:

```objc
NSError *msalError = nil;

MSALPublicClientApplicationConfig *config = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"];    
MSALPublicClientApplication *application = [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&msalError];
```

Swift:
```swift
let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>")
if let application = try? MSALPublicClientApplication(configuration: config){ /* Use application */}
```

### <a name="more-elaborated-configuration"></a>より詳細な構成

多数の修飾子を追加して、アプリケーションの構築を細かく設定できます。 たとえば、お使いのアプリケーションを国内クラウド (ここでは米国政府) のマルチテナント アプリケーションにする場合、次のように記述できます。

Objective-C:

```objc
MSALAADAuthority *aadAuthority =
                [[MSALAADAuthority alloc] initWithCloudInstance:MSALAzureUsGovernmentCloudInstance
                                                   audienceType:MSALAzureADMultipleOrgsAudience
                                                      rawTenant:nil
                                                          error:nil];

MSALPublicClientApplicationConfig *config =
                [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"
                                                                redirectUri:@"<your-redirect-uri-here>"
                                                                  authority:aadAuthority];

NSError *applicationError = nil;
MSALPublicClientApplication *application =
                [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&applicationError];
```

Swift:

```swift
let authority = try? MSALAADAuthority(cloudInstance: .usGovernmentCloudInstance, audienceType: .azureADMultipleOrgsAudience, rawTenant: nil)

let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>", redirectUri: "<your-redirect-uri-here>", authority: authority)
if let application = try? MSALPublicClientApplication(configuration: config) { /* Use application */}
```
---

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [デスクトップ アプリのトークンの取得](scenario-desktop-acquire-token.md)
