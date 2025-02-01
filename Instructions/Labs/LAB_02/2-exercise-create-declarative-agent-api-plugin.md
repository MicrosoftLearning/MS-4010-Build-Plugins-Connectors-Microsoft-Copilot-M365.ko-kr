---
lab:
  title: 연습 1 - 프로젝트 다운로드 및 파일 검사
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# 연습 1 - 프로젝트 다운로드 및 파일 검사

작업을 사용하여 선언적 에이전트를 확장하면 외부 시스템에 저장된 데이터를 실시간으로 검색하고 업데이트할 수 있습니다. API 플러그 인을 사용하여 API를 통해 외부 시스템에 연결하여 정보를 검색하고 업데이트할 수 있습니다.

### 연습 기간

- **예상 완료 시간**: 10분

## 작업 1 - 시작 프로젝트 다운로드

먼저 샘플 프로젝트를 다운로드합니다. 웹 브라우저에서:

1. [https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript)으로 이동합니다.
    1. 단계에 따라 [리포지토리 소스 코드를 컴퓨터에 다운로드](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view)합니다.
    1. 다운로드한 ZIP 파일 내용물의 압축을 풀고 **문서 폴더**에 압축을 풉니다.
    1. Visual Studio Code에서 폴더를 엽니다.

샘플 프로젝트는 선언적 에이전트와 Azure Functions에서 실행되는 익명 API를 포함하는 Teams 도구 키트 프로젝트입니다. 선언적 에이전트는 Teams 도구 키트를 사용하여 새로 만든 선언적 에이전트와 동일합니다. API는 가상의 이탈리아 레스토랑에 속하는 것으로, 오늘의 메뉴를 찾아 주문을 할 수 있습니다.

## 작업 2 - API 정의 검사

먼저 이탈리아 레스토랑 API의 API 정의를 살펴봅니다.

Visual Studio Code:

1. **탐색기** 보기에서 **appPackage/apiSpecificationFile/ristorante.yml** 파일을 엽니다. 이 파일은 이탈리아 레스토랑의 API를 설명하는 OpenAPI 사양입니다.
1. **servers.url** 속성을 찾습니다.

    ```yaml
    servers:
      - url: http://localhost:7071/api
        description: Il Ristorante API server
    ```

    Azure Functions를 로컬로 실행할 때 표준 URL과 일치하는 로컬 URL을 가리킵니다.

1. 두 개의 작업이 포함된 **paths** 속성을 찾습니다. 오늘의 메뉴를 검색하려면 **/dishes**, 주문을 하려면 **/orders**를 입력합니다.

    > [!IMPORTANT]
    > 각 작업에는 API 사양에서 작업을 고유하게 식별하는 **operationId** 속성이 포함되어 있습니다. Copilot에는 특정 사용자 프롬프트에 대해 호출해야 하는 API를 알 수 있도록 각 작업에 고유한 ID가 있어야 합니다.

## 작업 3 - API 구현 검사

다음으로, 이 연습에서 사용하는 샘플 API를 살펴봅니다.

Visual Studio Code:

1. **탐색기** 보기에서 **src/data.json** 파일을 엽니다. 이 파일에는 이탈리아 레스토랑의 가상 메뉴 항목이 포함되어 있습니다. 각 요리는 다음으로 구성됩니다.

    - 이름,
    - 설명,
    - 이미지 링크,
    - 가격,
    - 제공되는 코스,
    - 형식(요리 또는 음료),
    - 알레르기 유발 물질 목록(선택)

    이 연습에서 API는 이 파일을 데이터 원본으로 사용합니다.
1. 다음으로 **src/functions** 폴더를 확장합니다. **dishes.ts**와 **placeOrder.ts**라는 두 개의 파일을 확인합니다. 이러한 파일에는 API 사양에 정의된 두 작업의 구현이 포함되어 있습니다.
1. **src/functions/dishes.ts** 파일을 엽니다. 잠시 API의 작동 방식을 검토하세요. **src/functions/data.json** 파일에서 샘플 데이터를 로드하는 것으로 시작합니다.

    ```typescript
    import data from "../data.json";
    ```

    다음으로, API를 호출하는 클라이언트가 전달할 수 있는 가능한 필터를 다양한 쿼리 문자열 매개변수에서 찾습니다.

    ```typescript
    const course = req.query.get('course');
    const allergensString = req.query.get('allergens');
    const allergens: string[] = allergensString ? allergensString.split(",") : [];
    const type = req.query.get('type');
    const name = req.query.get('name');
    ```

    요청에 지정된 필터에 따라 API는 데이터 집합을 필터링하고 응답을 반환합니다.

1. 다음으로, **src/functions/placeOrder.ts** 파일에 정의된 주문하기 API를 살펴봅니다. API는 샘플 데이터를 참조하는 것으로 시작합니다. 그런 다음 클라이언트가 요청 본문에서 보내는 주문의 형태를 정의합니다.

    ```typescript
    interface OrderedDish {
      name?: string;
      quantity?: number;
    }
    interface Order {
      dishes: OrderedDish[];
    }
    ```

    API는 요청을 처리할 때 먼저 요청에 본문이 포함되어 있는지와 올바른 형태인지 확인합니다. 그렇지 않은 경우 400 잘못된 요청 오류로 요청을 거부합니다.

    ```typescript
    let order: Order | undefined;
    try {
      order = await req.json() as Order | undefined;
    }
    catch (error) {
      return {
        status: 400,
        jsonBody: { message: "Invalid JSON format" },
      } as HttpResponseInit;
    }
    if (!order.dishes || !Array.isArray(order.dishes)) {
      return {
        status: 400,
        jsonBody: { message: "Invalid order format" }
      } as HttpResponseInit;
    }
    ```

    다음으로, API는 요청을 메뉴의 요리로 확인하고 총 가격을 계산합니다.

    ```typescript
    let totalPrice = 0;
    const orderDetails = order.dishes.map(orderedDish => {
      const dish = data.find(d => d.name.toLowerCase().includes(orderedDish.name.toLowerCase()));
      if (dish) {
        totalPrice += dish.price * orderedDish.quantity;
        return {
          name: dish.name,
          quantity: orderedDish.quantity,
          price: dish.price,
        };
      }
      else {
        context.error(`Invalid dish: ${orderedDish.name}`);
        return null;
      }
    });
    ```

    > [!IMPORTANT]
    > API는 클라이언트가 ID가 아닌 이름의 일부로 요리를 지정하는 방법을 확인합니다. 이는 대규모 언어 모델이 숫자보다 단어에 더 잘 작동하기 때문에 일부러 그렇게 한 것입니다. 또한 주문하기 위해 API를 호출하기 전에 Copilot에는 사용자 프롬프트의 일부로 쉽게 사용할 수 있는 요리 이름이 있습니다. Copilot가 ID로 요리를 참조해야 하는 경우 먼저 추가 API 요청이 필요한 항목과 지금 수행할 수 없는 Copilot을 검색해야 합니다.

    API가 준비되면 총 가격 및 임의로 생성된 주문 ID 및 상태가 포함된 응답을 반환합니다.

    ```typescript
    const orderId = Math.floor(Math.random() * 10000);
    return {
      status: 201,
      jsonBody: {
        order_id: orderId,
        status: "confirmed",
        total_price: totalPrice,
      }
    } as HttpResponseInit;
    ```

