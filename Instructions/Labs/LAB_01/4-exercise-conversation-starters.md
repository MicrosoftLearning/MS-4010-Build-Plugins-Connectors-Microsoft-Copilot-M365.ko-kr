---
lab:
  title: 연습 3 - 선언적 에이전트에 대화 스타터 추가
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# 연습 3 - 선언적 에이전트에 대화 스타터 추가

이 연습에서는 사용자에게 질문할 수 있는 질문 유형을 이해하는 데 도움이 되는 샘플 프롬프트를 제공하는 대화 시작자를 포함하도록 선언적 에이전트를 업데이트합니다.

### 연습 기간

- **예상 완료 시간**: 5분

## 작업 1 - 대화 시작 요소 추가

Visual Studio Code:

1. **appPackage** 폴더에서 **declarativeAgent.json** 파일을 엽니다.
1. 파일에 다음 코드 을 추가합니다.

   ```json
   "conversation_starters": [
       {
           "title": "Product information",
           "text": "Tell me about Eagle Air"
       },
       {
           "title": "Returns policy",
           "text": "What is the returns policy?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Repair information",
           "text": "Can you provide information on how to get a product repaired?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
   ]
   ```

1. 변경 내용을 저장합니다.

**declarativeAgent.json** 파일은 다음과 같은 모양이어야 합니다.

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
  "version": "v1.0",
  "name": "Product support",
  "description": "Product support agent that can help answer customer queries about Contoso Electronics products",
  "instructions": "$[file('instruction.txt')]",
  "capabilities": [
    {
      "name": "OneDriveAndSharePoint",
      "items_by_url": [
        {
          "url": "https://{tenant}-my.sharepoint.com/personal/{user}/Documents/Products"
        }
      ]
    }
  ],
  "conversation_starters": [
    {
      "title": "Product information",
      "text": "Tell me about Eagle Air"
    },
    {
      "title": "Returns policy",
      "text": "What is the returns policy?"
    },
    {
      "title": "Product information",
      "text": "Can you provide information on a specific product?"
    },
    {
      "title": "Product troubleshooting",
      "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
    },
    {
      "title": "Repair information",
      "text": "Can you provide information on how to get a product repaired?"
    },
    {
      "title": "Contact support",
      "text": "How can I contact support for help?"
    }
  ]
}
```

## 작업 2 - Microsoft 365 Copilot에서 선언적 에이전트 테스트

다음으로, 변경 내용을 업로드하고 디버그 세션을 시작합니다.

Visual Studio Code:

1. **작업 표시줄**에서 **Teams 도구 키트** 확장을 엽니다.
1. **수명 주기** 섹션에서 **프로비전**을 선택합니다.
1. 업로드가 완료될 때까지 기다립니다.
1. **작업 표시줄**에서 **실행 및 디버그** 보기로 전환합니다.
1. 구성의 드롭다운 옆에 있는 **디버깅 시작** 버튼을 선택하거나 <kbd>F5</kbd> 키를 누릅니다. 새 브라우저 창이 시작되고 Microsoft 365 Copilot으로 이동합니다.

그런 다음, Microsoft 365에서 선언적 에이전트를 테스트하고 결과의 유효성을 검사합니다.

웹 브라우저에서 계속합니다:

1. **Microsoft 365 Copilot**에서 오른쪽 상단의 아이콘을 선택하여 **Copilot 가로 패널 확장**으로 이동합니다.
1. 에이전트 목록에서 **제품 지원**을 찾아서 선택하면 몰입형 환경으로 들어가 에이전트와 직접 채팅할 수 있습니다. 매니페스트에서 정의한 대화 시작이 사용자 인터페이스에 표시됩니다.

![사용자 지정 대화 스타터를 사용하는 몰입형 환경에서 제품 지원 선언적 에이전트를 보여 주는 Microsoft Edge의 스크린샷.](../media/LAB_01/test-conversation-starters.png)

브라우저를 닫아 Visual Studio Code에서 디버그 세션을 중지합니다.