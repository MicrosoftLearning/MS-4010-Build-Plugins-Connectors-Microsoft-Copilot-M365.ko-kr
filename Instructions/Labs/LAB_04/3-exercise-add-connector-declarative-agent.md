---
lab:
  title: 연습 2 - 선언적 에이전트 만들기 및 Graph 커넥터 통합
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# 연습 2 - 선언적 에이전트 만들기 및 Graph 커넥터 통합

이 연습에서는 처음부터 새 선언적 에이전트를 만들고 외부 연결을 해당 접지 원본으로 사용하도록 구성합니다.

### 연습 기간

- **예상 완료 시간**: 10분

## 작업 1 - 선언적 에이전트 만들기

선언적 에이전트를 만드는 한 가지 방법은 Teams 도구 키트를 사용하는 것입니다. Teams 도구 키트는 선언적 에이전트를 만들기 위한 템플릿 프로젝트를 제공하므로 에이전트 설정을 구성하고 추가 기능을 포함하여 시작할 수 있는 좋은 위치를 제공합니다.

새 선언적 에이전트를 만들려면 Visual Studio Code를 엽니다.

Visual Studio Code:

1. **작업 표시줄**(사이드바)에서 **Teams 도구 키트** 확장을 엽니다.
1. Teams 도구 키트 창에서 **새 앱 만들기** 단추를 선택합니다.
1. **새 프로젝트** 대화 상자에서 **에이전트** 옵션을 선택합니다.
1. 다음 대화 상자에서 **선언적 에이전트** 옵션을 선택합니다.
1. **플러그인 없음** 옵션을 선택하여 플러그인을 추가하지 않습니다.
1. 컴퓨터에서 프로젝트를 저장할 폴더를 선택합니다.
1. 프로젝트 이름을 `da-it-policies`(으)로 지정합니다.

## 작업 2 - 선언적 에이전트 지침 구성

Teams 도구 키트는 새 선언적 에이전트 프로젝트를 만듭니다. 시나리오로 범위를 지정하려면 에이전트의 설명 및 지침을 업데이트합니다.

Visual Studio Code:

1. **appPackage/declarativeAgent.json** 파일을 엽니다.
1. **이름** 속성 값을 `Contoso IT policies`(으)로 업데이트합니다.
1. **description** 속성 값을 `Assistant specialized in Contoso IT policies`(으)로 업데이트합니다.
1. 변경 내용을 저장합니다.
1. 업데이트된 파일의 내용은 다음과 같습니다.

    ```json
    {
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
    }
    ```

1. **appPackage/instruction.txt** 파일을 엽니다.
1. 콘텐츠를 다음으로 업데이트합니다.

    ```text
    You are a helpful assistant that can answer questions from users in a friendly manner. You should take your time to respond. Your responses should be accurate and helpful. If you don't have the information, you should say so in your response. When answering follow-up questions, you should review the information you gathered from external sources, if you don't already have the information to give an accurate answer, you should search for more information. Only answer when you have the information to give an accurate response.
    ```

1. 변경 내용을 저장합니다.

## 작업 3 - 선언적 에이전트와 Microsoft Graph 커넥터 통합

선언적 에이전트를 만든 후 다음 단계는 외부 데이터에 액세스할 수 있도록 Microsoft Graph 커넥터와 통합하는 것입니다.

Visual Studio Code:

1. **appPackage/declarativeAgent.json** 파일을 엽니다.
1. **instructions** 속성 뒤에 다음 코드를 사용하여 **capabilities**라는 새 속성을 추가합니다:

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": ""
            }
          ]
        }
      ]
    } 
    ```

1. **connection_id** 속성에서 외부 연결의 ID로 `policieslocal`(을)를 지정합니다. `policieslocal`은(는) 이전 단계에서 그래프 커넥터가 생성한 외부 연결의 ID입니다.
1. 업데이트된 파일의 내용은 다음과 같습니다.

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": "policieslocal"
            }
          ]
        }
      ]
    } 
    ```

1. 변경 내용을 저장합니다.
