---
lab:
  title: 연습 3 - Microsoft Entra 보호 API에서 제품 데이터 반환
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 연습 3 - Microsoft Entra 보호 API에서 제품 데이터 반환

이 연습에서는 사용자 지정 API에서 데이터를 검색하도록 메시지 확장을 업데이트합니다. 사용자 쿼리를 기반으로 사용자 지정 API에서 데이터를 가져와서 검색 결과의 데이터를 사용자에게 반환합니다.

![Microsoft Teams의 검색 기반 메시지 확장 프로그램에서 반환된 검색 결과의 스크린샷](../media/3-search-results-api.png)

### 연습 기간

  - **예상 완료 시간**: 50분

## 작업 1 - 개발 프록시 설치 및 구성

이 연습에서는 API를 시뮬레이션할 수 있는 명령줄 도구인 개발 프록시를 사용합니다. 실제 API를 만들지 않고도 앱을 테스트하려는 경우에 유용합니다.

이 연습을 완료하려면 [최신 버전의 개발 프록시](/microsoft-cloud/dev/dev-proxy/get-started)를 설치하고 이 모듈에 대한 개발 프록시 사전 설정을 다운로드해야 합니다.

사전 설정은 Microsoft Entra로 보호되는 메모리 내 데이터 저장소를 사용하여 CRUD(만들기, 읽기, 업데이트, 삭제) API를 시뮬레이션합니다. 즉, 인증이 필요한 실제 API를 호출하는 것처럼 앱을 테스트할 수 있습니다.

1. 개발 프록시를 설치하려면 **관리자 권한으로 명령 프롬프트 창**을 새로 엽니다.

    ```bash
    winget install Microsoft.DevProxy --silent
    ```

1. 사전 설정을 다운로드하려면 다음 명령을 사용합니다.

    ```bash
    devproxy preset get learn-copilot-me-plugin
    ```

1. 나중에 사용할 수 있도록 명령 프롬프트 창을 열어 둡니다.

## 작업 4 - 사용자 쿼리 값 가져오기

매개 변수 이름으로 사용자 쿼리 값을 가져오는 메서드를 만듭니다.

Visual Studio 및 **ProductsPlugin** 프로젝트에서 다음을 수행합니다.

1. **Helpers** 폴더에서 **MessageExtensionHelpers.cs**라는 새 파일을 만듭니다.

1. 파일에서 코드를 다음과 같이 바꿉니다.

   ```csharp
   using Microsoft.Bot.Schema.Teams;
   internal class MessageExtensionHelpers
   {
       internal static string GetQueryParameterValueByName(IList<MessagingExtensionParameter> parameters, string name) => parameters.FirstOrDefault(p => p.Name == name)?.Value as string ?? string.Empty;
   }
   ```

1. 변경 내용을 저장합니다.

다음으로, SearchApp 클래스에서 **OnTeamsMessagingExtensionQueryAsync** 메서드를 업데이트하여 새 도우미 메서드를 사용합니다.

1. **검색** 폴더에서 **SearchApp.cs**를 엽니다.

1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 다음 코드를 바꿉니다.

   ```csharp
   var text = query?.Parameters?[0]?.Value as string ?? string.Empty;
   ```

   다음과 같이 바꿉니다.

   ```csharp
   var text = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
   ```

1. 커서를 **텍스트** 변수로 이동하고, `Ctrl + R`, `Ctrl + R`을 사용하여, 변수 이름을 **이름**으로 바꿉니다.

1. **Enter** 키를 눌러 3개의 파일에서 변수 이름을 바꿉니다.

1. 변경 내용을 저장합니다.

이제 **OnTeamsMessagingExtensionQueryAsync** 메서드는 다음과 같습니다.

```csharp
protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
{
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
    var template = new AdaptiveCardTemplate(card);
    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "result",
            AttachmentLayout = "list",
            Attachments = [
                new MessagingExtensionAttachment
                    {
                        ContentType = AdaptiveCard.ContentType,
                        Content = JsonConvert.DeserializeObject(template.Expand(new { title = name })),
                        Preview = new ThumbnailCard { Title = name }.ToAttachment()
                    }
            ]
        }
    };
}
```

## 작업 3 - 사용자 지정 API에서 데이터 가져오기

사용자 지정 API에서 데이터를 가져오려면 요청의 인증 헤더에 액세스 토큰을 보내고 제품 데이터를 나타내는 모델로 응답을 역직렬화해야 합니다.

먼저 사용자 지정 API에서 반환되는 제품 데이터를 나타내는 모델을 만듭니다.

Visual Studio 및 **ProductsPlugin** 프로젝트에서 다음을 수행합니다.

1. **Models**라는 폴더를 만듭니다.

1. **Models** 폴더에 **Product.cs**라는 새 파일을 만듭니다.

1. 파일에서 기존 코드를 다음으로 바꿉니다.

   ```csharp
   using System.Text.Json.Serialization;
   internal class Product
   {
       [JsonPropertyName("productId")]
       public int Id { get; set; }
       [JsonPropertyName("imageUrl")]
       public string ImageUrl { get; set; }
       [JsonPropertyName("name")]
       public string Name { get; set; }
       [JsonPropertyName("category")]
       public string Category { get; set; }
       [JsonPropertyName("callVolume")]
       public int CallVolume { get; set; }
       [JsonPropertyName("releaseDate")]
       public string ReleaseDate { get; set; }
   }
   ```

1. 변경 내용을 저장합니다.

다음으로, 사용자 지정 API에서 제품 데이터를 검색하는 서비스 클래스를 만듭니다.

Visual Studio 및 **ProductsPlugin** 프로젝트에서 다음을 수행합니다.

1. **Services**라는 폴더를 만듭니다.

1. **Services** 폴더에서 **TokenProvider.cs**라는 새 파일을 만듭니다.

1. 파일에서 기존 코드를 다음으로 바꿉니다.

    ```csharp
    using System.Net.Http.Headers;
    internal class ProductsService
    {
        private readonly HttpClient _httpClient;
        private readonly string _baseUri = "https://api.contoso.com/v1/";
        internal ProductsService(string token)
        {
            _httpClient = new HttpClient();
            _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
        }
        internal async Task<Product[]> GetProductsByNameAsync(string name)
        {
            var response = await _httpClient.GetAsync($"{_baseUri}products?name={name}");
            response.EnsureSuccessStatusCode();
            var jsonString = await response.Content.ReadAsStringAsync();
            return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
        }
    }
    ```

1. 변경 내용을 저장합니다.

**ProductsService** 클래스에는 사용자 지정 API에서 제품 데이터를 가져오는 메서드가 포함되어 있습니다. 클래스 생성자는 액세스 토큰을 매개 변수로 사용하고 인증 헤더의 액세스 토큰을 통해 **HttpClient** 인스턴스를 설정합니다.

다음으로, **ProductsService** 클래스를 사용하여 사용자 지정 API에서 제품 데이터를 가져오기 위해 **OnTeamsMessagingExtensionQueryAsync** 메서드를 업데이트합니다.

1. **검색** 폴더에서 **SearchApp.cs**를 엽니다.

1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 **이름** 변수 선언 다음에 다음 코드를 추가하여 사용자 지정 API에서 제품 데이터를 가져옵니다.

   ```csharp
   var productService = new ProductsService(tokenResponse.Token);
   var products = await productService.GetProductsByNameAsync(name);
   ```

1. 변경 내용을 저장합니다.

## 작업 4 - 검색 결과 만들기

이제 제품 데이터가 있으므로 사용자에게 반환되는 검색 결과에 포함할 수 있습니다.

먼저 제품 정보를 표시하도록 기존 적응형 카드 템플릿을 업데이트해 보겠습니다.

Visual Studio 및 **ProductsPlugin** 프로젝트에서 계속 진행합니다.

1. **Resources** 폴더에서 **card.json** 이름을 **Product.json**으로 바꿉니다.

1. **Resources** 폴더에서 **Product.data.json**이라는 새 파일을 만듭니다. 이 파일에는 Visual Studio에서 적응형 카드 템플릿의 미리 보기를 생성하는 데 사용하는 예제 데이터가 포함되어 있습니다.

1. 파일에서 다음 JSON을 추가합니다.

    ```json
    {
      "callVolume": 36,
      "category": "Enterprise",
      "imageUrl": "https://raw.githubusercontent.com/SharePoint/sp-dev-provisioning-templates/master/tenant/productsupport/source/Product%20Imagery/Contoso4.png",
      "name": "Contoso Quad",
      "productId": 1,
      "releaseDate": "2019-02-09"
    }
    ```

1. 변경 내용을 저장합니다.

1. **Resources** 폴더에서 **Product.json**을 엽니다.

1. 파일에서 콘텐츠를 다음 JSON으로 바꿉니다.

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.5",
      "body": [
        {
          "type": "TextBlock",
          "text": "${name}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${category}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${imageUrl}",
              "altText": "${name}"
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
              "value": "${formatNumber(callVolume,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(releaseDate,'dd/MM/yyyy')}"
            }
          ]
        }
      ]
    }
    ```

1. 변경 내용을 저장합니다.

적응형 카드 템플릿은 바인딩 식을 사용하여 제품 정보를 표시합니다. **\$\{name\}**, **\$\{category\}**, **\$\{imageUrl\}**, **\$\{callVolume\}** 및 **\$\{releaseDate\}** 식은 제품 데이터의 해당 값으로 바뀝니다. **formatNumber** 및 **formatDateTime** 템플릿 함수는 **callVolume** 및 **releaseDate** 값을 각각 숫자와 날짜로 형식화하는 데 사용됩니다.

잠시 시간을 내어 Visual Studio에서 적응형 카드 미리 보기를 살펴보세요. 미리 보기는 제품 데이터가 템플릿에 바인딩된 경우 적응형 카드 템플릿의 모양을 보여줍니다. **Product.data.json** 파일의 예제 데이터를 사용하여 미리 보기를 생성합니다.

그런 다음 적응형 카드 템플릿의 이미지를 Microsoft Teams에 표시할 수 있도록 **raw.githubusercontent.com** 도메인을 포함하는 앱 매니페스트의 **validDomains** 속성을 업데이트합니다.

**TeamsApp** 프로젝트에서 다음을 수행합니다.

1. **appPackage** 폴더에서 **manifest.json**을 엽니다.

1. 파일에서 **ValidDomains** 속성에 GitHub 도메인을 추가합니다 

    ```json
      "validDomains": [
        "token.botframework.com",
        "raw.githubusercontent.com",
        "${{BOT_DOMAIN}}"
      ],
    ```

1. 변경 내용을 저장합니다.

다음으로, **OnTeamsMessagingExtensionQueryAsync** 메서드를 업데이트하여 제품 정보가 포함된 첨부 파일 목록을 만듭니다.

**ProductsPlugin** 프로젝트에서 다음을 수행합니다.

1. **검색** 폴더에서 **SearchApp.cs**를 엽니다.

1. **card.json**을 **Product.json**으로 업데이트하여 파일 이름의 변경 사항을 반영합니다. 34줄에서 다음 코드를 바꿉니다.

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
   ```

   다음과 같이 바꿉니다.

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "Product.json"), cancellationToken);
   ```

1. **템플릿** 변수 선언 이후에 다음 코드를 추가하여 첨부 파일 목록을 만듭니다.

   ```csharp
    var attachments = products.Select(product =>
    {
        var content = template.Expand(product);
        return new MessagingExtensionAttachment
        {
            ContentType = AdaptiveCard.ContentType,
            Content = JsonConvert.DeserializeObject(content),
            Preview = new ThumbnailCard
            {
                Title = product.Name,
                Subtitle = product.Category,
                Images = [new() { Url = product.ImageUrl }]
            }.ToAttachment()
        };
    }).ToList();
   ```

1. **attachments** 변수를 포함하도록 **return** 문을 업데이트합니다.

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

1. 변경 내용 저장

## 작업 5 - 리소스 만들기 및 업데이트

이제 모든 항목이 준비되면 **Teams 앱 종속성 준비** 프로세스를 실행하여 새 리소스를 만들고 기존 리소스를 업데이트합니다.

Visual Studio에서 다음을 테스트합니다.

1. **솔루션 탐색기**에서 **TeamsApp** 프로젝트를 마우스 오른쪽 단추로 클릭합니다.

1. **Teams Toolkit** 메뉴를 확장하고 **Teams 앱 종속성 준비**를 선택합니다.

1. **Microsoft 365 계정** 대화 상자에서 **계속**을 선택합니다.

1. **프로비전** 대화 상자에서 **프로비전**을 선택합니다.

1. **Teams Toolkit 경고** 대화 상자에서 **프로비전**을 선택합니다.

1. **Teams 도구 키트 정보** 대화 상자에서 교차 아이콘을 선택하여 대화 상자를 닫습니다.

## 작업 6 - 실행 및 디버그

리소스가 프로비전되면 디버깅 세션을 시작하여 메시지 확장을 테스트합니다.

먼저 개발 프록시를 시작하여 사용자 지정 API를 시뮬레이션합니다.

1. 여전히 열려 있는 **명령 프롬프트 창**에서 다음 명령을 실행하여 개발 프록시를 시작합니다.

   ```bash
   devproxy --config-file "~appFolder/presets/learn-copilot-me-plugin/products-api-config.json"
   ```

1. 메시지가 표시되면 인증서 경고를 수락합니다.

> [!NOTE]
> 개발 프록시가 실행되면 시스템 수준의 프록시 역할을 합니다.

다음으로 Visual Studio 디버그 세션을 시작합니다.

1. 새 디버그 세션을 시작하려면 <kbd>F5</kbd> 키를 누르거나 도구 모음에서 **시작**을 선택합니다.

1. 브라우저 창이 열리고 앱 설치 대화 상자가 Microsoft Teams 웹 클라이언트에 나타날 때까지 기다립니다. 메시지가 표시되면 Microsoft 365 계정 자격 증명을 입력합니다.

1. 앱 설치 대화 상자에서 **추가**를 선택합니다.

1. 신규 또는 기존 Microsoft Teams 채팅을 엽니다.

1. 메시지 작성 영역에서 **/apps**를 입력하여 앱 선택기를 엽니다.

1. 앱 목록에서 **Contoso 제품**을 선택하여 메시지 확장을 엽니다.

1. 텍스트 상자에 **mark8**을 입력합니다. 검색 쿼리를 여러 번 입력해야 할 수 있습니다.

1. 검색이 완료되고 결과가 표시될 때까지 기다립니다.

    ![Microsoft Teams의 검색 기반 메시지 확장 프로그램에서 반환된 검색 결과의 스크린샷](../media/3-search-results-api.png)

1. 결과 목록에서 검색 결과를 선택하여 메시지 작성 상자에 카드를 포함합니다.

Visual Studio로 돌아가서 도구 모음에서 **중지**를 선택하거나 <kbd>Shift</kbd> + <kbd>F5</kbd> 키를 눌러 디버그 세션을 중지합니다. 또한 <kbd>Ctrl</kbd> + <kbd>C</kbd>를 사용하여 개발 프록시를 종료합니다.

[이 연습의 다음 단계를 계속 진행합니다...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)