---
lab:
  title: 소개
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# 소개

Microsoft 365 Copilot 에이전트를 사용하면 특정 시나리오에 최적화된 AI 지원 도우미를 만들 수 있습니다. 지침을 사용하여 Copilot의 컨텍스트를 정의하고 에이전트의 목소리 톤이나 응답 방식을 구성할 수 있습니다. 에이전트의 지식을 구성하면 LLM(대규모 언어 모델)에 포함되지 않은 외부 데이터에 액세스할 수 있으므로 보다 정확하게 응답할 수 있습니다. 

## 예제 시나리오

대규모 조직의 IT 부서에서 일한다고 가정해 보겠습니다. 조직에서는 특수 시스템에 저장하는 다양한 정책을 통해 IT를 표준화합니다. IT 부서의 사용자와 동료는 정책에서 다루는 질문을 정기적으로 받습니다. 정책 관리 시스템에서 답변을 찾는 데 시간이 많이 걸립니다. 정책의 권위 있는 정보를 사용하여 동료의 질문에 답변할 수 있는 AI 기반 도우미를 조직에 제공하고자 합니다.

## 학습 목표

이 모듈을 마치면 Microsoft 365 Copilot용 선언적 에이전트를 빌드할 수 있습니다. 특정 시나리오에 맞게 최적화하도록 지침을 구성하는 방법을 이해합니다. Microsoft Graph 커넥터와 통합하여 외부 데이터에 대한 액세스 권한을 부여하는 방법도 알 수 있습니다. 이는 Microsoft 365 Copilot의 LLM에 속하지 않습니다.

## 필수 조건

- Microsoft 365 Copilot의 개념과 초급 수준에서 작동하는 방식에 대한 지식
- Microsoft 365 Copilot 선언적 에이전트를 빌드하는 방법에 대한 지식
- Graph 커넥터를 빌드하는 방법 알아보기
- Microsoft 365 Copilot를 사용하는 Microsoft 365 테넌트 및 테넌트 관리자 권한
- [Teams 도구 키트](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) 확장이 설치된 [Visual Studio Code](https://code.visualstudio.com/)
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)
- [Node.js v18](https://nodejs.org/)
