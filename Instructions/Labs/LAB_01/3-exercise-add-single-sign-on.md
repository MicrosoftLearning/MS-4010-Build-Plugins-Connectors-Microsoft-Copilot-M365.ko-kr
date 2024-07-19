---
lab:
  title: 연습 2 - Single Sign-On 추가
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 연습 2 - Single Sign-On 추가

이 연습에서는 사용자에게 로그인 및 인증하라는 메시지가 표시되도록 메시지 확장을 업데이트합니다. Single Sign-On을 사용하도록 Bot Microsoft Entra 앱 등록 및 앱 매니페스트를 구성합니다. Microsoft Graph로 인증하고 메시지 확장 논리를 업데이트하여 Bot Framework 토큰 서비스를 통해 액세스 토큰을 가져오도록 Microsoft Entra 앱 등록을 구성합니다. 그런 다음 Microsoft Teams에서 테스트할 메시지 확장을 실행하고 디버그합니다.

## 작업 1 - Single Sign-On 구성

먼저 Bot Microsoft Entra 앱 등록을 구성합니다.

Visual Studio에서:

1. **infra\entra** 폴더에서 이름이 **entra.bot.manifest.json**인 파일을 엽니다.
1. 파일에서 **identifierUris** 배열을 업데이트하여 애플리케이션 ID URI를 설정합니다.

    ```json
    "identifierUris": [
        "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    ],
    ```

1. 파일에서 **oauth2Permissions** 배열을 업데이트하여 Teams에서 웹 API를 관리자 또는 사용자로 호출할 수 있도록 범위를 만듭니다.

    ```json
      "oauth2Permissions": [
        {
          "adminConsentDescription": "Allows Teams to call the app's web APIs as the current user.",
          "adminConsentDisplayName": "Teams can access app's web APIs",
          "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Enable Teams to call this app's web APIs with the same rights that you have",
          "userConsentDisplayName": "Teams can access app's web APIs and make requests on your behalf",
          "value": "access_as_user"
        }
      ]
    ```

1. 파일에서 **preAuthorizedApplications** 배열을 업데이트하여 Microsoft Teams, Microsoft Outlook 및 Microsoft 365용 Copilot 클라이언트를 권한 있는 클라이언트 목록에 추가합니다.

    ```json
      "preAuthorizedApplications": [
        {
          "appId": "1fec8e78-bce4-4aaf-ab1b-5451cc387264",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "5e3ce6c0-2b1f-4285-8d4b-75ee78787346",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "4765445b-32c6-49b0-83e6-1d93765276ca",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "0ec893e0-5785-4de6-99da-4ed124e5296c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "d3590ed6-52b3-4102-aeff-aad2292ab01c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "bc59ab01-8403-45c6-8796-ac3ef710b3e3",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "27922004-5251-4030-b22d-91ecd9a37ea4",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        }
      ],
    ```

1. 변경 내용을 저장합니다.

다음으로, 앱에서 Single Sign-On 흐름을 시작할 때 클라이언트가 사용해야 하는 리소스를 정의하도록 앱 매니페스트 파일을 업데이트합니다.

Visual Studio에서 다음을 테스트합니다.

1. **appPackage** 폴더에서 **manifest.json** 파일을 엽니다.
1. 파일에서 **설명** 뒤에 다음 코드를 추가합니다.

    ```json
    "webApplicationInfo": {
      "id": "${{BOT_ID}}",
      "resource": "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    },
    ```

1. 변경 내용을 저장합니다.

## 작업 2 - Microsoft Graph용 Microsoft Entra 앱 등록 매니페스트 파일 만들기

Microsoft Graph로 인증하려면 새 앱 등록 매니페스트 파일을 만듭니다.

Visual Studio에서 다음을 테스트합니다.

1. **infra\entra** 폴더에서 **entra.graph.manifest.json**이라는 파일을 만듭니다.
2.  파일에서 다음 코드를 추가합니다.

    ```json
    {
      "id": "${{GRAPH_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{GRAPH_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}",
      "accessTokenAcceptedVersion": 2,
      "signInAudience": "AzureADMultipleOrgs",
      "optionalClaims": {
        "idToken": [],
        "accessToken": [
          {
            "name": "idtyp",
            "source": null,
            "essential": false,
            "additionalProperties": []
          }
        ],
        "saml2Token": []
      },
      "requiredResourceAccess": [
        {
          "resourceAppId": "Microsoft Graph",
          "resourceAccess": [
            {
              "id": "Sites.ReadWrite.All",
              "type": "Scope"
            }
          ]
        }
      ],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": [
        {
          "url": "https://token.botframework.com/.auth/web/redirect",
          "type": "Web"
        }
      ]
    }
    ```

1. 변경 내용을 저장합니다.

**requiredResourceAccess** 배열은 API 권한 범위를 정의하고 **replyUrlsWithType** 배열은 앱 등록에서 리디렉션 URI를 정의합니다.

이제 앱 등록을 만들기 위한 작업으로 프로젝트 파일을 업데이트합니다.

1. 프로젝트 루트 폴더에서 **teamsapp.local.yml**을 엽니다.
1. 파일에서 **arm/deploy** 작업을 사용하는 단계를 찾습니다.
1. 이 단계 앞에 다음 코드를 추가합니다.

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: GRAPH_ENTRA_APP_ID
          clientSecret: SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET
          objectId: GRAPH_ENTRA_APP_OBJECT_ID
          tenantId: GRAPH_ENTRA_APP_TENANT_ID
          authority: GRAPH_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: GRAPH_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.graph.manifest.json"
          outputFilePath : "./build/entra.graph.manifest.${{TEAMSFX_ENV}}.json"
    ```

1. 변경 내용을 저장합니다.

## 작업 3 - OAuth 연결 설정 만들기

Azure AI Bot Service 연결 설정은 봇 및 메시지 확장에서 사용자 인증을 관리하는 데 사용됩니다.

먼저 연결 설정을 환경 변수로 만드는 데 사용되는 OAuth 연결 설정의 이름을 중앙 집중화한 다음 런타임에 환경 변수 값을 사용하는 코드를 추가합니다.

Visual Studio에서 다음을 테스트합니다.  

1. **env** 폴더에서 **.env.local**을 엽니다
1.  파일에서 다음 코드를 추가합니다.

    ```text
    CONNECTION_NAME=MicrosoftGraph
    ```

1. 프로젝트 루트 폴더에서 ** teamsapp.local.yml**이라는 파일을 엽니다.
1. 파일에서 **./appsettings.Development.json** 파일을 대상으로 하는 **file/createOrUpdateJsonFile** 작업을 사용하는 단계를 찾습니다.
1. 콘텐츠 배열을 업데이트하고 **CONNECTION_NAME** 변수를 추가합니다.

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. 변경 내용을 저장합니다.
1. 프로젝트 루트 폴더에서 **Config.cs**를 엽니다.
1. **ConfigOptions** 클래스에서 이름이 **CONNECTION_NAME**인 새 문자열 속성을 추가합니다.

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. 변경 내용을 저장합니다.
1. 프로젝트 루트 폴더에서 이름이 **Program.cs**인 파일을 엽니다.
1. 파일에서 새 줄을 추가하여 **CONNECTION_NAME** 환경 변수를 앱 구성 설정으로 추가합니다.

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    ```

다음으로, 앱 구성에 액세스하도록 봇 작업 처리기를 업데이트합니다.

1. **검색** 폴더에서 **SearchApp.cs** 파일을 엽니다.
1. **SearchApp** 클래스에서 **connectionName**이라는 이름으로 읽기 전용 문자열 속성을 만듭니다.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    }
    ```

1. 앱 구성을 삽입하고 connectionName 속성의 값을 설정하는 생성자를 만듭니다.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    
      public SearchApp(IConfiguration configuration)
      {
        connectionName = configuration["CONNECTION_NAME"];
      }  
    }
    ```

1. 변경 내용을 저장합니다.

다음으로, OAuth 연결 설정을 프로비전하도록 Bicep 파일을 업데이트합니다.

먼저 Microsoft Graph Microsoft Entra 앱 등록의 자격 증명과 연결 설정의 이름을 전달하도록 매개 변수 파일을 업데이트합니다.

1. 인프라 **폴더**에서 이름이 **azure.parameters.local.json**인 파일을 엽니다.
1. 매개 변수 **개체**에서 **graphEntraAppClientId**, **graphEntraAppClientSecret**, **connectionName** 매개 변수를 추가합니다.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "resourceBaseName": {
          "value": "bot-${{RESOURCE_SUFFIX}}-${{TEAMSFX_ENV}}"
        },
        "botEntraAppClientId": {
          "value": "${{BOT_ID}}"
        },
        "botDisplayName": {
          "value": "${{APP_DISPLAY_NAME}}"
        },
        "botAppDomain": {
          "value": "${{BOT_DOMAIN}}"
        },
        "graphEntraAppClientId": {
          "value": "${{GRAPH_ENTRA_APP_ID}}"
        },
        "graphEntraAppClientSecret": {
          "value": "${{SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. 변경 내용을 저장합니다.

다음으로, Bicep 파일을 업데이트합니다.

1. **인프라** 폴더에서 이름이 **azure.local.bicep**인 파일을 엽니다.
1. 파일에서 **botAppDomain** 매개 변수 선언 후 **graphEntraAppClientId**, **graphEntraAppClientSecret**, **connectionName** 매개 변수 선언을 추가합니다.

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. **azureBotRegistration** 모듈에서 **graphEntraAppClientId**, **graphEntraAppClientSecret**, **connectionName** 매개 변수를 추가합니다.

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        graphEntraAppClientId: graphEntraAppClientId
        graphEntraAppClientSecret: graphEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. 변경 내용을 저장합니다.

마지막으로 봇 등록 Bicep 파일을 업데이트합니다.

1. **infra/botRegistration** 폴더에서 **azurebot.bicep**이라는 파일을 엽니다.
1. 파일에서 **botAppDomain** 매개 변수 선언 후 **graphEntraAppClientId**, **graphEntraAppClientSecret**, **connectionName** 매개 변수 선언을 추가합니다.

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. **botServiceM365ExtensionsChannel** 리소스 뒤에 Microsoft Graph 연결에 대한 새 리소스를 추가합니다.

    ```bicep
    resource botServicesMicrosoftGraphConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: graphEntraAppClientId
        clientSecret: graphEntraAppClientSecret
        scopes: 'email offline_access openid profile Sites.ReadWrite.All'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${ botAadAppClientId}'
          }
        ]
      }
    }
    ```

1. 변경 내용을 저장합니다.

## 작업 4 - 사용자 쿼리 인증

다음으로, 메시지 확장을 사용하여 검색을 시작할 때 사용자를 인증하는 코드를 추가합니다.

Visual Studio에서 다음을 테스트합니다.

1. **검색** 폴더에서 **SearchApp.cs** 파일을 엽니다.
1. 파일의 Bot Framework SDK에서 **인증** 네임스페이스를 추가하여 시작합니다.

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. **OnTeamsMessagingExtensionQueryAsync** 메서드의 시작 부분에 다음 코드를 추가합니다.

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    
    if (!HasToken(tokenResponse))
    {
        return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

위의 코드 블록은 사용자 인증을 처리하는 세 가지 방법을 사용합니다.

- **GetToken**은 토큰 서비스 클라이언트를 사용하여 현재 사용자에 대한 액세스 토큰을 가져옵니다.
- **HasToken**은 토큰 서비스의 응답에 액세스 토큰이 포함되어 있는지 확인합니다.
- 토큰이 반환되지 않으면 **CreateAuthResponse**가 호출되고 사용자 인터페이스에 로그인 링크가 표시되는 응답을 반환합니다.

이제 **SearchApp** 클래스에서 메서드를 만듭니다 .

- 다음 코드를 사용하여 **GetToken** 메서드를 만듭니다.

```csharp
private static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
{
  var magicCode = string.Empty;

  if (!string.IsNullOrEmpty(state))
  {
    if (int.TryParse(state, out var parsed)) 
    {
        magicCode = parsed.ToString();
    }
  }

  return await userTokenClient.GetUserTokenAsync(userId, connectionName, channelId, magicCode, cancellationToken);
}
```

먼저 상태 매개 변수가 null이거나 비어 있지 않은지 확인합니다. 코드는 int.TryParse 메서드를 사용하여 정수로 구문 분석하려고 시도합니다. 구문 분석이 성공하면 구문 분석된 값이 magicCode 변수에 문자열로 할당됩니다. 그런 다음 magicCode는 다른 매개 변수와 함께 GetUserTokenAsync 메서드에 인수로 전달됩니다. GetUserTokenAsync 메서드는 magicCode를 사용하여 요청의 신뢰성을 확인합니다. 인증 프로세스를 시작한 동일한 엔터티에서 사용자 토큰을 요청합니다.

- 다음 코드를 사용하여 HasToken 메서드를 만듭니다.

```csharp
private static bool HasToken(TokenResponse tokenResponse)
{
    return tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
}
```

토큰 응답 및 응답의 토큰 속성이 비어 있지 않거나 null이 아닌지 확인하여 토큰 서비스에서 유효한 토큰을 가져오는지 여부를 확인합니다.

- 다음 코드를 사용하여 **CreateAuthResponse** 메서드를 만듭니다.

```csharp
private static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
{
    var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);

    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "auth",
            SuggestedActions = new MessagingExtensionSuggestedAction
            {
                Actions = new List<CardAction>
                {
                    new() {
                        Type = ActionTypes.OpenUrl,
                        Value = resource.SignInLink,
                        Title = "Sign In",
                    },
                },
            },
        },
    };
}
```

먼저 **GetSignInResourceAsync** 메서드를 사용하여 토큰 서비스에서 로그인 링크를 검색합니다. 로그인 링크는 **MessagingExtensionResponse** 개체를 생성하는 데 사용됩니다. 새 개체를 만들고 응답의 **ComposeExtension** 속성을 새 **MessagingExtensionResult** 개체로 설정합니다. 결과의 type 속성은 결과가 인증 응답임을 나타내는 "인증"으로 설정됩니다. 결과의 **SuggestedActions** 속성은 새 **MessagingExtensionSuggestedAction** 개체로 설정됩니다. 제안된 작업의 Actions 속성은 단일 **CardAction** 개체를 포함하는 목록으로 설정됩니다. 이 **CardAction** 개체는 사용자가 수행할 수 있는 작업을 나타냅니다. **CardAction**의 Type 속성은 URL을 여는 작업임을 나타내는 **ActionTypes.OpenUrl**로 설정됩니다. Value 속성은 리소스에서 검색된 로그인 링크로 설정됩니다. Title 속성은 작업의 제목을 지정하는 "로그인"으로 설정됩니다. 마지막으로, 생성된 응답이 메서드에서 반환됩니다.

사용자가 로그인 링크를 따르면 외부 도메인에 호스트된 리소스로 이동됩니다. 도메인은 앱 매니페스트 파일에 포함되어야 합니다. 앱 매니페스트에 Bot Framework 토큰 서비스 도메인을 추가합니다.

Visual Studio에서 다음을 테스트합니다.

1. **appPackage** 폴더에서 **manifest.json**을 엽니다.
1. 파일에서 **validDomains** 배열을 업데이트하고 토큰 서비스의 도메인을 추가합니다.

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]    
    ```

1. 변경 내용을 저장합니다.

## 작업 5 - 리소스 프로비전

이제 모든 항목이 준비되면 Teams 앱 종속성 준비 프로세스를 실행하여 필요한 리소스를 프로비전합니다.

Visual Studio에서 다음을 테스트합니다.

1. **솔루션 탐색기**에서 **TeamsApp** 프로젝트를 마우스 오른쪽 단추로 클릭합니다.
1. **Teams Toolkit** 메뉴를 확장하고 **Teams 앱 종속성 준비**를 선택합니다.
1. **Microsoft 365 계정** 대화 상자에서 **계속**을 선택합니다.
1. **프로비전** 대화 상자에서 **프로비전**을 선택합니다.
1. **Teams Toolkit 경고** 대화 상자에서 **프로비전**을 선택합니다.
1. **Teams 도구 키트 정보** 대화 상자에서 **프로비전된 리소스 보기**를 선택하여 새 브라우저 창을 엽니다.

잠시 시간을 내어 Azure에서 생성 및 업데이트되는 리소스를 살펴봅니다.

## 작업 6 - 실행 및 디버그

이제 웹 서비스를 시작하고 Microsoft Teams에서 메시지 확장을 테스트합니다.

Visual Studio에서 다음을 테스트합니다.

1. **F5** 키를 눌러 디버깅 세션을 시작하고 Microsoft Teams 웹 클라이언트로 이동하는 새 브라우저 창을 엽니다.
1. 브라우저에서 필요한 경우 Microsoft 365 계정 자격 증명을 입력하고 Microsoft Teams로 계속 진행합니다.
1. 앱 설치 대화 상자에서 **추가**를 선택합니다.
1. 신규 또는 기존 Microsoft Teams 채팅을 엽니다.
1. 메시지 작성 영역에서 **...** 를 선택하여 앱 플라이아웃을 엽니다.
1. 앱 목록에서 **Contoso 제품**을 선택하여 메시지 확장을 엽니다.
1. 텍스트 상자에 **Bot Builder**를 입력하여 검색을 시작합니다.
1. **이 앱을 사용하려면 로그인해야 합니다.** 라는 메시지가 표시됩니다.
1. **로그인 링크**를 선택하여 새 탭을 열고 인증 흐름을 시작합니다.
1. 사용 권한 동의 페이지에서 요청되는 사용 권한을 검토합니다.
1. **수락**을 선택한 후 탭을 닫고 Microsoft Teams로 돌아갑니다.
1. 결과 목록에서 **결과를 선택하여** 메시지 작성 상자에 카드를 포함하고 보냅니다.

디버깅 세션을 중지하는 브라우저입니다.

[ 다음 연습으로 계속...](./4-exercise-retrieve-product-information-from-sharepoint-online.md)