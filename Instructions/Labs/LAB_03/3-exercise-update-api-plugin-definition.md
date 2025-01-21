---
lab:
  title: 연습 2 - API 플러그 인 정의 업데이트
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# 연습 2 - API 플러그 인 정의 업데이트

다음 단계는 Copilot이 API의 데이터를 사용자에게 표시하는 데 사용해야 하는 적응형 카드로 API 플러그 인 정의를 업데이트하는 것입니다.

### 연습 기간

- **예상 완료 시간**: 10분

## 작업 1 - 요리 표시에 적응형 카드 추가

Visual Studio Code:

1. **cards/dish.json** 파일을 열고 내용을 복사합니다.
1. **appPackage/ai-plugin.json** 파일을 엽니다.
1. **functions.getDishes.capabilities.response_semantics** 속성에 **static_template**라는 새 속성을 추가하고 **body** 값을 **dish.json**의 내용에 설정합니다.
1. 완성된 코드 조각은 다음과 같습니다.

    ```json
    "static_template": {
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

1. 변경 내용을 저장합니다.

## 작업 2 - 주문 요약을 표시하는 적응형 카드 템플릿 추가

Visual Studio Code:

1. **cards/order.json** 파일을 열고 내용을 복사합니다.
1. **appPackage/ai-plugin.json** 파일을 엽니다.
1. **functions.placeOrder.capabilities.response_semantics** 속성에 **static_template**이라는 새 속성을 추가하고 해당 내용을 적응형 카드로 설정합니다.
1. 전체 파일은 다음과 같습니다.

    ```json
    "static_template": {
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

1. 변경 내용을 저장합니다.
