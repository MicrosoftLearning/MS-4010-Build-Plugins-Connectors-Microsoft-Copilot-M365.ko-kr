---
lab:
  title: 소개
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# 소개

선언적 에이전트를 사용하여 Microsoft 365 Copilot을 확장합니다. 사용자 지정 지식을 정의하여 권위 있는 콘텐츠로 질문에 답변할 수 있는 에이전트를 만들 수 있습니다.

## 예제 시나리오

고객 지원팀에서 일한다고 가정해 보겠습니다. 사용자와 팀은 조직에서 고객으로부터 만드는 제품에 대한 쿼리를 처리합니다. 응답 시간을 개선하고 더 나은 환경을 제공하려고 합니다. 제품 사양, 자주하는 질문, 수리, 반품 및 보증 처리 정책을 포함하는 SharePoint Online 사이트의 문서 라이브러리에 문서를 저장합니다. 자연어를 사용하여 이러한 문서의 정보를 쿼리하고 고객 쿼리에 대한 답변을 빠르게 얻을 수 있기를 원합니다.

## 이 모듈에서 수행할 작업

여기서는 Microsoft 365의 문서에 저장된 정보를 사용하여 제품 지원 질문에 대답할 수 있는 선언적 에이전트를 만듭니다.

- **만들기**: 선언적 에이전트 프로젝트를 만들고 Visual Studio Code에서 Teams 도구 키트를 사용합니다.
- **사용자 지정 지침**: 사용자 지정 지침을 정의하여 응답을 형성합니다.
- **사용자 지정 그라운딩**: 그라운딩 데이터를 구성하여 에이전트에 추가 컨텍스트를 추가합니다.
- **대화 시작**: 새 대화를 시작하기 위한 프롬프트를 정의합니다.
- **프로비전**: Microsoft 365 Copilot에 선언적 에이전트를 업로드하고 결과의 유효성을 검사합니다.

## 필수 조건

- Microsoft 365 Copilot의 정의 및 작동 방식에 대한 기본 지식
- Microsoft 365 Copilot의 선언적 에이전트 정의에 대한 기본 지식
- Microsoft 365 Copilot을 사용하는 Microsoft 365 테넌트
- Microsoft Teams에 사용자 지정 앱을 업로드할 수 있는 권한이 있는 계정
- Microsoft 365 Copilot을 사용하는 Microsoft 365 테넌트에 액세스
- [Teams 도구 키트](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) 확장이 설치된 [Visual Studio Code](https://code.visualstudio.com/)
- [Node.js v18](https://nodejs.org/en/download/package-manager)

## 랩 기간

- **예상 완료 시간**: 30분

## 학습 목표

이 모듈을 마치면 선언적 에이전트를 만들고 Microsoft 365에 업로드한 다음 Microsoft 365 Copilot에서 사용하여 결과의 유효성을 검사할 수 있습니다.

시작 준비가 되면 [첫 번째 연습을 진행합니다.](./2-exercise-create-declarative-agent.md)
