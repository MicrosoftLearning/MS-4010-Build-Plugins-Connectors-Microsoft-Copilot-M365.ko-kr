---
lab:
  title: 연습 1 - 메시지 확장 프로그램 만들기
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 연습 1 - 메시지 확장 프로그램 만들기

이 연습에서는 검색 명령을 사용하여 메시지 확장  만듭니다. 먼저 Teams Toolkit 프로젝트 템플릿을 사용하여 프로젝트를 스캐폴드한 다음, 로컬 개발에 Azure AI Bot Service 리소스를 사용하 구성하도록 업데이트합니다. 개발자 터널을 만들어 봇 서비스와 로컬로 실행 중인 웹 서비스 간의 통신을 사용하도록 설정합니다. 그런 다음 필요한 리소스를 프로비전하도록 앱을 준비합니다. 마지막으로 메시지 확장을 실행 및 디버그하고 Microsoft Teams에서 테스트합니다.

![Microsoft Teams의 검색 기반 메시지 확장 프로그램에서 반환된 검색 결과의 스크린샷](../media/2-search-results-nuget.png)

## 작업 1 - Visual Studio용 Teams Toolkit를 사용하여 새 프로젝트 만들기

먼저 새 프로젝트를 만듭니다.

1. **Visual Studio 2022** 열기
1. **파일** 메뉴를 열고 ** 새 ** 메뉴를 확장하고 **새 프로젝트**를 선택합니다
1. 새 프로젝트 만들기 화면에서 **모든 플랫폼** 드롭다운을 확장한 다음, **Microsoft Teams**를 선택합니다. **다음**을 선택하여 작업을 계속할 수 있습니다.
1. 새 프로젝트 구성 대화 상자에서 다음을 수행합니다. 다음 값 중 하나를 지정합니다.
    1. **프로젝트 이름**: MsgExtProductSupport
    1. **위치**: 기본 위치 선택
1. **만들기**를 선택하여 프로젝트를 비계화합니다
1. 새 Teams 애플리케이션 만들기 대화 상자에서 ** 모든 앱 유형** 드롭다운을 확장한 다음, **메시지 확장 프로그램**을 선택합니다.
1. 템플릿 목록에서 **사용자 지정 검색 결과 **를 선택합니다
1. **만들기**를 선택하여 앱을 비계화합니다

## 작업 2 - Azure AI Bot Service 구성

Azure에서 리소스로 또는 dev.botframework.com 통해 봇 서비스 리소스를 만들 수 있습니다. 기본적으로 사용자 지정 검색 결과 템플릿은 dev.botframework.com 사용하여 봇을 등록합니다. 현재 dev.botframework.com 봇을 등록하는 것은 Microsoft 365용 Copilot와 호환되지 않습니다.

Microsoft 365용 Copilot를 지원하려면 Azure에서 Azure AI Bot Service 리소스를 프로비전하고 로컬 개발에 사용하도록 프로젝트를 업데이트합니다.

먼저 파일 전체에서 다시 사용하고 리소스를 프로비전할 때 사용할 수 있는 앱의 내부 이름을 중앙 집중화하는 환경 변수를 만들어 보겠습니다.

Visual Studio에서:

1. **env** 폴더에서 **.env.local**을 엽니다
1.  파일에서 다음 코드를 추가합니다.

    ```text
    APP_INTERNAL_NAME=msgext-product-support
    ```

1. 변경 내용을 저장합니다.

예를 들어 `${{APP_INTERNAL_NAME}}` 데이터 바인딩 식을 사용하면 Teams 도구 키트를 사용하여 리소스를 프로비전할 때 환경 변수 값을 파일에 삽입할 수 있습니다.

Azure AI Bot Service 리소스를 프로비전하려면 Microsoft Entra 앱 등록이 필요합니다. Teams 도구 키트에서 앱 등록을 프로비전하는 데 사용하는 앱 등록 매니페스트 파일을 만듭니다.

Visual Studio에서 다음을 테스트합니다.

1. **infra** 폴더에 **entra**라는 새 폴더를 만듭니다.
1. 폴더에서 이름이 **entra.bot.manifest.json**인 파일을 
1.  파일에서 다음 코드를 추가합니다.

    ```json
    {
      "id": "${{BOT_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{BOT_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}",
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
      "requiredResourceAccess": [],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": []
    }
    ```

1. 변경 내용을 저장합니다.

Teams 도구 키트는 Bicep 파일을 사용하여 Azure에서 리소스를 프로비전하고 구성합니다. 먼저 매개 변수 파일을 만듭니다. 매개 변수 파일은 환경 변수를 Bicep 템플릿에 전달하는 데 사용됩니다.

Visual Studio에서 다음을 테스트합니다.

1. ** infra** 폴더에서 **azure.parameters.local.json**이라는 새 파일을 만듭니다
1.  파일에서 다음 코드를 추가합니다.

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
        }
      }
    }
    ```

1. 변경 내용을 저장합니다.

이제 매개 변수 파일과 함께 사용되는 Bicep 파일을 만듭니다.

1. **infra** 폴더에서 **azure.local.bicep**이라는 이름으로 새 파일을 만듭니다.
1.  파일에서 다음 코드를 추가합니다.

    ```bicep
    @maxLength(20)
    @minLength(4)
    @description('Used to generate names for all resources in this file')
    param resourceBaseName string
    
    @description('Required when create Azure Bot service')
    param botEntraAppClientId string
    @maxLength(42)
    param botDisplayName string
    param botAppDomain string
    
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
      }
    }
    ```

1. 변경 내용을 저장합니다.

마지막 단계는 Teams 도구 키트 프로젝트 파일을 업데이트하는 것입니다. Bot Framework 작업을 통해 매니페스트 파일을 사용하여 봇 Microsoft Entra 앱 등록을 프로비전하고 Bicep 파일을 사용하여 Azure AI Bot Service 리소스를 프로비전하는 단계를 대체합니다.

Visual Studio에서 다음을 테스트합니다.

1. 프로젝트 루트 폴더에서 **teamsapp.local.yml**을 엽니다.
1. 파일에서 **botAadApp/create** 작업(줄 17~26) 을 사용하는 단계를 찾아 다음으로 바꿉니다.

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: BOT_ID
          clientSecret: SECRET_BOT_PASSWORD
          objectId: BOT_ENTRA_APP_OBJECT_ID
          tenantId: BOT_ENTRA_APP_TENANT_ID
          authority: BOT_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: BOT_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.bot.manifest.json"
          outputFilePath : "./build/entra.bot.manifest.${{TEAMSFX_ENV}}.json"
    
      - uses: arm/deploy
        with:
          subscriptionId: ${{AZURE_SUBSCRIPTION_ID}}
          resourceGroupName: ${{AZURE_RESOURCE_GROUP_NAME}}
          templates:
            - path: ./infra/azure.local.bicep
              parameters: ./infra/azure.parameters.local.json
              deploymentName: Create-resources-for-${{APP_INTERNAL_NAME}}-${{TEAMSFX_ENV}}
          bicepCliVersion: v0.9.1
    ```

1. 파일에서 **botFramework/create** 작업(줄 53~62)을 사용하는 단계를 제거합니다.
1. 변경 내용을 저장합니다.

앱 등록은 두 단계로 프로비전됩니다. 먼저 **aadApp/create** 작업은 클라이언트 암호를 사용하여 새 다중 테넌트 앱 등록을 만들어 **.env.local** 파일에 출력을 환경 변수로 작성합니다. 그런 다음 **aadApp/update** 작업은 **entra.bot.manifest.json** 파일을 사용하여 앱 등록을 업데이트합니다.

마지막 단계에서는 **arm/deploy** 작업을 사용하여 **azure.parameters.local.json** 파일 및 **azure.local.bicep** 파일을 사용하여 리소스 그룹에 Azure AI Bot Service 리소스를 프로비전합니다.

## 작업 3 - 개발 터널 만들기

사용자가 메시지 확장과 상호 작용하면 Bot Service는 웹 서비스에 요청을 보냅니다. 개발하는 동안 웹 서비스는 컴퓨터에서 로컬로 실행됩니다. Bot Service가 웹 서비스에 도달할 수 있도록 하려면 개발 터널을 사용하여 컴퓨터 외에 노출해야 합니다.

![Visual Studio의 확장된 개발 터널 메뉴 스크린샷](../media/18-select-dev-tunnel.png)

Visual Studio에서 다음을 테스트합니다.

1. 도구 모음에서 **MsgExtProductSupport**가 시작 프로젝트로 선택되어 있는지 확인하고 **Microsoft Teams(브라우저) 단추** 또는 **프로젝트 시작** 옆에 있는 드롭다운을 선택하여 디버그 프로필 메뉴를 확장합니다.
1. **개발 터널(활성 터널 없음)** 메뉴를 확장하고 **터널 만들기...** 를 선택합니다.
1. 대화 상자에서 다음 값을 지정합니다.
    1. **계정**: Microsoft 365 사용자 계정으로 로그인합니다.
    1. **이름**: MsgExtProductSupport
    1. **터널 유형**: 임시
    1. **액세스:** 퍼블릭
1. **확인**을 선택하여 터널을 만듭니다.
1. **확인**을 선택하여 프롬프트를 닫습니다.

## 작업 4 - 앱 매니페스트 업데이트

앱 매니페스트는 앱의 특징과 기능을 설명합니다. 앱 매니페스트의 속성을 업데이트하여 앱의 기능 및 특징을 더 잘 설명합니다.

먼저 앱 아이콘을 다운로드하여 프로젝트에 추가합니다.

![로컬 개발에 사용되는 색 아이콘입니다.](../media/app/color-local.png)

![원격 개발에 사용되는 색 아이콘입니다.](../media/app/color-dev.png)

1. **color-local.png** 및 **color-dev.png**를 다운로드합니다.
1. **appPackage** 폴더에서 **color-local.png** 및 **color-dev.png**를 추가합니다.
1. 폴더에서 이름이 **color.png**인 파일을 삭제합니다.

앱 이름이 프로젝트의 다른 위치에 복제되므로 이 값을 중앙에 저장할 새 환경 변수를 만듭니다.

Visual Studio에서 다음을 테스트합니다.

1. ** env** 폴더에서 ** .env.local**이라는 파일을 엽니다.
1.  파일에서 다음 코드를 추가합니다.

    ```text
    APP_DISPLAY_NAME=Contoso products
    ```

1. 변경 내용을 저장합니다.

마지막으로 앱 매니페스트 파일의 아이콘, 이름 및 설명 개체를 업데이트합니다.

1. **appPackage** 폴더에서 **manifest.json** 파일을 엽니다.
1. 파일에서 **아이콘**, **이름** 및 **설명** 개체를 다음(줄 13~24)으로 바꿉니다.

    ```json
        "icons": {
            "color": "color-${{TEAMSFX_ENV}}.png",
            "outline": "outline.png"
        },
        "name": {
            "short": "${{APP_DISPLAY_NAME}}",
            "full": "${{APP_DISPLAY_NAME}}"
        },
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation."
        },
    ```

1. 변경 내용을 저장합니다.

## 작업 6 - 리소스 프로비전

이제 모든 항목이 준비되면 Teams 도구 키트를 통해 Teams 앱 종속성 준비 프로세스를 실행하여 필요한 리소스를 프로비전합니다.

![Visual Studio의 확장된 Teams 도구 키트 메뉴 스크린샷](../media/19-prepare-teams-app-dependencies.png)

Teams 앱 종속성 준비 프로세스는 활성 개발 터널 URL을 사용하여 .env.local 파일의 **BOT_ENDPOINT** 및 **BOT_DOMAIN** 환경 변수를 업데이트하고 **teamsapp.local.yml** 파일에 설명된 작업을 실행합니다.

Visual Studio에서 다음을 테스트합니다.

1. 솔루션 탐색기에서 **TeamsApp**을 마우스 오른쪽 단추로 클릭합니다.
1. **Teams Toolkit** 메뉴를 확장하고 **Teams 앱 종속성 준비**를 선택합니다.
1. **Microsoft 365 계정** 대화 상자에서 개발자 테넌트에 대한 계정을 선택한 다음, **계속**을 선택합니다.
1. **프로비전** 대화 상자에서 Azure에 리소스를 배포하는 데 사용할 계정을 선택하고 다음 값을 지정합니다.
    1. **구독 이름**: 드롭다운을 사용하여 구독을 선택합니다.
    1. **리소스 그룹**: 드롭다운을 확장하고 사용자 계정에 대해 미리 만들어진 리소스 그룹을 선택합니다.
    1. **지역**: 드롭다운을 사용하여 가장 가까운 지역을 선택합니다.
1. **프로비전**을 선택하여 Azure에서 리소스를 프로비전합니다.
1. Teams 도구 키트 경고 프롬프트에서 **프로비전**을 선택합니다.
1. Teams 도구 키트 정보 프롬프트에서 **프로비전된 리소스 보기**를 선택하여 새 브라우저 창을 엽니다.

잠시 시간을 내어 Azure에서 만든 리소스를 살펴봅니다.

## 작업 7 - 실행 및 디버그  

이제 웹 서비스를 시작하고 메시지 확장을 테스트합니다. Teams 도구 키트를 사용하여 앱 매니페스트를 업로드하고 Microsoft Teams에서 메시지 확장을 테스트합니다.

Visual Studio에서 다음을 테스트합니다.

1. F5 키를 눌러 디버깅 세션을 시작하고 Microsoft Teams 웹 클라이언트를 탐색하는 새 브라우저 창을 엽니다.
1. 다른 SSL 인증서를 신뢰하라는 메시지가 표시되면 **예**를 선택하고 보안 경고에 대해 **예**를 다시 선택합니다. 인증서를 수락한 후 디버거를 다시 시작해야 할 수 있습니다.
1. 메시지가 표시되면 Microsoft 365 계정 자격 증명을 입력합니다.

  > [!IMPORTANT]
  > Microsoft Teams에 "이 앱을 찾을 수 없습니다"라는 메시지와 함께 대화 상자가 나타나면 아래 단계에 따라 앱 패키지를 수동으로 업로드합니다.
  >
  >  1. 대화 상자를 닫습니다.
  >  2. 사이드바에서 **앱**으로 이동합니다.
  >  3. 왼쪽 메뉴에서 **앱 관리**를 선택합니다.
  >  4. 명령 모음에서 **파일 업로드**를 선택합니다.
  >  5. 대화 상자에서 **사용자 지정된 앱 업로드**를 선택합니다.
  >  6. 파일 탐색기에서 솔루션 폴더로 이동하여 **appPackage\build** 폴더를 열고 **appPackage.local.zip**을 선택한 다음 **추가**를 선택합니다.

앱 설치를 계속합니다.

1. 앱 설치 대화 상자에서 **추가**를 선택합니다.
1. 신규 또는 기존 Microsoft Teams 채팅을 엽니다.
1. 메시지 작성 영역에서 **/apps**를 입력하여 플라이아웃을 엽니다.
1. 앱 목록에서 **Contoso 제품**을 선택하여 메시지 확장을 엽니다.
1. 텍스트 상자에 **Bot Builder**를 입력하여 검색을 시작합니다.
1. 결과 목록에서 결과를 선택하여 메시지 작성 상자에 카드를 포함합니다.

디버깅 세션을 중지하는 브라우저입니다.

[ 다음 연습으로 계속...](./3-exercise-add-single-sign-on.md)