---
lab:
  title: 개발 환경 준비
  module: 'LAB 03: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# 개발 환경 준비

먼저 개발 환경, 계정 및 소프트웨어를 준비해 보겠습니다. 시작하기 전에 다음 작업을 완료합니다.

## 작업 1: 필수 구성 요소 설치

> [!IMPORTANT]
> 이 프로젝트를 성공적으로 완료하려면 애플리케이션 업로드 권한이 있는 Microsoft 365 계정이 필요합니다. **연습 2**를 완료하려면 계정에 Microsoft 365용 Microsoft Copilot에 대한 라이선스도 있어야 합니다.

새 테넌트를 사용하는 경우 시작하기 전에 [https://office.com](https://office.com)에서 Microsoft 365 페이지에 [로그인](https://office.com)하는 것이 좋습니다. 테넌트가 구성된 방식에 따라 다단계 인증을 설정하라는 메시지가 표시될 수 있습니다. 계속하기 전에 Microsoft Teams 및 Microsoft Outlook에 액세스할 수 있는지 확인합니다.

다음 도구는 **MS-4010-DEVBOX**의 랩에 이미 설치되어 있습니다. 다음이 설치되어 있고 작동 가능한지 확인합니다.

1. [Visual Studio Code](https://code.visualstudio.com/)(최신 버전)

1. [Azure Storage Explorer](https://azure.microsoft.com/products/storage/storage-explorer/) - 이 샘플에 사용된 Northwind 데이터베이스를 보고 편집하려는 경우 다운로드합니다.

## 작업 2 - nvm-windows 설치

이 도구를 사용하여 Node.js를 설치하고 필요에 따라 프로젝트에 필요한 노드 버전을 전환합니다.

1. 웹 브라우저에서 [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases)으로 이동합니다.
2. 최신 릴리스 버전을 찾고 다운로드할 **nvm-setup.zip** 파일을 선택합니다.  파일은 로컬 머신에 다운로드됩니다.
3. 파일 폴더를 열고 zip 폴더의 내용을 컴퓨터의 폴더로 **추출** 합니다.
4. 새 폴더에서 **nvm-setup.exe**를 선택하여 설치 파일을 엽니다.
5. 설치 프로그램의 프롬프트에 따라 기본 옵션을 사용하여 도구를 설치합니다.
6. Nvm for Windows가 컴퓨터에 설치됩니다.

## 작업 3 - Node.js 설치

이 과정의 모든 솔루션과 호환되는 Node.js 버전 18.18.2를 설치합니다.

1. **명령 프롬프트** 애플리케이션을 엽니다.
2. `nvm install 18.18` 명령을 입력하여 Node.js를 설치합니다.
3. nvm 출력은 설치가 완료되었는지 확인해야 합니다.
4. 이 버전의 Node.js를 사용하려면 `nvm use 18.18` 명령을 실행합니다.
5. `node -v` 명령을 실행하여 버전 18.18.2가 설치되어 있는지 확인합니다.

이제 Node.js 버전 18.18.2를 설치하고 구성했습니다.

## 작업 4 - 샘플 코드 다운로드

샘플 리포지토리를 [복제](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples.git)하거나 [다운로드](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples.git)하세요. [https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/) 

복제되거나 다운로드된 리포지토리 내에서 **samples/msgext-northwind-inventory-ts** 폴더로 이동합니다. 이러한 랩은 작업할 위치이므로 이를 "**작업 폴더**"라고 합니다.

## 작업 3 - OneDrive에 샘플 문서 복사

샘플 애플리케이션에는 랩 중에 참조할 Copilot에 대한 일부 문서가 포함되어 있습니다. 이 작업에서는 Copilot에서 찾을 수 있도록 이러한 파일을 사용자의 OneDrive에 복사합니다. 테넌트를 설정하는 방법에 따라 이 프로세스의 일부로 다단계 인증을 설정하라는 메시지가 표시될 수 있습니다.

1. 브라우저를 열고 Microsoft 365([https://www.office.com/](https://www.office.com/))로 이동합니다. 랩 전체에서 사용할 Microsoft 365 계정을 사용하여 로그인합니다. 다단계 인증을 사용하라는 요청을 받을 수 있습니다.

1. 페이지 왼쪽 위 모서리에 있는 앱 메뉴 1️⃣를 사용하여 Microsoft 365 2️⃣내의 OneDrive 애플리케이션으로 이동합니다.

    ![Microsoft 365에서 OneDrive 애플리케이션으로 이동하는 스크린샷입니다.](../media/1-02-copy-sample-files-01.png)

1. OneDrive 내에서 **내 파일** 1️⃣로 이동합니다 문서 폴더가 있는 경우 해당 폴더로도 이동합니다. 아니면 **내 파일** 위치 내에서 바로 작업할 수 있습니다.

    ![OneDrive에서 문서로 이동하는 스크린샷입니다.](../media/1-02-copy-sample-files-02.png)

1. 이제 **새 항목 추가** 1️⃣ 및 **폴더** 2️⃣를 선택하여 새 폴더를 만듭니다.

    ![OneDrive에서 새 폴더를 추가하는 스크린샷입니다.](../media/1-02-copy-sample-files-03.png)

1. 폴더 이름을 **Northwind contracts**로 지정하고 **만들기**를 선택합니다.

    ![새 폴더의 이름을 'Northwind contracts'로 지정하는 스크린샷](../media/1-02-copy-sample-files-03-b.png)

1. 이제 이 새 폴더에서 **새로 추가** 1️⃣ 를 다시 선택하는데 이번에는 **파일 업로드** 2️⃣ 를 선택합니다.

    ![새 폴더에 새 파일을 추가하는 스크린샷](../media/1-02-copy-sample-files-04.png)

1. 이제 **작업 폴더**에서 **sampleDocs** 폴더를 찾습니다. 모든 파일을 1️⃣ 강조 표시하고 **확인** 2️⃣ 을 선택하여 파일을 모두 업로드합니다.

    ![이 리포지토리의 샘플 파일을 폴더에 업로드하는 스크린샷](../media/1-02-copy-sample-files-05.png)

이 작업을 일찍 수행하면 Microsoft 365 검색 엔진에서 검색을 할 준비가 되었을 때 이미 그 항목을 발견할 가능성이 높습니다.

## 작업 4 - Visual Studio Code용 Teams 도구 키트 구성

이 작업에서는 [Visual Studio Code용 Teams 도구 키트](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals?pivots=visual-studio-code-v5)의 현재 버전을 설치합니다. 이 작업을 수행하는 가장 쉬운 방법은 Visual Studio Code 내에서 바로 시작하는 것입니다.

1. Visual Studio Code에서 **작업 폴더** 열기 이 폴더의 작성자를 신뢰하라는 메시지가 표시될 수 있습니다. 그렇다면 수행하세요.

1. 이제 왼쪽에 있는 **Teams 도구 키트** 아이콘 1️⃣ 을 선택합니다. 새 프로젝트를 만드는 옵션이 제공된다면 잘못된 폴더에 있는 것일 수 있습니다. **Visual Studio Code 파일 메뉴**에서 **폴더 열기**를 선택하고 **msgext-northwind-inventory-ts** 폴더를 직접 엽니다. 아래와 같이 계정, 환경 등에 대한 섹션이 표시됩니다.

1. **계정**에서 **Microsoft 365에 로그인** 2️⃣ 을 선택하고 Microsoft 365 계정으로 로그인합니다.

    ![Teams 도구 키트 내에서 Microsoft 365에 로그인하는 스크린샷](../media/1-04-setup-teams-toolkit-01.png)

1. 브라우저 창이 열리고 Microsoft 365에 로그인할 수 있습니다. **지금 로그인한 상태이며 이 페이지를 닫습니다**라고 표시되면 닫으세요.

1. 마지막으로 **사용자 지정 앱 업로드 활성화됨** 옆에 녹색 체크 표시가 나타나는지 확인합니다. 그렇지 않은 경우 사용자 계정에 Teams 애플리케이션을 업로드할 수 있는 권한이 없다는 의미입니다. 이 권한은 기본적으로 꺼져 있습니다. [사용자가 사용자 지정 앱을 업로드할 수 있도록 설정하는 방법](https://learn.microsoft.com/microsoftteams/teams-custom-app-policies-and-settings#allow-users-to-upload-custom-apps)은 다음과 같습니다.

    ![테스트용 로드가 활성화되어 있는지 확인하는 스크린샷](../media/1-04-setup-teams-toolkit-03.png)

> [!NOTE]
> Microsoft 365 개발자 프로그램에는 Microsoft 365용 Copilot 라이선스가 포함되어 있지 않습니다. 따라서 개발자 테넌트를 사용하기로 결정한 경우 샘플을 메시지 확장으로만 테스트할 수 있습니다.

## 작업 확인

위의 모든 작업을 수행한 후에는 다음을 설치하고 컴퓨터에 다운로드해야 합니다.

- [Visual Studio Code](https://code.visualstudio.com/)(최신 버전)

- [Node.js(버전 18.x)](https://nodejs.org/download/release/v18.18.2/)

- [Azure Storage Explorer](https://azure.microsoft.com/products/storage/storage-explorer/)(선택 사항)

- [Visual Studio Code용 Teams 도구 키트](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals?pivots=visual-studio-code-v5)

- 샘플 리포지토리: [https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/)

모든 것이 올바르게 준비되었으면 이제 샘플 애플리케이션을 메시지 확장으로 실행할 준비가 되었습니다. 

[ 다음 연습으로 계속... ](./3-exercise-1-run-message-extension.md) 