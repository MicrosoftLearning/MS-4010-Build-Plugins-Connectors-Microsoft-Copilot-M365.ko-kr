---
lab:
  title: 연습 3 - 테스트 및 디버그
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# 연습 3 - 테스트 및 디버그

이 연습에서는 선언적 에이전트를 Microsoft 365에 테스트 및 배포하고 Microsoft 365 Copilot Chat을 사용하여 테스트합니다.

### 연습 기간

- **예상 완료 시간**: 5분

## 작업 1 - Microsoft 365 Copilot에서 선언적 에이전트 테스트

선언적 에이전트를 테스트하려면 Microsoft 365 테넌트에 앱으로 배포합니다. Microsoft 365 Copilot에서 연 후 의도한 대로 작동하는지 확인합니다.

Visual Studio Code:

1. **작업 표시줄**(사이드바)에서 **Teams 도구 키트** 확장을 엽니다.
1. **수명 주기** 창에서 **프로비전**을 선택합니다. Teams 도구 키트는 선언적 에이전트 프로젝트를 앱으로 패키지하고 Microsoft 365에 업로드합니다.
1. 웹 브라우저를 엽니다.

웹 브라우저에서:

1. [https://www.microsoft365.com/chat](https://www.microsoft365.com/chat)(으)로 이동합니다.
1. Microsoft 365 테넌트에 속한 회사 계정으로 로그인합니다.
1. Microsoft 365 Copilot의 가로 패널에서 **Contoso IT 정책** 에이전트를 선택하여 활성화합니다.
1. 채팅 텍스트 상자에 `What's the acceptable use policy at Contoso?`(을)를 입력합니다.
1. 에이전트가 응답할 때까지 기다립니다. 답변에 Graph 커넥터가 수집한 외부 콘텐츠에 대한 참조가 어떻게 포함되는지 확인합니다. 각 참조의 URL은 콘텐츠가 저장된 외부 시스템의 위치를 가리킵니다.

    ![사용자의 프롬프트에 응답하는 Microsoft 365 Copilot의 스크린샷.](../media/LAB_04/3-copilot-response.png)