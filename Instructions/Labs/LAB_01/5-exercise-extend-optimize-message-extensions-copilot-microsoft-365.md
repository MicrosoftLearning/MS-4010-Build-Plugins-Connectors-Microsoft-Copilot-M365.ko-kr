---
lab:
  title: 연습 4 - Microsoft 365용 Copilot에서 사용할 메시지 확장 및 최적화
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 연습 4 - Microsoft 365용 Copilot에서 사용할 메시지 확장 및 최적화

이 연습에서는 Microsoft 365용 Copilot에서 사용할 메시지 확장을 확장하고 최적화합니다. 대상 그룹이라는 새 매개 변수를 추가하고 여러 매개 변수를 처리하도록 메시지 확장 논리를 업데이트합니다. 마지막으로, 메시지 확장을 실행 및 디버그하고 Microsoft Teams의 Copilot에서 테스트합니다.

![메시지 확장 플러그 인에서 반환된 정보를 포함하는 Microsoft 365용 Copilot의 답변 스크린샷입니다. 제품 정보를 보여 주는 적응형 카드가 표시됩니다.](../media/5-copilot-answer.png)

> [!NOTE]
> 이 연습에서 Microsoft 365 Copilot 라이선스가 필요한 작업은 작업 5뿐입니다. 테넌트에 Copilot 있는지 여부에 관계없이 이전 작업을 수행해야 합니다.

### 연습 기간

  - **예상 완료 시간**: 40분

## 작업 1 - 앱 설명 업데이트

앱 매니페스트에 간결하고 정확한 설명을 지정하는 것은 Copilot이 플러그 인을 언제 어떠한 방법으로 호출할지 알 수 있게 하는 데 매우 중요합니다. 앱 매니페스트에서 앱, 명령 및 매개 변수 설명을 업데이트합니다.

**TeamsApp** 프로젝트에서 Visual Studio를 엽니다.

1. **appPackage** 폴더에서 **manifest.json**을 엽니다.

1. **설명** 개체 업데이트

    ```json
    "description": {
        "short": "Product look up tool.",
        "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
    },
    ```

## 작업 2 - 새 매개 변수 추가

이제 Copilot에서 사용할 수 있는 새 매개 변수를 추가합니다. 이 새로운 매개 변수는 사용자가 개인 및 기업과 같은 다양한 대상 그룹을 지원하는 Copilot을 사용하여 제품을 조회하는 데 도움이 됩니다.

Visual Studio 및 **TeamsApp** 프로젝트에서 계속 진행합니다.

1. **manifest.json** 내의 **매개 변수** 배열에서 **ProductName** 매개 변수 뒤에 **TargetAudience** 매개 변수를 추가합니다.

    ```json
    "parameters": [
        {
            "name": "ProductName",
            "title": "Product name",
            "description": "The name of the product as a keyword",
            "inputType": "text"
        },
        {
            "name": "TargetAudience",
            "title": "Target audience",
            "description": "Audience that the product is aimed at. Consumer products are sold to individuals. Enterprise products are sold to businesses",
            "inputType": "text"
        }
    ]
    ```

1. 변경 내용을 저장합니다.

**TargetAudience** 매개 변수에 대한 설명은 해당 매개 변수가 무엇인지 설명하고 매개 변수는 **소비자** 또는 **엔터프라이즈**에 허용되는 값이어야 한다고 설명합니다.

다음으로 새 매개 변수를 포함하도록 명령 설명을 업데이트합니다.

1. **명령** 배열에서 **36줄**의 명령 **설명**을 업데이트합니다.

    ```json
    "description": "Find products by name or by target audience.",
    ```

## 작업 2 - 메시지 확장 논리 업데이트

새 매개 변수를 지원하고 복잡한 프롬프트를 지원하려면 봇 작업 처리기에서 **OnTeamsMessagingExtensionQueryAsync** 메서드를 업데이트하여 여러 매개 변수를 처리합니다.

먼저 이름 및 대상 그룹 매개 변수에 따라 제품을 검색하도록 **ProductService** 클래스를 업데이트합니다.

**ProductPlugin** 프로젝트의 Visual Studio에서 계속 진행합니다.

1. **Services** 폴더에서 **ProductsService.cs**를 엽니다.

1. 파일에서 **GetProductsByCategoryAsync** 및 **GetProductsByNameAndCategoryAsync**라는 새 메서드를 만듭니다.

    ```csharp
    internal async Task<Product[]> GetProductsByCategoryAsync(string category)
    {
        var response = await _httpClient.GetAsync($"{_baseUri}products?category={category}");
        response.EnsureSuccessStatusCode();
        var jsonString = await response.Content.ReadAsStringAsync();
        return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
    }
    internal async Task<Product[]> GetProductsByNameAndCategoryAsync(string name, string category)
    {
        var response = await _httpClient.GetAsync($"{_baseUri}?name={name}&category={category}");
        response.EnsureSuccessStatusCode();
        var jsonString = await response.Content.ReadAsStringAsync();
        return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
    }
    ```

1. 변경 내용을 저장합니다.

다음으로, **MessageExtensionHelper** 클래스에 새 메서드를 추가하여 이름 및 대상 그룹 매개 변수를 기반으로 제품을 검색합니다.

1. **Helpers** 폴더에서 **MessageExtensionHelper.cs**를 엽니다.

1. 파일에서 이름 및 대상 그룹 매개 변수에 따라 제품을 검색하는 **RetrieveProducts**라는 새 메서드를 만듭니다.

    ```csharp
    internal static async Task<IList<Product>> RetrieveProducts(string name, string audience, ProductsService productsService)
    {
        IList<Product> products;
        if (string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByCategoryAsync(audience);
        }
        else if (!string.IsNullOrEmpty(name) && string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByNameAsync(name);
        }
        else if (!string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByNameAndCategoryAsync(name, audience);
        }
        else
        {
            products = [];
        }
        return products;
    }
    ```

1. 변경 내용을 저장합니다.

**RetrieveProduct** 메서드는 이름 및 대상 그룹 매개 변수를 기반으로 제품을 검색합니다. 이름 매개 변수가 비어 있고 대상 그룹 매개 변수가 비어 있지 않으면 메서드는 대상 그룹 매개 변수를 기반으로 제품을 검색합니다. 이름 매개 변수가 비어 있지 않고 대상 그룹 매개 변수가 비어 있으면 메서드는 이름 매개 변수를 기반으로 제품을 검색합니다. 이름 및 대상 그룹 매개 변수가 모두 비어 있지 않으면 메서드는 두 매개 변수를 기반으로 제품을 검색합니다. 두 매개 변수가 모두 비어 있으면 메서드는 빈 목록을 반환합니다.

다음으로, **earchApp** 클래스를 업데이트하여 새 매개 변수를 처리합니다.

1. **검색** 폴더에서 **SearchApp.cs**를 엽니다.

1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 **30줄**부터 다음 코드를 바꿉니다.

    ```csharp
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var productService = new ProductsService(tokenResponse.Token);
    var products = await productService.GetProductsByNameAsync(name);
    ```

    다음으로 바꾸기:

    ```csharp
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var audience = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "TargetAudience");
    var productService = new ProductsService(tokenResponse.Token);
    var products = await MessageExtensionHelpers.RetrieveProducts(name, audience, productService);
    ```

1. 변경 내용을 저장합니다.

이제 **OnTeamsMessagingExtensionQueryAsync** 메서드는 쿼리 매개 변수에서 이름 및 대상 매개 변수를 검색합니다. 그런 다음 **RetrieveProducts** 메서드를 사용하여 이름 및 대상 그룹 매개 변수를 기반으로 제품을 검색합니다 

## 작업 4 - 리소스 만들기 및 업데이트

이제 모든 항목이 준비되면 **Teams 앱 종속성 준비** 프로세스를 실행하여 새 리소스를 만들고 기존 리소스를 업데이트합니다.

Visual Studio에서 다음을 테스트합니다.

1. **솔루션 탐색기**에서 **TeamsApp** 프로젝트를 마우스 오른쪽 단추로 클릭합니다.

1. **Teams Toolkit** 메뉴를 확장하고 **Teams 앱 종속성 준비**를 선택합니다.

1. **Microsoft 365 계정** 대화 상자에서 **계속**을 선택합니다.

1. **프로비전** 대화 상자에서 **프로비전**을 선택합니다.

1. **Teams Toolkit 경고** 대화 상자에서 **프로비전**을 선택합니다.

1. **Teams 도구 키트 정보** 대화 상자에서 교차 아이콘을 선택하여 대화 상자를 닫습니다.

## 작업 5 - 실행 및 디버그

리소스가 프로비전되면 디버깅 세션을 시작하여 메시지 확장을 테스트합니다.

먼저 **개발 프록시**를 시작하여 사용자 지정 API를 시뮬레이션합니다.

1. 여전히 열려 있는 **명령 프롬프트 창**에서 다음 명령을 실행하여 개발 프록시를 시작합니다.

1. 다음 명령을 실행하여 개발 프록시를 시작합니다.

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

1. Microsoft Teams에서 **Copilot** 앱을 엽니다.

1. 메시지 작성 영역에서 **플러그 인** 플라이아웃을 엽니다.

1. 플러그 인 목록에서 **Contoso 제품** 플러그 인을 토글하여 사용하도록 설정합니다.

    ![Contoso 제품 플러그 인이 활성화된 Microsoft Teams의 Microsoft 365용 Copilot 스크린샷](../media/20-copilot-plugin-enabled.png)

1. **개인을 대상으로 하는 Contoso 제품 찾기**를 메시지로 입력하고 보냅니다.

1. Copilot이 응답할 때까지 기다립니다.

    ![사용자의 요청을 처리하는 동안 표시되는 Copilot 메시지를 보여 주는 Microsoft Teams의 Microsoft 365용 Copilot 스크린샷](../media/21-copilot-thinking.png)

1. Copilot 응답에서 플러그 인 응답에 반환된 데이터가 표시되고 플러그 인이 응답에서 참조됩니다.

    ![메시지 확장 플러그 인에서 반환된 정보를 포함하는 Microsoft 365용 Copilot의 답변 스크린샷입니다. 제품 정보를 보여 주는 적응형 카드가 표시됩니다.](../media/5-copilot-answer.png)

1. 결과와 관련된 적응형 카드를 보려면 Copilot 응답의 참조에 마우스를 가져다 댑니다.

    ![제품 정보가 표시된 적응형 카드를 보여 주는 Microsoft Teams의 Microsoft 365용 Copilot 스크린샷 사용자가 Copilot 응답의 참조 위로 마우스를 가져가면 카드가 표시됩니다.](../media/22-copilot-reference.png)

Visual Studio로 돌아가서 도구 모음에서 **중지**를 선택하거나 <kbd>Shift</kbd> + <kbd>F5</kbd> 키를 눌러 디버그 세션을 중지합니다. 또한 <kbd>Ctrl</kbd> + <kbd>C</kbd>를 사용하여 개발 프록시를 종료합니다.

[랩 요약으로 계속 진행...](./6-summary.md)