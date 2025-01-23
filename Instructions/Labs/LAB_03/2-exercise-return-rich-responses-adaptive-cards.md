---
lab:
  title: 연습 1 - 프로젝트 다운로드 및 적응형 카드 빌드
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# 연습 1 - 프로젝트 다운로드 및 적응형 카드 빌드

먼저 에이전트에 대한 적응형 카드 템플릿을 빌드하여 응답에 데이터를 표시해 보겠습니다. 적응형 카드 템플릿을 빌드하려면 적응형 카드 미리 보기 Visual Studio Code 확장을 사용하여 Visual Studio Code에서 직접 작업을 쉽게 미리 볼 수 있습니다. 확장을 사용하면 데이터에 대한 참조를 사용하여 적응형 카드 템플릿을 빌드할 수 있습니다. 런타임 시 에이전트는 자리 표시자를 API에서 검색하는 데이터로 채웁니다.

### 연습 기간

- **예상 완료 시간**: 10분

## 작업 1 - 시작 프로젝트 다운로드

먼저 샘플 프로젝트를 다운로드합니다. 웹 브라우저에서:

1. [https://github.com/microsoft/learn-declarative-agent-api-plugin-adaptive-cards-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-adaptive-cards-typescript)로 이동합니다.
  1. 단계에 따라 [리포지토리 소스 코드를 컴퓨터에 다운로드](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view)합니다.
  1. 다운로드한 ZIP 파일 내용물의 압축을 풀고 **문서 폴더**에 압축을 풉니다.
  1. Visual Studio Code에서 폴더를 엽니다.

샘플 프로젝트는 API 플러그 인을 사용하여 빌드된 작업이 있는 선언적 에이전트를 포함하는 Teams 도구 키트 프로젝트입니다. API 플러그 인은 프로젝트에 포함된 Azure Functions에서 실행되는 익명 API에 연결합니다. API는 가상의 이탈리아 레스토랑에 속하는 것으로, 오늘의 메뉴를 찾아 주문을 할 수 있습니다.

## 작업 2 - 요리에 대한 적응형 카드 빌드

먼저 단일 요리에 대한 정보를 표시하는 적응형 카드를 만듭니다.

Visual Studio Code:

1. **탐색기** 보기에서 **cards**라는 이름의 새 폴더를 만듭니다.
1. **cards** 폴더에 **dish.json**이라는 새 파일을 만듭니다. 빈 적응형 카드를 나타내는 다음 내용을 붙여넣습니다.

  ```json
  {
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": []
  }
  ```

1. 계속하기 전에 작업 표시줄의 **확장** 탭에서 **적응형 카드 미리보기** 확장을 검색하여 설치한 다음 적응형 카드의 데이터 파일을 생성합니다.
  1. 키보드에서 <kbd>CTRL</kbd>+<kbd>P</kbd> 키를 눌러 명령 팔레트를 엽니다. `>Adaptive`(을)를 입력하면 적응형 카드 작업과 관련된 명령을 찾을 수 있습니다.

    ![적응형 카드 작업과 관련된 명령을 보여 주는 Visual Studio Code의 스크린샷.](../media/LAB_03/LAB_03/3-visual-studio-code-adaptive-card-commands.png)

  1. 목록에서 **적응형 카드: 새 데이터 파일**을 선택합니다. Visual Studio Code에서 **bicepconfig.json**이라는 새 파일을 만듭니다.
  1. 해당 내용을 요리를 나타내는 데이터로 대체합니다.

  ```json
  {
    "id": 4,
    "name": "Caprese Salad",
    "description": "Juicy vine-ripened tomatoes, fresh mozzarella, and fragrant basil leaves, drizzled with extra virgin olive oil and a touch of balsamic.",
    "image_url": "https://raw.githubusercontent.com/pnp/copilot-pro-dev-samples/main/samples/da-ristorante-api/assets/caprese_salad.jpeg",
    "price": 10.5,
    "allergens": [
    "dairy"
    ],
    "course": "lunch",
    "type": "dish"
  }
  ```

  1. 변경 내용을 저장합니다.
1. **dish.json** 파일로 돌아갑니다.
1. 렌즈에서 **적응형 카드 미리 보기**를 선택합니다.

  ![적응형 카드 미리 보기를 보여 주는 Visual Studio Code의 스크린샷.](../media/LAB_03/LAB_03/3-visual-studio-code-adaptive-card-preview.png)

  Visual Studio Code는 카드의 미리 보기를 측면에 엽니다. 카드를 편집할 때 변경 내용이 측면에 즉시 표시됩니다.

1. **body** 배열에, **Container** 요소를 추가하고, **image_url** 속성에 저장된 이미지 URL을 참조합니다.

  ```json
  {
    "type": "Container",
    "items": [
    {
      "type": "Image",
      "url": "${image_url}",
      "size": "large"
    }
    ]
  }
  ```

  카드 미리보기가 자동으로 업데이트되어 카드가 표시되는 방식에 주목합니다.

  ![이미지가 있는 적응형 카드 미리 보기를 보여 주는 Visual Studio Code의 스크린샷.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-image.png)

1. 다른 요리 속성에 대한 참조를 추가합니다. 전체 카드는 다음과 같습니다.

  ```json
  {
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [
    {
      "type": "Container",
      "items": [
      {
        "type": "Image",
        "url": "${image_url}",
        "size": "large"
      },
      {
        "type": "TextBlock",
        "text": "${name}",
        "weight": "Bolder"
      },
      {
        "type": "TextBlock",
        "text": "${description}",
        "wrap": true
      },
      {
        "type": "TextBlock",
        "text": "Allergens: ${if(count(allergens) > 0, join(allergens, ', '), 'none')}",
        "weight": "Lighter"
      },
      {
        "type": "TextBlock",
        "text": "**Price:** €${formatNumber(price, 2)}",
        "weight": "Lighter",
        "spacing": "None"
      }
      ]
    }
    ]
  }
  ```

  ![요리의 적응형 카드 미리 보기를 보여 주는 Visual Studio Code의 스크린샷.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-image-properties.png)

  알레르기 유발 물질을 표시하려면 함수를 사용하여 알레르기 유발 물질을 문자열로 연결해야 한다는 점에 유의합니다. 요리에 알레르기 유발 성분이 없는 경우 **없음**으로 표시됩니다. 가격 서식이 올바르게 지정되도록 하려면 카드에 표시할 소수점 수를 지정할 수 있는 **formatNumber** 함수를 사용합니다.

## 작업 3 - 주문 요약에 대한 적응형 카드 빌드

샘플 API를 사용하면 사용자가 메뉴를 찾아보고 주문을 할 수 있습니다. 주문 요약을 보여 주는 적응형 카드를 만들어 보겠습니다.

Visual Studio Code:

1. **cards** 폴더에 **order.json**이라는 새 파일을 만듭니다. 빈 적응형 카드를 나타내는 다음 내용을 붙여넣습니다.

  ```json
  {
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": []
  }
  ```

1. 적응형 카드에 대한 데이터 파일을 만듭니다.
  1. 키보드에서 <kbd>CTRL</kbd>+<kbd>P</kbd>(macOS의 경우 <kbd>CMD</kbd>+<kbd>P</kbd>) 키를 눌러 명령 팔레트를 엽니다. `>Adaptive`(을)를 입력하면 적응형 카드 작업과 관련된 명령을 찾을 수 있습니다.

    ![적응형 카드 작업과 관련된 명령을 보여 주는 Visual Studio Code의 스크린샷.](../media/LAB_03/3-visual-studio-code-adaptive-card-commands.png)

  1. 목록에서 **적응형 카드: 새 데이터 파일**을 선택합니다. Visual Studio Code는 **order.data.json**이라는 새 파일을 만듭니다.
  1. 해당 내용을 주문 요약을 나타내는 데이터로 교체합니다.

    ```json
    {
      "order_id": 6210,
      "status": "confirmed",
      "total_price": 25.48
    }
    ```

  1. 변경 내용을 저장합니다.
1. **order.json** 파일로 돌아갑니다.
1. 렌즈에서 **적응형 카드 미리 보기**를 선택합니다.
1. 다음으로 **order.json** 파일의 내용을 다음 코드로 바꿉니다.

  ```json
  {
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [
    {
      "type": "TextBlock",
      "text": "Order Confirmation 🤌",
      "size": "Large",
      "weight": "Bolder",
      "horizontalAlignment": "Center"
    },
    {
      "type": "Container",
      "items": [
      {
        "type": "TextBlock",
        "text": "Your order has been successfully placed!",
        "weight": "Bolder",
        "spacing": "Small"
      },
      {
        "type": "FactSet",
        "facts": [
        {
          "title": "Order ID:",
          "value": "${order_id} "
        },
        {
          "title": "Status:",
          "value": "${status}"
        },
        {
          "title": "Total Price:",
          "value": "€${formatNumber(total_price, 2)}"
        }
        ]
      }
      ]
    }
    ]
  }
  ```

  이전 섹션과 마찬가지로 카드의 각 요소를 데이터 속성에 매핑합니다.

  ![주문의 적응형 카드 미리 보기를 보여주는 Visual Studio Code의 스크린샷.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-order.png)

  > [!IMPORTANT]
  > **${order_id}** 의 후행 공백에 주목합니다. 이는 적응형 카드에서 숫자를 렌더링할 때 발생하는 알려진 문제로 인해 의도된 것입니다. 테스트하려면 공백을 제거하고 미리보기에서 숫자가 사라지는지 확인합니다.
  >
  > ![주문 번호가 없는 적응형 카드의 미리 보기를 보여 주는 Visual Studio Code의 스크린샷.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-no-number.png)

  카드가 제대로 표시되도록 후행 공백을 복원하고 변경 사항을 저장합니다.