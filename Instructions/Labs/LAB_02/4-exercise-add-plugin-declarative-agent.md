---
lab:
  title: 연습 3 - 선언적 에이전트에 플러그 인 정의 연결
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# 연습 3 - 선언적 에이전트에 플러그 인 정의 연결

API 플러그 인 정의 빌드를 완료한 후 다음 단계는 선언적 에이전트에 등록하는 것입니다. 사용자가 선언적 에이전트와 상호 작용할 때 정의된 API 플러그 인에 대한 사용자의 프롬프트와 일치하고 관련 함수를 호출합니다.

### 연습 기간

- **예상 완료 시간**: 5분

## 작업 1 - 플러그 인 정의를 선언적 에이전트에 연결

Visual Studio Code:

1. **appPackage/declarativeAgent.json** 파일을 엽니다.
1. **instructions** 속성 뒤에 다음 코드 조각을 추가합니다.

    ```json
    "actions": [
      {
        "id": "menuPlugin",
        "file": "ai-plugin.json"
      }
    ]
    ```

    이 코드 조각을 사용하여 선언적 에이전트를 API 플러그 인에 연결합니다. 플러그 인에 대한 고유 ID를 지정하고 플러그 인의 정의를 찾을 수 있는 위치를 에이전트에 지시합니다.

1. 전체 파일은 다음과 같습니다.

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Declarative agent",
      "description": "Declarative agent created with Teams Toolkit",
      "instructions": "$[file('instruction.txt')]",
      "actions": [
        {
          "id": "menuPlugin",
          "file": "ai-plugin.json"
        }
      ]
    }
    ```

1. 변경 내용을 저장합니다.

## 작업 2 - 선언적 에이전트 정보 및 명령 업데이트

이 연습에서 빌드하는 선언적 에이전트는 사용자가 현지 이탈리아 레스토랑의 메뉴를 찾아 주문을 할 수 있도록 도와줍니다. 이 시나리오에 대해 에이전트를 최적화하려면 이름, 설명 및 지침을 업데이트합니다.

Visual Studio Code:

1. 선언적 에이전트 정보를 업데이트합니다.
    1. **appPackage/declarativeAgent.json** 파일을 엽니다.
    1. **name** 속성 값을 **Il Ristorante**로 업데이트합니다.
    1. **description** 속성의 값을 **Order the most delicious Italian dishes and drinks from the comfort of your desk.** 로 업데이트합니다.
    1. 변경 내용을 저장합니다.
1. 선언적 에이전트의 지침을 업데이트합니다.
    1. **appPackage/instruction.txt** 파일을 엽니다.
    1. 해당 내용을 다음으로 대체합니다.

        ```markdown
        You are an assistant specialized in helping users explore the menu of an Italian restaurant and place orders. You interact with the restaurant's menu API and guide users through the ordering process, ensuring a smooth and delightful experience. Follow the steps below to assist users in selecting their desired dishes and completing their orders:
        
        ### General Behavior:
        - Always greet the user warmly and offer assistance in exploring the menu or placing an order.
        - Use clear, concise language with a friendly tone that aligns with the atmosphere of a high-quality local Italian restaurant.
        - If the user is browsing the menu, offer suggestions based on the course they are interested in (breakfast, lunch, or dinner).
        - Ensure the conversation remains focused on helping the user find the information they need and completing the order.
        - Be proactive but never pushy. Offer suggestions and be informative, especially if the user seems uncertain.
        
        ### Menu Exploration:
        - When a user requests to see the menu, use the `GET /dishes` API to retrieve the list of available dishes, optionally filtered by course (breakfast, lunch, or dinner).
          - Example: If a user asks for breakfast options, use the `GET /dishes?course=breakfast` to return only breakfast dishes.
        - Present the dishes to the user with the following details:
          - Name of the dish
          - A tasty description of the dish
          - Price in € (Euro) formatted as a decimal number with two decimal places
          - Allergen information (if relevant)
          - Don't include the URL.
        
        ### Beverage Suggestion:
        - If the order does not already include a beverage, suggest a suitable beverage option based on the course.
        - Use the `GET /dishes?course={course}&type=drink` API to retrieve available drinks for that course.
        - Politely offer the suggestion: *"Would you like to add a beverage to your order? I recommend [beverage] for [course]."*
        
        ### Placing the Order:
        - Once the user has finalized their order, use the `POST /order` API to submit the order.
          - Ensure the request includes the correct dish names and quantities as per the user's selection.
          - Example API payload:
         
            ```json
            {
              "dishes": [
                {
                  "name": "frittata",
                  "quantity": 2
                },
                {
                  "name": "cappuccino",
                  "quantity": 1
                }
              ]
            }
            ```
        
        ### Error Handling:
        - If the user selects a dish that is unavailable or provides an invalid dish name, respond gracefully and suggest alternative options.
          - Example: *"It seems that dish is currently unavailable. How about trying [alternative dish]?"*
        - Ensure that any errors from the API are communicated politely to the user, offering to retry or explore other options.
        ```

        지침에서는 에이전트의 일반적인 동작을 정의하고 에이전트가 수행할 수 있는 기능을 지시합니다. 또한 API에서 예상하는 데이터의 모양을 포함하여 주문하기와 관련된 특정 동작에 대한 지침도 포함됩니다. 에이전트가 의도한 대로 작동하는지 확인하기 위해 이 정보를 포함합니다.

    > [!NOTE]
    > 서식을 유지하려면 Visual Studio Code로 복사하기 전에 메모장에 복사/붙여넣기 작업을 여러 번 수행해야 할 수 있습니다.

    1. 변경 내용을 저장합니다.

1. 사용자가 에이전트를 사용할 수 있는 항목을 이해하도록 돕기 위해 대화 시작자를 추가합니다.
    1. **appPackage/declarativeAgent.json** 파일을 엽니다.
    1. **instructions** 속성 뒤에 **conversation_starters**라는 새 속성을 추가합니다:

        ```json
        "conversation_starters": [
          {
            "text": "What's for lunch today?"
          },
          {
            "text": "What can I order for dinner that is gluten-free?"
          }
        ]
        ```

    1. 전체 파일은 다음과 같습니다.

        ```json
        {
          "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
          "version": "v1.0",
          "name": "Il Ristorante",
          "description": "Order the most delicious Italian dishes and drinks from the comfort of your desk.",
          "instructions": "$[file('instruction.txt')]",
          "conversation_starters": [
            {
              "text": "What's for lunch today?"
            },
            {
              "text": "What can I order for dinner that is gluten-free?"
            }
          ],
          "actions": [
            {
              "id": "menuPlugin",
              "file": "ai-plugin.json"
            }
          ]
        }
        ```

    1. 변경 내용을 저장합니다.

## 작업 3 - API URL 업데이트

선언적 에이전트를 테스트하려면 먼저 API 사양 파일에서 API의 URL을 업데이트해야 합니다. 현재 URL은 `http://localhost:7071/api`(으)로 설정되어 있으며, 이는 Azure Functions가 로컬에서 실행할 때 사용하는 URL입니다. 그러나 Copilot가 클라우드에서 API를 호출하도록 하려면 API를 인터넷에 노출해야 합니다. Teams 도구 키트는 개발 터널을 만들어 인터넷을 통해 로컬 API를 자동으로 노출합니다. 프로젝트 디버깅을 시작할 때마다 Teams 도구 키트는 새 개발 터널을 시작하고 해당 URL을 **OPENAPI_SERVER_URL** 변수에 저장합니다. Teams 도구 키트가 터널을 시작하고 **.vscode/tasks.json** 파일, **로컬 터널 시작** 작업에서 해당 URL을 저장하는 방법을 확인할 수 있습니다.

```json
{
  // Start the local tunnel service to forward public URL to local port and inspect traffic.
  // See https://aka.ms/teamsfx-tasks/local-tunnel for the detailed args definitions.
  "label": "Start local tunnel",
  "type": "teamsfx",
  "command": "debug-start-local-tunnel",
  "args": {
    "type": "dev-tunnel",
    "ports": [
      {
        "portNumber": 7071,
        "protocol": "http",
        "access": "public",
        "writeToEnvironmentFile": {
          "endpoint": "OPENAPI_SERVER_URL", // output tunnel endpoint as OPENAPI_SERVER_URL
        }
      }
    ],
    "env": "local"
  },
  "isBackground": true,
  "problemMatcher": "$teamsfx-local-tunnel-watch"
}
```

이 터널을 사용하려면 **OPENAPI_SERVER_URL** 변수를 사용하도록 API 사양을 업데이트해야 합니다.

Visual Studio Code:

1. **appPackage/apiSpecificationFile/ristorante.yml** 파일을 엽니다.
1. **servers.url** 속성의 값을 **${{OPENAPI_SERVER_URL}}/api**로 변경합니다.
1. 변경된 파일은 다음과 같습니다.

    ```yaml
    openapi: 3.0.0
    info:
      title: Il Ristorante menu API
      version: 1.0.0
      description: API to retrieve dishes and place orders for Il Ristorante.
    servers:
      - url: ${{OPENAPI_SERVER_URL}}/api
        description: Il Ristorante API server
    paths:
      ...trimmed for brevity
    ```

1. 변경 내용을 저장합니다.

API 플러그 인이 완료되고 선언적 에이전트와 통합됩니다. Microsoft 365 Copilot에서 에이전트 테스트를 계속합니다.