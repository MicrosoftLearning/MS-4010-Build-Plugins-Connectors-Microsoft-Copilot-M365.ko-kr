---
lab:
  title: 연습 4 - Microsoft 365용 Copilot에서 사용할 메시지 확장 및 최적화
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 연습 4 - Microsoft 365용 Copilot에서 사용할 메시지 확장 및 최적화

이 연습에서는 Microsoft 365용 Copilot에서 사용할 메시지 확장을 확장하고 최적화합니다. 대상 그룹이라는 새 매개 변수를 추가하고 여러 매개 변수를 처리하도록 메시지 확장 논리를 업데이트합니다. 마지막으로, 메시지 확장을 실행 및 디버그하고 Microsoft Teams의 Copilot에서 테스트합니다.

## 작업 1 - 앱 매니페스트 업데이트

앱 매니페스트에 간결하고 정확한 설명을 지정하는 것은 Copilot이 플러그 인을 언제 어떠한 방법으로 호출할지 알 수 있게 하는 데 매우 중요합니다. 앱 매니페스트에서 앱, 명령 및 매개 변수 설명을 업데이트합니다.

Visual Studio를 엽니다:

1. **appPackage** 폴더에서 **manifest.json** 파일을 엽니다.
1. **설명** 개체 업데이트

    ```json
    {
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
        },
    }
    ```

    명령에 다른 매개 변수를 추가할 예정이므로 새 매개 변수를 포함하도록 명령 설명도 업데이트합니다.

1. **명령** 배열에서 명령 **설명**을 업데이트합니다.

    ```json
    {
        "commands": [
            {
                "id": "Search",
                "type": "query",
                "title": "Products",
                "description": "Find products by name or by target audience",
                "initialRun": true,
                "fetchTask": false,
                "context": [...],
                "parameters": [...]
            }
        ]
    }
    ```

    이제 Copilot에서 사용할 수 있는 새 매개 변수를 추가합니다. 이 새로운 매개 변수는 사용자가 개인 및 기업과 같은 다양한 대상 그룹을 대상으로 하는 Copilot를 사용하여 제품을 조회하는 데 도움이 됩니다.

1. **매개 변수** 배열에서 **ProductName** 매개 변수 뒤에 **TargetAudience** 매개 변수를 추가합니다.

    ```json
    {    
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
    }
    ```

1. 변경 내용을 저장합니다.

**TargetAudience** 매개 변수에 대한 설명은 해당 매개 변수가 무엇인지 설명하고 매개 변수는 **소비자** 또는 **엔터프라이즈**에 허용되는 값이어야 한다고 설명합니다.

## 작업 2 - 메시지 확장 논리 업데이트

새 매개 변수를 지원하고 복잡한 프롬프트를 지원하려면 봇 작업 처리기에서 OnTeamsMessagingExtensionQueryAsync 메서드를 업데이트하여 여러 매개 변수를 처리합니다.

사용자가 "Mark8이라는 이름을 가진 개인을 대상으로 하는 Contoso 제품 찾기"라는 프롬프트를 입력한다고 가정합니다. 매개 변수 설명에 따라 '개인을 대상으로'가 **소비자**로 변환되고 **TargetAudience** 매개 변수의 값으로 전달됩니다. 'Mark8'은 **ProductName** 매개 변수의 값으로 전달됩니다.

Visual Studio에서 다음을 테스트합니다.

1. **검색 ** 폴더에서 **SearchApp.cs** 파일을 엽니다.
1. **OnTeamsMessagingExtensionQueryAsync** 메서드에서 아래 코드 블록을 찾습니다.

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters); 
    ```

1. 코드 블록을 업데이트하여 **TargetAudience** 매개 변수의 값을 얻고 SharePoint Online 목록을 쿼리할 때 사용할 필터 쿼리를 만듭니다.

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var retailCategory = GetQueryData(query.Parameters, "TargetAudience");
    
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var retailCategoryFilter = !string.IsNullOrEmpty(retailCategory) ? $"fields/RetailCategory eq '{retailCategory}'" : string.Empty;
    var filters = new List<string> { nameFilter };
    filters.RemoveAll(f => string.IsNullOrEmpty(f));
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. 변경 내용을 저장합니다.

## 작업 3 - 리소스 프로비전

Teams 앱 종속성 준비 프로세스를 실행하여 리소스를 프로비전합니다.

Visual Studio에서 다음을 테스트합니다.

1. **솔루션 탐색기**에서 **MsgExtProductSupport** 프로젝트를 마우스 오른쪽 단추로 클릭합니다.
1. **Teams Toolkit** 메뉴를 확장하고 **Teams 앱 종속성 준비**를 선택합니다.
1. **Microsoft 365 계정** 대화 상자에서 **계속**을 선택합니다.
1. **프로비전** 대화 상자에서 **프로비전**을 선택합니다.
1. **Teams Toolkit 경고** 대화 상자에서 **프로비전**을 선택합니다.
1. **Teams Toolkit 정보** 대화 상자에서 프롬프트를 **닫습니다**.

## 작업 4 - 실행 및 디버그

이제 웹 서비스를 시작하고 Microsoft 365용 Copilot에서 메시지 확장을 테스트합니다.

1. **F5** 키를 눌러 디버깅 세션을 시작하고 Microsoft Teams 웹 클라이언트로 이동하는 새 브라우저 창을 엽니다.
1. Microsoft 365 계정 자격 증명을 입력하고 Microsoft Teams로 계속 진행합니다.
1. 앱 설치 대화 상자에서 **추가**를 선택합니다.
1. Microsoft Teams에서 **Copilot** 앱을 엽니다.
1. 메시지 작성 영역에서 **플러그 인** 플라이아웃을 엽니다.
1. 플러그 인 목록에서 **Contoso 제품** 플러그 인을 토글하여 사용하도록 설정합니다.
1. **개인을 대상으로 하는 Contoso 제품 찾기**를 메시지로 입력하고 보냅니다.
1. Copilot 응답에서 로그인 버튼이 반환되면 **로그인** 버튼을 선택하여 인증합니다.
1. 인증에 성공하면 **개인을 대상으로 하는 Contoso 제품 찾기를 메시지로 입력**하고 보냅니다.
1. Copilot 응답에서 플러그 인 응답에 반환된 데이터가 표시되고 플러그 인이 응답에서 참조됩니다.
1. 결과와 관련된 적응형 카드를 보려면 Copilot 응답의 참조를 마우스로 가리킵니다.

디버깅 세션을 중지하는 브라우저입니다.

[랩 요약으로 계속...](./6-summary.md)