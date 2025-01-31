---
lab:
  title: 연습 3 - Microsoft 365 Copilot에서 API 플러그 인을 사용하여 선언적 에이전트 테스트
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# 연습 3 - Microsoft 365 Copilot에서 API 플러그 인을 사용하여 선언적 에이전트 테스트

마지막 단계는 Microsoft 365 Copilot에서 API 플러그 인을 사용하여 선언적 에이전트를 테스트하는 것입니다.

### 연습 기간

- **예상 완료 시간**: 10분

## 작업 1 - 디버그 프로비전 및 디버깅 시작

Visual Studio Code:

1. **작업 표시줄**에서 **Teams 도구 키트**를 선택합니다.
1. **계정** 섹션에서 Microsoft 365 Copilot를 사용하여 Microsoft 365 테넌트에 로그인되었는지 확인합니다.

    ![Visual Studio Code의 Teams 도구 키트 계정 섹션 스크린샷.](../media/LAB_03/3-teams-toolkit-accounts.png)

1. **작업 표시줄**에서 **실행 및 디버그**를 선택합니다.
1. **Copilot에서 디버그** 구성을 선택하고 **디버깅 시작** 버튼을 사용하여 디버깅을 시작합니다.  

    ![Visual Studio Code의 Copilot 구성 디버그 스크린샷.](../media/LAB_03/3-visual-studio-code-start-debugging.png)

1. Visual Studio Code는 프로젝트를 빌드하고 Microsoft 365 테넌트에 배포하고 새 웹 브라우저 창을 엽니다.

## 작업 2 - 테스트 및 결과 검토

웹 브라우저에서:

1. 메시지가 표시되면 Microsoft 365 Copilot을 사용하여 Microsoft 365 테넌트에 속한 계정으로 로그인합니다.
1. 사이드 바에서 **Il Ristorante**를 선택합니다.

    ![Il Ristorante 에이전트가 선택된 Microsoft 365 Copilot 인터페이스의 스크린샷.](../media/LAB_03/3-copilot-select-agent.png)

1. **What's for lunch today?** 대화 시작을 선택하고 프롬프트를 제출합니다.

    ![점심 식사 프롬프트가 있는 Microsoft 365 Copilot 인터페이스의 스크린샷.](../media/LAB_03/3-copilot-lunch-prompt.png)

1. 메시지가 표시되면 에이전트가 API로 전송하는 데이터를 검토하고 **한 번만 허용** 버튼을 사용하여 확인합니다.

    ![점심 식사 확인이 포함된 Microsoft 365 Copilot 인터페이스의 스크린샷.](../media/LAB_03/3-copilot-lunch-confirm.png)

1. 에이전트가 응답할 때까지 기다립니다. 이제 인용의 팝업에는 API의 추가 정보가 포함된 사용자 지정 적응형 카드가 포함됩니다.

    ![점심 응답이 있는 Microsoft 365 Copilot 인터페이스의 스크린샷.](../media/LAB_03/3-copilot-lunch-response.png)

1. 프롬프트 텍스트 상자에 **1x spaghetti, 1x iced tea**와 같이 입력한 후 프롬프트를 제출합니다.
1. 에이전트가 API로 전송하는 데이터를 검토하고 **확인** 버튼을 계속 사용합니다.

    ![주문 확인이 포함된 Microsoft 365 Copilot 인터페이스의 스크린샷.](../media/LAB_03/3-copilot-order-confirm.png)

1. 에이전트가 주문을 하고 주문 요약을 반환할 때까지 기다립니다. API가 단일 항목을 반환하기 때문에 에이전트는 적응형 카드를 사용하여 렌더링하고 응답에 카드를 직접 포함합니다.

    ![주문 응답이 있는 Microsoft 365 Copilot 인터페이스의 스크린샷.](../media/LAB_03/3-copilot-order-response.png)

1. Visual Studio Code로 돌아가서 디버깅을 중지합니다.
1. **터미널** 탭으로 전환하고 활성화된 모든 터미널을 닫습니다.

    ![모든 터미널을 닫는 옵션이 있는 Visual Studio Code 터미널 탭의 스크린샷.](../media/LAB_03/3-visual-studio-code-close-terminal.png)
