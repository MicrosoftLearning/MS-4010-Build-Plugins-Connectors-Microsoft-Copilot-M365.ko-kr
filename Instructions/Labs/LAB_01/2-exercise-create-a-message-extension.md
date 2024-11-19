---
lab:
  title: 연습 1 - 메시지 확장 프로그램 만들기
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 연습 1 - 메시지 확장 프로그램 만들기

이 연습에서는 메시지 확장 솔루션을 만듭니다. Visual Studio에서 Teams 도구 키트를 사용하여 필요한 리소스를 만든 다음, 디버그 세션을 시작하고 Microsoft Teams에서 테스트합니다.

![Microsoft Teams의 검색 기반 메시지 확장 프로그램에서 반환된 검색 결과의 스크린샷](../media/1-search-results.png)

### 연습 기간

  - **예상 완료 시간**: 25분

## 작업 1 - Visual Studio용 Teams Toolkit를 사용하여 새 프로젝트 만들기

먼저 검색 명령이 포함된 메시지 확장으로 구성된 새 Microsoft Teams 앱 프로젝트를 만듭니다. Visual Studio용 Teams 도구 키트 프로젝트 템플릿을 사용하여 프로젝트를 만들 수 있지만 이 모듈을 완료하려면 스캐폴드된 프로젝트를 변경해야 합니다. 대신 NuGet 패키지로 사용할 수 있는 사용자 지정 프로젝트 템플릿을 사용합니다. 사용자 지정 템플릿을 사용하면 필요한 파일과 종속성이 있는 솔루션을 만들어 시간을 절약할 수 있다는 이점이 있습니다.

1. 관리자 권한으로 PowerShell 세션을 엽니다.

1. 다음을 실행하여 적절한 작업 디렉터리로 변경합니다.

    ```Powershell
    cd ~\Documents
    ```

1. 다음을 실행하여 NuGet에서 템플릿 패키지 설치를 시작합니다.

    ```PowerShell
    dotnet new install M365Advocacy.Teams.Templates
    ```

1. 다음을 실행하여 새 프로젝트를 만듭니다.

    ```PowerShell
    dotnet new teams-msgext-search --name "ProductsPlugin" `
      --internal-name "msgext-products" `
      --display-name "Contoso products" `
      --short-description "Product look up tool." `
      --full-description "Get real-time product information and share them in a conversation." `
      --command-id "Search" `
      --command-description "Find products by name" `
      --command-title "Products" `
      --parameter-name "ProductName" `
      --parameter-title "Product name" `
      --parameter-description "The name of the product as a keyword" `
      --allow-scripts Yes
    ```

1. 프로젝트가 만들어질 때까지 기다립니다.

1. `cd ProductsPlugin` 실행을 통해 프로젝트 디렉터리로 변경합니다.

1. `.\ProductsPlugin.sln` 실행을 통해 Visual Studio에서 솔루션을 엽니다.

1. 애플리케이션 선택 창에서 **Visual Studio 2022**를 선택한 다음, **항상**을 선택합니다.

## 작업 3 - 개발 터널 만들기

사용자가 메시지 확장과 상호 작용하면 Bot Service는 웹 서비스에 요청을 보냅니다. 개발하는 동안 웹 서비스는 컴퓨터에서 로컬로 실행됩니다. Bot Service가 웹 서비스에 도달할 수 있도록 하려면 개발 터널을 사용하여 컴퓨터 외에 노출해야 합니다.

![Visual Studio 개발 터널 창의 스크린샷](../media/14-select-dev-tunnel.png)

Visual Studio에서 다음을 테스트합니다.

1. 도구 모음에서 **시작** 단추 옆의 드롭다운을 선택하고, **개발 터널(활성 터널 없음)** 메뉴를 확장하여 **터널 만들기**를 선택합니다.

1. 대화 상자에서 다음 값을 지정합니다.

    1. **계정**: 제공된 Microsoft 365 계정을 사용하여 로그인합니다. **회사 또는 학교 계정**을 선택합니다.

    1. **이름**: msgext-products

    1. **터널 유형**: 임시

    1. **액세스:** 퍼블릭

1. **확인**을 선택하여 터널을 만듭니다. 새 터널이 현재 활성 터널임을 나타내는 프롬프트가 표시됩니다.

1. **확인**을 선택하여 프롬프트를 닫습니다.

## 작업 3 - 리소스 준비

이제 모든 항목이 준비되면 Teams 도구 키트를 통해 **Teams 앱 종속성 준비** 프로세스를 실행하여 필요한 리소스를 만듭니다.

![Visual Studio의 확장된 Teams 도구 키트 메뉴 스크린샷](../media/15-prepare-teams-app-dependencies.png)

Teams 앱 종속성 준비 프로세스는 활성 개발 터널 URL을 사용하여 .**TeamsApp\\env\\.env.local**파일의 **BOT_ENDPOINT** 및 **BOT_DOMAIN** 환경 변수를 업데이트하고 **TeamsApp\\teamsapp.local.yml** 파일에 설명된 작업을 실행합니다.

잠시 시간을 내어 **teamsapp.local.yml** 파일의 단계를 탐색합니다.

Visual Studio에서 다음을 테스트합니다.

1. **프로젝트** 메뉴를 열고(또는 솔루션 탐색기 **TeamsApp** 프로젝트를 마우스 오른쪽 단추로 선택하고), **Teams 도구 키트** 메뉴를 확장하고, **Teams 앱 종속성 준비**를 선택할 수 있습니다.

1. **Microsoft 365 계정** 대화 상자에서 로그인하거나 기존 계정을 선택하여 Microsoft 365 테넌트에 액세스한 다음 **계속**을 선택합니다.

1. **프로비전** 대화 상자에서 로그인하거나 Azure에 리소스를 배포하는 데 사용할 계정을 선택하고 다음 값을 지정합니다.

      1. **구독 이름**: 드롭다운을 사용하여 구독을 선택합니다.

      1. **리소스 그룹**: 드롭다운 목록에서 미리 채워진 리소스 그룹을 선택합니다.

1. **프로비전**을 선택하여 Azure에서 리소스를 만듭니다.

1. Teams 도구 키트 경고 프롬프트에서 **프로비전**을 선택합니다.

1. Teams 도구 키트 정보 프롬프트에서 **프로비전된 리소스 보기**를 선택하여 새 브라우저 창을 엽니다.

잠시 시간을 내어 Azure에서 만든 리소스를 탐색하고 **.env.local** 파일에서 만든 환경 변수도 확인합니다.

> [!NOTE]
> Visual Studio를 닫고 다시 열면 개발 터널 URL이 변경되고 더 이상 활성 터널로 선택되지 않습니다. 이 경우 터널을 다시 선택하고 **Teams 앱 종속성 준비** 프로세스를 실행하여 앱 매니페스트에서 업데이트된 URL을 반영해야 합니다.

## 작업 4 - 실행 및 디버그

Teams 도구 키트는 다중 프로젝트 시작 프로필을 사용합니다. 프로젝트를 실행하려면 Visual Studio에서 미리 보기 기능을 사용하도록 설정해야 합니다.

Visual Studio에서:

1. **도구** 메뉴에서 **옵션**을 선택합니다.

1. 검색 창에 **다중 프로젝트**를 입력합니다.

1. **환경** 아래에서 **미리 보기 기능**을 선택합니다.

1. **다중 프로젝트 시작 프로필 사용** 옆의 확인란을 선택하고 **확인**을 선택하여 변경 내용을 저장합니다.

디버그 세션을 시작하고 Microsoft Teams에서 앱을 설치하려면 다음을 수행합니다.

1. <kbd>F5</kbd> 키를 누르거나 도구 모음에서 **시작**을 선택합니다.

1. 앱을 처음 시작할 때 나타나는 SSL 인증 경고를 신뢰하거나 승인합니다.

1. 브라우저 창이 열리고 앱 설치 대화 상자가 Microsoft Teams 웹 클라이언트에 나타날 때까지 기다립니다. 메시지가 표시되면 Microsoft 365 계정 자격 증명을 입력합니다.

1. 앱 설치 대화 상자에서 **추가**를 선택합니다.

메시지 확장을 테스트하려면 다음을 수행합니다.

1. 새 채팅(<kbd>Alt+N</kbd>)을 열고 **Contoso**를 **To** 상자에 입력하고 **Contoso 제품 지원을** 선택합니다.

    > [!NOTE]
    > 사용자 계정으로 채팅하는 경우 작동하지 않습니다. 다른 사용자 또는 그룹이어야 합니다.

1. 메시지 작성 영역에서 **/apps**를 입력하여 앱 선택기를 엽니다.

1. 앱 목록에서 **Contoso 제품**을 선택하여 메시지 확장을 엽니다.

1. 텍스트 상자에 **hello**를 입력합니다. 검색을 여러 번 입력해야 할 수 있습니다.

1. 검색 결과가 나타날 때까지 기다립니다.

1. 결과 목록에서 **hello**를 선택하여 메시지 작성 상자에 카드를 포함합니다.

![Microsoft Teams의 검색 기반 메시지 확장 프로그램에서 반환된 검색 결과의 스크린샷](../media/1-search-results.png)

Visual Studio로 돌아가서 도구 모음에서 **중지**를 선택하거나 <kbd>Shift</kbd> + <kbd>F5</kbd> 키를 눌러 디버그 세션을 중지합니다.

[이 연습의 다음 단계를 계속 진행합니다...](./3-exercise-add-single-sign-on.md)