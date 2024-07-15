---
lab:
  title: 연습 3 - SharePoint 온라인에서 제품 정보 검색하기
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 연습 3 - SharePoint 온라인에서 제품 정보 검색하기

이 연습에서는 제품 정보를 목록의 항목으로 저장하는 SharePoint 온라인 사이트를 프로비저닝하고 구성합니다. Microsoft Graph SDK를 사용하여 SharePoint Online에서 목록 항목을 검색하고 검색 결과에서 목록 항목 데이터를 반환하려면 메시지 확장 코드를 업데이트합니다. 마지막으로 메시지 확장을 실행 및 디버그하고 Microsoft Teams에서 테스트합니다.

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text=" Microsoft Teams의 검색 기반 메시지 확장 프로그램에서 반환된 검색 결과 스크린샷입니다. 검색 결과는 SharePoint Online에서 반환됩니다. 각 검색 결과에는 제품 이름, 카테고리 및 제품 이미지가 표시됩니다." lightbox="../media/4-search-results-sharepoint-online.png":::

## 작업 1 - 제품 마케팅 SharePoint 사이트 프로비저닝 및 구성

먼저 SharePoint 룩북 서비스를 사용하여 SharePoint 온라인 사이트를 만듭니다.

웹 브라우저에서:

1. [ https://lookbook.microsoft.com](https://lookbook.microsoft.com)에서 ** SharePoint look book**으로 이동합니다.
1. 상단 탐색에서 ** 디자인 뷰**를 펼칩니다.
1. **디자인 보기** 메뉴에서 **팀**을 확장하고 **제품 지원**을 선택합니다.
1. ** 테넌트 추가**를 선택합니다.
1. 메시지가 표시되면 테넌트에 로그인합니다.
1. 권한 동의 화면에서 필요한 권한을 검토하고 ** 동의**를 선택하여 SharePoint 룩북 서비스로 돌아갑니다.
1. 양식에서 기본값을 수락하고 ** 프로비저닝**을 선택합니다.

사이트 프로비저닝이 완료되면 회원님의 이메일 주소로 이메일이 전송되어 이를 알려드립니다. 이 프로세스를 완료하는 데 몇 분이 걸릴 수 있습니다.

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text=" 제품 지원 SharePoint 온라인 팀 사이트의 홈페이지 스크린샷입니다. 최근에 출시된 제품 목록이 표시됩니다." lightbox="../media/1-sharepoint-online-product-support-site.png":::

Microsoft Graph API를 사용하여 목록을 쿼리할 때 제목 및 소매 카테고리 열에서 필터링을 사용하려면 목록에 인덱스를 생성합니다.

웹 브라우저에서 계속합니다:

1. ** <https://tenant.sharepoint.com/sites/productmarketing>** 의 ** 제품 지원** 사이트로 이동하여 ** 테넌트**를 SharePoint 온라인 인스턴스의 이름으로 바꿉니다.
1. ** Microsoft 365 제품군 모음**에서 ** 설정 톱니바퀴**를 선택하여 설정 사이드 패널을 엽니다.
1. ** SharePoint** 제목 아래에서 ** 사이트 콘텐츠**를 선택합니다.
1. 목록 및 라이브러리 목록에서 ** 제품** 목록 위로 마우스를 가져가 ** 세로 점 3개** 아이콘을 선택하여 ** 작업 표시** 메뉴를 펼친 다음 ** 설정**을 선택합니다.
1. ** 열** 섹션의 열 목록 아래에서 ** 인덱싱된 열**을 선택합니다.
1. ** 새 인덱스 만들기**를 선택합니다.

## 작업 2 - SharePoint 호스트 이름 및 사이트 URL 환경 변수 추가하기

다음으로, SharePoint Online 인스턴스의 호스트 이름과 제품 지원 사이트 URL을 환경 변수로 중앙 집중화합니다. 그런 다음 값을 런타임에 사용할 환경 변수로 노출하고 코드를 업데이트하여 값을 읽습니다.

Visual Studio를 엽니다:

1. ** env** 폴더에서 ** .env.local**이라는 파일을 엽니다.
1. 파일에서 ** SPO_HOSTNAME** 및 ** SPO_SITE_URL** 환경 변수를 추가하고, ** tenant**를 SharePoint Online 인스턴스의 이름으로 바꿉니다:

    ```text
    SPO_HOSTNAME=tenant.sharepoint.com
    SPO_SITE_URL=sites/productmarketing
    ```

1. 변경 내용을 저장합니다.

다음으로 앱 설정 파일에 환경 변수를 작성하는 작업을 업데이트합니다.

1. 프로젝트 루트 폴더에서 ** teamsapp.local.yml**이라는 파일을 엽니다.
1. ** ./appsettings.Development.json** 파일을 대상으로 하는 ** file/createOrUpdateJsonFile** 작업을 사용하는 단계를 찾습니다.
1. 파일에서 **콘텐츠** 배열을 업데이트하고 **SPO_HOSTNAME** 및 **SPO_SITE_URL** 변수를 추가합니다.

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
            SPO_HOSTNAME: ${{SPO_HOSTNAME}}
            SPO_SITE_URL: ${{SPO_SITE_URL}}
    ```

1. 변경 내용을 저장합니다.

이제 새 환경 변수를 포함하도록 ConfigOptions 클래스를 업데이트합니다.

1. 프로젝트 루트 폴더에서 Config.cs를 엽니다.
1. ConfigOptions 클래스에서 이름이 SPO_HOSTNAME 및 SPO_SITE_URL인 새 문자열 속성을 추가합니다.

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
      public string SPO_HOSTNAME { get; set; }
      public string SPO_SITE_URL { get; set; }
    }
    ```

1. 변경 내용을 저장합니다.

다음으로 두 환경 변수를 사용해 앱 구성을 업데이트합니다.

1. 프로젝트 루트 폴더에서 Program.cs를 엽니다.
1. 새 줄을 추가하여 SPO_HOSTNAME 및 SPO_SITE_URL 환경 변수를 앱 구성 설정으로 추가합니다.

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    builder.Configuration["SPO_HOSTNAME"] = config.SPO_HOSTNAME;
    builder.Configuration["SPO_SITE_URL"] = config.SPO_SITE_URL;
    ```

1. 변경 내용을 저장합니다.

마지막 단계는 앱 구성에서 값을 읽고 읽기 전용 속성에 값을 저장하도록 봇 작업 처리기를 업데이트하는 것입니다.

1. 검색 폴더에서 이름이 SearchApp.cs인 파일을 엽니다.
1. SearchApp 클래스에서 spoHostname 및 spoSiteUrl 이름을 사용하여 읽기 전용 문자열 속성을 만듭니다.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
      private readonly string spoHostname;
      private readonly string spoSiteUrl;
    }
    ```

1. 삽입된 앱 구성을 사용하여 속성 값을 설정하도록 생성자를 업데이트합니다.

    ```csharp
    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    } 
    ```

1. 변경 내용을 저장합니다.

## 작업 3 - 검색 명령 업데이트

메시지 확장이 제품 정보를 반환할 때 검색 명령 제목 및 설명을 업데이트하고 매개 변수 이름과 설명을 업데이트합니다.

Visual Studio에서 다음을 테스트합니다.

1. **appPackage** 폴더에서 **manifest.json** 파일을 엽니다.
1. **composeExtensions** 배열에서 명령 개체를 다음으로 업데이트합니다.

    ```json
    "composeExtensions": [
      {
        "botId": "${{BOT_ID}}",
        "commands": [
          {
            "id": "Search",
            "type": "query",
            "title": "Products",
            "description": "Find products by name",
            "initialRun": false,
            "fetchTask": false,
            "context": [
              "commandBox",
              "compose",
              "message"
            ],
            "parameters": [
              {
                "name": "ProductName",
                "title": "Product name",
                "description": "The name of the product as a keyword",
                "inputType": "text"
              }
            ]
          }
        ]
      }
    ],
    ```

1. 변경 내용을 저장합니다.

## 작업 4 - 사용자 쿼리 값 가져오기

OnTeamsMessagingExtensionQueryAsync 메서드를 실행할 때 가장 먼저 수행할 작업은 사용자가 검색 상자에 입력한 내용을 이해하는 것입니다.

먼저 기존 코드를 제거해 보겠습니다.

Visual Studio에서 다음을 테스트합니다.

1. **검색** 폴더에서 **SearchApp.cs** 파일을 엽니다.
1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 액세스 토큰을 확인하는 **if** 문 **뒤**의 모든 코드를 제거합니다.
1. **SearchApp** 클래스에서 **FindPackages** 메서드 및 **_adaptiveCardFilePath** 속성을 제거합니다.

기존 코드를 제거하고 나면 **SearchApp** 클래스는 다음 코드 조각과 비슷하게 보입니다.

```csharp
public class SearchApp : TeamsActivityHandler
{
    private readonly string connectionName;
    private readonly string spoHostname;
    private readonly string spoSiteUrl;

    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    }

    protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
    {
        var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
        var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);

        if (!HasToken(tokenResponse))
        {
            return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
        }
    }
}
```

이제 **ProductName** 매개 변수의 값을 가져오는 코드를 작성해 보겠습니다.

1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 **MessagingExtensionQuery** 개체의 **Parameters** 배열로부터 **ProductName** 매개 변수 값을 검색하는 코드를 추가합니다.

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    ```

1. **SearchApp** 클래스에서 **GetQueryData** 메서드를 구현합니다.

    ```csharp
    private static string GetQueryData(IList<MessagingExtensionParameter> parameters, string key)
    {
      if (parameters.Any() != true)
      {
        return string.Empty;
      }
    
      var foundPair = parameters.FirstOrDefault(pair => pair.Name == key);
      return foundPair?.Value?.ToString() ?? string.Empty;
    }
    ```

1. 변경 내용을 저장합니다.

**GetQueryData** 메서드는 **MessagingExtensionParameter** 개체 목록에서 특정 키와 연결된 값을 검색하는 데 사용됩니다. **MessagingExtensionQuery** 개체의 매개변수 배열에서 데이터를 추출하는 편리한 방법을 제공합니다.

## 작업 5 - SharePoint 목록 쿼리 OData 필터 만들기

이제 사용자가 전달한 값이 있으므로 이 값을 사용하여 OData 쿼리 필터를 만듭니다. 필터는 제품 이름이 포함된 제목 열에서 SharePoint Online 목록을 쿼리하는 데 사용됩니다.

Visual Studio에서 다음을 테스트합니다.

1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 **filterQuery** 변수를 만드는 코드를 추가합니다.

    ```csharp
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. 변경 내용을 저장합니다.

이 코드는 **name** 매개 변수를 기반으로 필터 쿼리를 생성합니다. name 매개 변수가 제공되면 제공된 이름으로 시작하는 **Title** 필드가 포함된 항목을 검색하는 필터 식을 만듭니다. name 매개 변수가 제공되지 않으면 빈 문자열을 필터 쿼리로 할당합니다. 결과 필터 쿼리는 코드의 뒷부분에서 SharePoint 사이트에서 필터링된 항목을 검색하는 데 사용됩니다.

## 작업 6 - Microsoft Graph SDK 설치 및 구성

Microsoft Graph에 인증된 요청을 실행하려면 **Microsoft Graph SDK**를 사용합니다.

NuGet에서 Microsoft Graph SDK 패키지를 설치하고 토큰 서비스에서 가져온 액세스 토큰을 사용한 다음 새 **GraphServiceClient**를 초기화할 수 있는 **TokenProvider** 클래스를 만듭니다.

Visual Studio에서 다음을 테스트합니다.

1. 솔루션 탐색기에서 **MsgExtProductSupport** 프로젝트를 마우스 오른쪽 단추로 클릭합니다.
1. **NuGet 패키지 관리...** 를 선택합니다.
1. **찾아보기** 탭을 선택하고 **Microsoft.Graph**을 검색합니다.
1. 결과 목록에서 **Microsoft.Graph**를 선택합니다.
1. **버전** 드롭다운 목록에서 **5.42.0**을 선택합니다.
1. **설치**를 선택합니다.
1. **라이선스 동의** 대화 상자에서 **동의함**을 선택하여 SDK를 설치합니다.

패키지를 설치한 후 Microsoft Graph SDK용 토큰 공급자를 만듭니다.

1. **검색** 폴더에서 **TokenProvider.cs**라는 새 파일을 만듭니다.
1.  파일에서 다음 코드를 추가합니다.

    ```csharp
    using Microsoft.Kiota.Abstractions.Authentication;
    
    namespace MsgExtProductSupport.Search
    {
       public class TokenProvider : IAccessTokenProvider
        {
            public string Token { get; set; }
            public AllowedHostsValidator AllowedHostsValidator => throw new NotImplementedException();
    
            public Task<string> GetAuthorizationTokenAsync(Uri uri, Dictionary<string, object>? additionalAuthenticationContext = null, CancellationToken cancellationToken = default)
            {
                return Task.FromResult(Token);
            }
        }
    }
    ```

1. 변경 내용을 저장합니다.

이제 새 메서드를 만들어 **GraphServiceClient** 인스턴스를 만듭니다.

1. **검색** 폴더에서 **SearchApp.cs** 파일을 엽니다.
1. 파일에서 필요한 네임스페이스를 가져옵니다.

    ```csharp
    using Microsoft.Graph;
    using Microsoft.Kiota.Abstractions.Authentication;
    ```

1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 새 Graph 클라이언트를 만드는 코드를 추가합니다. 이 코드는 Microsoft Graph에 요청을 보내는 데 사용합니다.

    ```csharp
    var graphClient = CreateGraphClient(tokenResponse);
    ```

1. **SearchApp** 클래스에서 **CreateGraphClient** 메서드를 구현합니다.

    ```csharp
    private static GraphServiceClient CreateGraphClient(TokenResponse tokenResponse)
    {
      TokenProvider provider = new() { Token = tokenResponse.Token };
      var authenticationProvider = new BaseBearerTokenAuthenticationProvider(provider);
      var graphClient = new GraphServiceClient(authenticationProvider);
      return graphClient;
    }
    ```

1. 변경 내용을 저장합니다.

이 코드는 인증 공급자를 설정하고 제공된 액세스 토큰을 사용하여 Microsoft Graph API와 상호 작용하는 데 사용할 수 있는 클라이언트 개체를 만듭니다.

## 작업 7 - 제품 목록 쿼리

제품 목록의 항목을 쿼리하고 나중에 검색 결과를 만들려면 GraphServiceClient를 사용하여 SharePoint Online에서 제품 데이터를 가져오기 위한 요청을 보냅니다.

Visual Studio에서 다음을 테스트합니다.

**OnTeamsMessagingExtensionQueryAsync** 메서드에서 SharePoint 데이터를 가져오기 위한 코드를 추가합니다.

  ```csharp
  var site = await GetSharePointSite(graphClient, spoHostname, spoSiteUrl, cancellationToken);
  var drive = await GetSharePointDrive(graphClient, site.SharepointIds.SiteId, "Product Imagery", cancellationToken);
  var items = await GetProducts(graphClient, site.SharepointIds.SiteId, filterQuery, cancellationToken);
  ```

이 코드는 다음 작업을 수행합니다.

- **제품 마케팅 사이트를 가져옵니다**. 사이트 개체에는 사이트의 개체를 반환하고 쿼리하는 데 사용되는 SharePoint 사이트의 ID가 포함됩니다.
- **제품 이미지 드라이브를 가져옵니다**. 드라이브는 제품 이미지가 포함된 문서 라이브러리를 나타냅니다. 나중에 드라이브를 사용하여 검색 결과에 표시되는 제품 이미지를 가져옵니다.
- **사용자 쿼리를 가져옵니다**. 이 쿼리는 사용자 쿼리를 기반으로 제품 목록을 쿼리하는 데 사용됩니다

**SearchApp** 클래스에서 세 가지 메서드를 구현합니다.

- **GetSharePointSite** 메서드 구현

    ```csharp
    private static async Task<Site> GetSharePointSite(GraphServiceClient graphClient, string hostName, string siteUrl, CancellationToken cancellationToken)
    {
        return await graphClient.Sites[$"{hostName}:/{siteUrl}"].GetAsync(r => r.QueryParameters.Select = new string[] { "sharePointIds" }, cancellationToken);
    }
    ```

이 메서드는 GraphServiceClient를 사용하여 경로를 통해 사이트 개체를 반환하는 요청을 Microsoft Graph에 보냅니다. 이 경로는 SharePoint Online 호스트 이름과 사이트 URL을 결합하여 만들어집니다. sharePointIds 속성 값만 필요하므로 Select 쿼리 매개 변수는 응답에서만 이 속성을 반환하도록 구성됩니다.

- **GetSharePointDrive** 메서드 구현

    ```csharp
    private static async Task<Drive> GetSharePointDrive(GraphServiceClient graphClient, string siteId, string name, CancellationToken cancellationToken)
    {
        var drives = await graphClient.Sites[siteId].Drives.GetAsync(r => r.QueryParameters.Select = new string[] { "id", "name" }, cancellationToken);
        var drive = drives.Value.Find(d => d.Name == name);
        return drive;
    }
    ```

이 메서드는 GraphServiceClient 및 사이트 ID를 사용하여 사이트에서 문서 라이브러리 컬렉션을 반환하고 각 문서 라이브러리의 ID 및 이름 속성을 반환합니다. 그런 다음 라이브러리 컬렉션은 이름 메서드 매개 변수와 이름이 같은 드라이브를 반환하도록 필터링됩니다.

- **GetProducts** 메서드 구현

    ```csharp
    private static async Task<SiteCollectionResponse> GetProducts(GraphServiceClient graphClient, string siteId, string filterQuery, CancellationToken cancellationToken)
    {
        var fields = new string[]
        {
            "fields/Id",
            "fields/Title",
            "fields/RetailCategory",
            "fields/PhotoSubmission",
            "fields/CustomerRating",
            "fields/ReleaseDate"
        };
    
        var request = graphClient.Sites.WithUrl($"https://graph.microsoft.com/v1.0/sites/{siteId}/lists/Products/items?expand={string.Join(",", fields)}&$filter={filterQuery}");
        return await request.GetAsync(null, cancellationToken);
    }
    ```

이 메서드는 GraphServiceClient를 사용하여 전달된 필터 쿼리로 제품 목록에서 필터링된 목록 항목을 반환하고 필드 배열에 정의된 목록 항목 데이터를 반환합니다.

## 작업 8 - 검색 결과 만들기

SharePoint에서 제품을 가져온 후 사용자에게 반환될 검색 결과를 만듭니다.

검색 결과 만들기는 항목 배열을 반복하는 것으로 구성되며, 각 항목에 대해 미리 보기 및 콘텐츠 카드가 포함된 MessagingExtensionAttachment를 만듭니다.

항목을 반복하기 전에 루프에서 사용할 적응형 카드 템플릿을 만듭니다.

Visual Studio에서 다음을 테스트합니다.

1. **Resources** 폴더에서 **Product.json**이라는 새 파일을 만듭니다.

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.6",
      "body": [
        {
          "type": "TextBlock",
          "text": "${Product.Title}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${Product.RetailCategory}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${ProductImage}",
              "altText": "${Product.Title}"
            }
          ],
          "minHeight": "350px",
          "verticalContentAlignment": "Center",
          "horizontalAlignment": "Center"
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "Call Volume",
              "value": "${formatNumber(Product.CustomerRating,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(Product.ReleaseDate,'dd/MM/yyyy')}"
            }
          ]
        },
        {
          "type": "ActionSet",
          "actions": [
            {
              "type": "Action.OpenUrl",
              "title": "View",
              "url": "https://${SPOHostname}/${SPOSiteUrl}/Lists/Products/DispForm.aspx?ID=${Product.Id}"
            }
          ]
        }
      ]
    }
    ```

템플릿은 데이터 바인딩 식을 사용하며, 이 식은 적응형 카드를 렌더링할 때 실제 값으로 대체됩니다. 카드가 렌더링되면 일부 제품 정보, 제품 이미지 및 작업 단추가 포함됩니다. 실행 단추는 제품 항목에 대한 SharePoint 목록 표시 양식으로 이동하는 브라우저를 엽니다.

다음으로 JSON 파일을 적응형 카드 템플릿으로 변환하는 코드를 추가합니다.

1. **검색** 폴더에서 **SearchApp.cs**를 엽니다.
1. 파일에서 필요한 네임스페이스를 가져옵니다.

    ```csharp
    using AdaptiveCards.Templating;
    ```

1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 JSON 파일의 내용을 읽고 새 **AdaptiveCardTemplate** 개체를 만드는 코드를 추가합니다.

    ```csharp
    var card = File.ReadAllText(@"Resources\Product.json");
    var template = new AdaptiveCardTemplate(card);
    ```

1. 변경 내용을 저장합니다.

다음으로 목록 항목을 반복하는 루프를 만듭니다. 각 반복은 다음과 같습니다.

- 제품 개체로 현재 항목 데이터 역직렬화
- 제품 이미지 미리 보기 가져오기
- 콘텐츠 카드 만들기
- 미리 보기 카드 만들기
- 콘텐츠와 미리 보기 카드를 결합한 MessagingExtensionAttachment 만들기
- 목록에 MessagingExtensionAttachment 추가

루프가 완료되면 사용자에게 반환할 수 있는 첨부 파일 목록이 생깁니다.

1. **검색** 폴더에서 **SearchApp.cs**를 엽니다.
1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 새 목록을 만드는 코드를 추가하여 **MessagingExtensionAttachment** 개체를 저장합니다.

    ```csharp
    var attachments = new List<MessagingExtensionAttachment>();
    ```

1. 목록 항목을 반복하는 foreach 루프를 만듭니다.

    ```csharp
    foreach (var item in items.Value) { 
            
    }
    ```

1. foreach 루프에 다음 코드를 추가합니다.

    ```csharp
    var product = JsonConvert.DeserializeObject<Product>(item.AdditionalData["fields"].ToString());
    product.Id = item.Id;
    
    var thumbnails = await GetThumbnails(graphClient, drive.Id, product.PhotoSubmission, cancellationToken);
    
    var resultCard = template.Expand(new
    {
      Product = product,
      ProductImage = thumbnails.Large.Url,
      SPOHostname = spoHostname,
      SPOSiteUrl = spoSiteUrl,
    });
    
    var previewcard = new ThumbnailCard
    {
      Title = product.Title,
      Subtitle = product.RetailCategory,
      Images = new List<CardImage> { new() { Url = thumbnails.Small.Url } }
    }.ToAttachment();
    
    var attachment = new MessagingExtensionAttachment
    {
      Content = JsonConvert.DeserializeObject(resultCard),
      ContentType = AdaptiveCard.ContentType,
      Preview = previewcard
    };
    
    attachments.Add(attachment);
    ```

1. 변경 내용을 저장합니다.

목록 항목 데이터를 강력하게 입력하려면 **제품**을 나타내는 모델을 만듭니다.

1. 프로젝트 루트 폴더에 **Models**라는 새 폴더를 만듭니다.
1. **Models** 폴더에 **Product.cs**라는 새 파일을 만듭니다.

    ```csharp
    namespace MsgExtProductSupport.Models
    {
        public class Product
        {
            public string Title { get; set; }
            public string RetailCategory { get; set; }
            public Link Specguide { get; set; }
            public string PhotoSubmission { get; set; }
            public double CustomerRating { get; set; }
            public DateTime ReleaseDate { get; set; }
            public string Id { get; set; }
            public string ContentType { get; set; }
            public DateTime Modified { get; set; }
            public DateTime Created { get; set; }
        }
    
        public class Link
        {
            public string Description { get; set; }
            public string Url { get; set; }
        }
    }
    ```

1. 변경 내용을 저장합니다.

다음으로 **GetThumbnails** 메서드를 구현하여 제품에 대한 Microsoft Graph에서 썸네일 이미지를 검색합니다.

1. **검색** 폴더에서 **SearchApp.cs** 파일을 엽니다.
1. **SearchApp** 클래스에서 **GetThumbnails** 메서드를 만듭니다.

    ```csharp
    private static async Task<ThumbnailSet> GetThumbnails(GraphServiceClient graphClient, string driveId, string photoUrl, CancellationToken cancellationToken)
    {
        var fileName = photoUrl.Split('/').Last();
        var driveItem = await graphClient.Drives[driveId].Root.ItemWithPath(fileName).GetAsync(null, cancellationToken);
        var thumbnails = await graphClient.Drives[driveId].Items[driveItem.Id].Thumbnails["0"].GetAsync(r => r.QueryParameters.Select = new string[] { "small", "large" }, cancellationToken);
        return thumbnails;
    }
    ```

1. 변경 내용을 저장합니다.

**GetThumbnails** 메서드는 Microsoft Graph API의 썸네일 엔드포인트를 사용하여 SharePoint에 저장된 제품 이미지의 크고 작은 썸네일을 반환합니다.

## 작업 9 - 검색 결과 반환

이제 MessagingExtensionResult 개체 컬렉션이 있으므로 사용자에게 검색 결과로 다시 반환할 수 있습니다.

- **OnTeamsMessagingExtensionQueryAsync** 메서드에서 검색 결과를 메시지 확장 응답으로 반환하는 코드를 추가합니다.

    ```csharp
    return new MessagingExtensionResponse
    {
      ComposeExtension = new MessagingExtensionResult
      {
        Type = "result",
        AttachmentLayout = "list",
        Attachments = attachments
      }
    };
    ```

## 작업 10 - 리소스 프로비전

Teams 앱 종속성 준비 프로세스를 실행하여 리소스를 프로비전합니다.

Visual Studio에서 다음을 테스트합니다.

1. **솔루션 탐색기**에서 **MsgExtProductSupport** 프로젝트를 마우스 오른쪽 단추로 클릭합니다.
1. **Teams Toolkit** 메뉴를 확장하고 **Teams 앱 종속성 준비**를 선택합니다.
1. **Microsoft 365 계정** 대화 상자에서 **계속**을 선택합니다.
1. **프로비전** 대화 상자에서 **프로비전**을 선택합니다.
1. **Teams Toolkit 경고** 대화 상자에서 **프로비전**을 선택합니다.
1. **Teams Toolkit 정보** 대화 상자에서 프롬프트를 **닫습니다**.

## 작업 11 - 실행 및 디버그

이제 웹 서비스를 시작하고 Microsoft Teams에서 메시지 확장을 테스트합니다.

Visual Studio에서 다음을 테스트합니다.

1. **F5** 키를 눌러 디버깅 세션을 시작하고 Microsoft Teams 웹 클라이언트로 이동하는 새 브라우저 창을 엽니다.
1. Microsoft 365 계정 자격 증명을 입력하고 Microsoft Teams로 계속 진행합니다.
1. 앱 설치 대화 상자에서 **추가**를 선택합니다.
1. 신규 또는 기존 Microsoft Teams 채팅을 엽니다.
1. 메시지 작성 영역에서 **...** 를 선택하여 앱 플라이아웃을 엽니다.
1. 앱 목록에서 **Contoso 제품**을 선택하여 메시지 확장을 엽니다.
1. 텍스트 상자에 **Mark8**을 입력합니다. **Mark8** 및 **Mark8 컨트롤러**라는 두 가지 결과가 표시됩니다.
1. 메시지 작성 상자에 카드를 포함하려면 **Mark8**을 선택합니다.
1. 카드가 포함된 **메시지 보내기**
1. 제출된 카드에서 **보기** 버튼을 선택하여 새 탭의 제품 목록에서 제품의 SharePoint 목록 항목을 봅니다.

디버깅 세션을 중지하는 브라우저입니다.

[ 다음 연습으로 계속...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)