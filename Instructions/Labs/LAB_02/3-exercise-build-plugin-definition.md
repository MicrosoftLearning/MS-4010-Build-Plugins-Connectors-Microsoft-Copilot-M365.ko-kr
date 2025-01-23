---
lab:
  title: 연습 2 - API 플러그 인 정의 빌드
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# 연습 2 - API 플러그 인 정의 빌드

다음 단계는 플러그 인 정의를 프로젝트에 추가하는 것입니다. 플러그 인 정의에는 다음 정보가 들어 있습니다.

- 플러그인 이 수행할 수 있는 작업입니다.
- 예상하고 반환하는 데이터의 형태는 무엇인가요?
- 선언적 에이전트가 기본 API를 호출하는 방법입니다.

### 연습 기간

- **예상 완료 시간**: 10분

## 작업 1 - 기본 플러그 인 정의 구조 추가

Visual Studio Code:

1. **appPackage** 폴더에 **ai-plugin.json**이라는 새 파일을 추가합니다.
1. 다음 콘텐츠를 붙여넣습니다.

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [ 
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

    파일에는 사용자 및 모델에 대한 설명이 포함된 API 플러그 인의 기본 구조가 포함되어 있습니다. **description_for_model**에는 에이전트가 플러그 인 호출을 고려해야 하는 시점을 이해하는 데 도움이 되도록 플러그 인이 수행할 수 있는 작업에 대한 자세한 정보가 포함되어 있습니다.
1. 변경 내용을 저장합니다.

## 작업 2 - 함수 정의

API 플러그 인은 API 사양에 정의된 API 작업에 매핑되는 하나 이상의 함수를 정의합니다. 각 함수는 사용자에게 데이터를 표시하는 방법을 에이전트에 지시하는 이름 및 설명 및 응답 정의로 구성됩니다.

### 메뉴를 검색하는 함수 정의

오늘 메뉴에 대한 정보를 검색하는 함수 정의부터 시작합니다.

Visual Studio Code:

1. **appPackage/ai-plugin.json** 파일을 엽니다.
1. **functions** 배열에 다음 코드 조각을 추가합니다.

    ```json
    {
      "name": "getDishes",
      "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.dishes",
          "properties": {
            "title": "$.name",
            "subtitle": "$.description"
          }
        }
      }
    }
    ```

    API 사양에서 **getDishes** 작업을 호출하는 함수를 정의하는 것으로 시작합니다. 다음으로 함수 설명을 제공합니다. 이 설명은 Copilot이 사용자 프롬프트에 대해 어떤 기능을 호출할지 결정할 때 사용하기 때문에 중요합니다.

    response_semantics 속성에서 Copilot이 API에서 수신한 데이터를 표시하는 방법을 지정합니다. API는 메뉴의 요리에 대한 정보를 **dishes** 속성으로 반환하므로 **data_path** 속성을 `$.dishes` JSONPath 표현식으로 설정합니다.

    다음으로, **속성** 섹션에서 API 응답에서 제목, 설명 및 URL을 나타내는 속성을 매핑합니다. 이 경우 요리에 URL이 없으므로 **제목**과 **설명**만 매핑합니다.

1. 완성된 코드 조각은 다음과 같습니다.

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.dishes",
              "properties": {
                "title": "$.name",
                "subtitle": "$.description"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. 변경 내용을 저장합니다.

### 주문하는 함수 정의

다음으로, 주문하는 함수를 정의합니다.

Visual Studio Code:

1. **appPackage/ai-plugin.json** 파일을 엽니다.
1. **functions** 배열 끝에 다음 코드 조각을 추가합니다.

    ```json
    {
      "name": "placeOrder",
      "description": "Places an order and returns the order details",
      "capabilities": {
        "response_semantics": {
          "data_path": "$",
          "properties": {
            "title": "$.order_id",
            "subtitle": "$.total_price"
          }
        }
      }
    }
    ```

    먼저 ID **placeOrder**로 API 작업을 참조합니다. 그런 다음 Copilot이 이 기능을 사용자의 프롬프트와 일치시키는 데 사용하는 설명을 입력합니다. 다음으로, 데이터를 반환하는 방법을 Copilot에 지시합니다. 다음은 주문 후 API가 반환하는 데이터입니다.

    ```json
    {
      "order_id": 6532,
      "status": "confirmed",
      "total_price": 21.97
    }
    ```

    표시하려는 데이터가 응답 객체의 루트에 직접 위치하므로 **data_path**를 JSON 객체의 최상위 노드를 나타내는 **$**(으)로 설정합니다. 제목에 주문 번호를 표시하고 부제에 가격을 표시하도록 정의합니다.

1. 전체 파일은 다음과 같습니다.

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          "description": "Places an order and returns the order details",
          "capabilities": {
            "response_semantics": {
              "data_path": "$",
              "properties": {
                "title": "$.order_id",
                "subtitle": "$.total_price"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. 변경 내용을 저장합니다.

## 작업 3 - 런타임 정의

Copilot 호출할 함수를 정의한 후 다음 단계는 호출 방법을 지시하는 것입니다. 플러그 인 정의의 **런타임** 섹션에서 이 작업을 수행합니다.

Visual Studio Code:

1. **appPackage/ai-plugin.json** 파일을 엽니다.
1. 런타임 배열에서 다음 코드를 추가합니다.

    ```json
    {
      "type": "OpenApi",
      "auth": {
        "type": "None"
      },
      "spec": {
        "url": "apiSpecificationFile/ristorante.yml"
      },
      "run_for_functions": [
        "getDishes",
        "placeOrder"
      ]
    }
    ```

    먼저 Copilot에 호출할 API(**유형: OpenAPI**)에 대한 OpenAPI 정보를 제공하고 익명(**auth.type: None**)임을 알려주어야 합니다. 다음으로, **사양** 섹션에서 프로젝트에 있는 API 사양에 대한 상대 경로를 지정합니다. 마지막으로 **run_for_functions** 속성에는 이 API에 속하는 모든 함수를 나열합니다.

1. 전체 파일은 다음과 같습니다.

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          ...trimmed for brevity
        }
      ],
      "runtimes": [
        {
          "type": "OpenApi",
          "auth": {
            "type": "None"
          },
          "spec": {
            "url": "apiSpecificationFile/ristorante.yml"
          },
          "run_for_functions": [
            "getDishes",
            "placeOrder"
          ]
        }
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. 변경 내용을 저장합니다.

