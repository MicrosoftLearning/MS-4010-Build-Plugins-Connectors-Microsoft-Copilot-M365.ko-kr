---
lab:
  title: 연습 3 - 새 명령 추가
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft 365 Copilot'
---

# 연습 3 - 새 명령 추가

이 연습에서는 새 명령을 추가하여 Teams 메시지 확장 및 Copilot 플러그 인을 향상시킵니다. 현재 메시지 확장은 Northwind 인벤토리 데이터베이스 내의 제품에 대한 정보를 효과적으로 제공하지만 Northwind의 고객과 관련된 정보는 제공하지 않습니다. 사용자가 지정한 고객 이름으로 주문한 제품을 검색하는 API 호출과 연결된 새 명령을 소개합니다. 이 연습에서는 적어도 연습 1, 2 및 3을 완료한 것으로 가정합니다. Microsoft 365 Copilot 라이선스가 없는 경우 연습 4를 건너뛰어도 괜찮습니다.

이 작업을 완수하려면 다음 작업을 수행합니다.

1. Teams 앱 매니페스트를 수정하여 **메시지 확장/플러그 인 사용자 인터페이스를 확장합니다**. 여기에는 **"companySearch"** 라는 새 명령 도입이 포함됩니다. 메시지 확장의 UI는 적응형 카드인 반면, Copilot의 경우 Copilot 채팅의 텍스트 입력 및 출력입니다.

1. **'companySearch' 명령에 대한 처리기를 만듭니다**. 이렇게 하면 메시지 라우팅 코드에서 전달된 쿼리 문자열을 구문 분석하고 입력의 유효성을 검사하며 회사 API에서 제품 검색을 호출합니다. 또한 이 작업은 메시지 또는 Copilot 채팅 UI에 표시되는 반환된 제품 목록으로 적응형 카드를 채웁니다.

1. 명령 **라우팅** 코드를 업데이트하여 새 명령을 이전 작업에서 만든 처리기로 라우팅합니다. 사용자가 Northwind 데이터베이스(**handleTeamsMessagingExtensionQuery**)를 쿼리할 때 Bot Framework에서 호출하는 메서드를 확장하여 이 작업을 수행합니다. 

1. 해당 회사에서 주문한 제품 목록을 반환하는 **회사별 제품 검색을 구현합니다**.

1. **앱을 실행하고** 지정된 회사에서 구매한 제품을 검색합니다.

## 작업 1 - 메시지 확장/플러그 인 사용자 인터페이스 확장 

1. **작업 폴더**의 Visual Studio Code에서 **manifest.json**을 열고 `discountSearch` 명령(**98줄**) 바로 뒤에 JSON을 추가합니다. 이 추가 정보를 사용하여 플러그 인에서 지원하는 명령 목록을 정의하는 `commands`배열에 추가합니다.

   ```json
   {
       "id": "companySearch",
       "context": [
           "compose",
           "commandBox"
       ],
       "description": "Given a company name, search for products ordered by that company",
       "title": "Customer",
       "type": "query",
       "parameters": [
           {
               "name": "companyName",
               "title": "Company name",
               "description": "The company name to find products ordered by that company",
               "inputType": "text"
           }
       ]
   }
   ```

> [!NOTE] 
> **ID**는 UI와 코드 간의 연결입니다. 이 값은 **discount\product\SearchCommand.ts**파일에서 **COMMAND_ID**로 정의됩니다. 이러한 각 파일에 **ID** 값에 해당하는 고유한 **COMMAND_ID**를 부여하는 방법을 알아봅니다.

## 작업 2 - 'companySearch' 명령에 대한 처리기 만들기

이 연습에서는 기존 코드 중 일부를 복사하여 명령에 대한 새 처리기를 만듭니다. 

1. **작업 디렉터리** 아래의 Visual Studio Code에서 **.\src\messageExtensions**로 이동하여 '**productSearchCommand.ts**'를 복사하고 같은 폴더에 붙여넣어 복사본을 만듭니다. 이 파일의 이름을 **customerSearchCommand.ts**로 바꿉니다.

1. 7행을 다음으로 변경합니다.

    ```typescript
    import { searchProductsByCustomer } from "../northwindDB/products";
    ```

1. 10행을 다음으로 변경합니다.

   ```javascript
   const COMMAND_ID = "companySearch";
   ```



1. **handleTeamsMessagingExtensionQuery**의 콘텐츠를 다음으로 바꿉니다.

   ```javascript
    {
       let companyName;
   
       // Validate the incoming query, making sure it's the 'companySearch' command
       // The value of the 'companyName' parameter is the company name to search for
       if (query.parameters.length === 1 && query.parameters[0]?.name === "companyName") {
           [companyName] = (query.parameters[0]?.value.split(','));
       } else { 
           companyName = cleanupParam(query.parameters.find((element) => element.name === "companyName")?.value);
       }
       console.log(`🍽️ Query #${++queryCount}:\ncompanyName=${companyName}`);    
   
       const products = await searchProductsByCustomer(companyName);
   
       console.log(`Found ${products.length} products in the Northwind database`)
       const attachments = [];
       products.forEach((product) => {
           const preview = CardFactory.heroCard(product.ProductName,
               `Customer: ${companyName}`, [product.ImageUrl]);
   
           const resultCard = cardHandler.getEditCard(product);
           const attachment = { ...resultCard, preview };
           attachments.push(attachment);
       });
       return {
           composeExtension: {
               type: "result",
               attachmentLayout: "list",
               attachments: attachments,
           },
       };
   }
   ```

> [!NOTE]
> `searchProductsByCustomer`작업 4에서 구현합니다.

## 작업 3 - 명령 라우팅 업데이트

이 작업에서는 이전 작업에서 구현한 처리기로 `companySearch`명령을 라우팅합니다.

1. **searchApp.ts**를 열고 **10줄**에서 이를 찾습니다.

   ```javascript
   import discountedSearchCommand from "./messageExtensions/discountSearchCommand";
   ```

1. 이를 **11줄**로 추가합니다.

   ```javascript
   import customerSearchCommand from "./messageExtensions/customerSearchCommand";
   ```

1. 이 문 아래입니다.

   ```javascript
         case discountedSearchCommand.COMMAND_ID: {
           return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
         }
   ```

1. 이 문을 추가합니다.

   ```javascript
         case customerSearchCommand.COMMAND_ID: {
           return customerSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
         }
   ```

> [!NOTE]
> 플러그 인의 UI 기반 작업을 명시적으로 호출합니다. 그러나 Microsoft 365 Copilot에서 호출하면 Copilot 오케스트레이터에 의해 명령이 트리거됩니다.

## 작업 4 - 회사별 제품 검색 구현

이 작업에서는 **회사 이름으로** 제품 검색을 구현하고 회사의 주문 제품 목록을 반환합니다. 쿼리의 테이블 출력은 다음과 같습니다.

| 테이블         | Find        | 조회 기준    |
| ------------- | ----------- | ------------- |
| 고객      | 고객 ID | 고객 이름 |
| 주문        | 주문 ID    | 고객 ID   |
| OrderDetails   | Product     | 주문 ID      |

쿼리의 작동 방식은 다음과 같습니다. 

1. **Customer** 테이블을 사용하여 **Customer Name**으로 **Customer Id**를 찾습니다. 

1. **Customer Id**로 **Orders** 테이블을 쿼리하여 연결된 **Order Ids**를 검색합니다. 

1. 각 **Order Id**에 대해 **OrderDetail** 테이블에서 연결된 제품을 찾습니다. 

1. 마지막으로 지정된 회사 이름으로 주문한 제품 목록을 반환합니다.

이제 **products.ts** 파일을 수정하여 새 검색 쿼리를 추가할 수 있습니다.

1. **.\src\northwindDB\products.ts** 열기

1. OrderDetail, Order 및 Customer를 포함하도록 첫 줄의 `import` 문을 업데이트합니다. 아래와 같이 표시됩니다.

   ```javascript
   import {
       TABLE_NAME, Product, ProductEx, Supplier, Category, OrderDetail,
       Order, Customer
   } from './model';
   ```

1. 이 **8줄** 아래입니다.

   ```javascript
   import { getInventoryStatus } from '../adaptiveCards/utils';
   ```

       Add the new function `searchProductsByCustomer()`:

   ```javascript
   export async function searchProductsByCustomer(companyName: string): Promise<ProductEx[]> {

       let result = await getAllProductsEx();
   
       let customers = await loadReferenceData<Customer>(TABLE_NAME.CUSTOMER);
       let customerId="";
       for (const c in customers) {
           if (customers[c].CompanyName.toLowerCase().includes(companyName.toLowerCase())) {
               customerId = customers[c].CustomerID;
               break;
           }
       }
       
       if (customerId === "") 
           return [];

       let orders = await loadReferenceData<Order>(TABLE_NAME.ORDER);
       let orderdetails = await loadReferenceData<OrderDetail>(TABLE_NAME.ORDER_DETAIL);
       // build an array orders by customer id
       let customerOrders = [];
       for (const o in orders) {
           if (customerId === orders[o].CustomerID) {
               customerOrders.push(orders[o]);
           }
       }
       
       let customerOrdersDetails = [];
       // build an array order details customerOrders array
       for (const od in orderdetails) {
           for (const co in customerOrders) {
               if (customerOrders[co].OrderID === orderdetails[od].OrderID) {
                   customerOrdersDetails.push(orderdetails[od]);
               }
           }
       }

       // Filter products by the ProductID in the customerOrdersDetails array
       result = result.filter(product => 
           customerOrdersDetails.some(order => order.ProductID === product.ProductID)
       );

       return result;
   }
   ```

## 작업 5 - 앱 실행하기! 회사 이름으로 제품 검색

이제 Microsoft 365 Copilot용 플러그 인으로 샘플을 테스트할 준비가 되었습니다.

1. Teams에서 **Northwest 인벤토리** 앱을 제거합니다. 매니페스트를 업데이트하기 때문에 이 작업이 필요합니다. 매니페스트 업데이트를 사용하려면 앱을 다시 설치해야 합니다. 이 작업을 수행하는 가장 깔끔한 방법은 먼저 Teams에서 제거하는 것입니다.

    1. Teams 사이드바에서 세 개의 점(...) 1️⃣을 선택합니다. 애플리케이션 목록에 **Northwind 인벤토리** 2️⃣가 표시됩니다.

    1. **Northwest 인벤토리** 아이콘을 마우스 오른쪽 단추로 클릭하고 제거 3️⃣을 선택합니다.

        ![Northwind 인벤토리를 제거하는 방법의 스크린샷.](../media/3-01-uninstall-app.png)

1. **F5** 키를 눌러 **Teams(Edge)의 디버그** 프로필을 사용하여 Visual Studio Code에서 앱을 시작합니다.

1. Teams에서 **채팅**을 선택한 다음 **Copilot**를 선택합니다. Copilot가 가장 많은 옵션이어야합니다.

1. **플러그 인 아이콘**을 선택하고 **Northwind 인벤토리**를 선택하여 플러그 인을 사용하도록 설정합니다.

1. 프롬프트 를 입력합니다. 

   ```console
   What are the products ordered by 'Consolidated Holdings' in Northwind Inventory?
   ```

   터미널 출력은 Copilot가 쿼리를 이해하고 `companySearch` 명령을 실행하여 Copilot에서 추출한 회사 이름을 전달하는 것을 보여 줍니다.

   ![쿼리를 이해하고 'companySearch' 명령을 실행한 Copilot의 스크린샷.](../media/3-08-terminal-query-output.png)

   Copilot의 출력은 다음과 같습니다.

   ![명령에서 결과를 생성하는 Copilot의 스크린샷.](../media/3-07-response-customer-search.png)

다음은 시도할 다른 프롬프트입니다.

```console
What are the products ordered by 'Consolidated Holdings' in Northwind Inventory? Please list the product name, price and supplier in a table.
```

물론 이전 연습에서와 같이 샘플을 메시지 확장으로 사용하여 이 새 명령을 테스트할 수도 있습니다. 

1. Teams 사이드바에서 **채팅** 섹션으로 이동하여 채팅을 선택하거나 동료와 새 채팅을 시작합니다.

1. 앱 메뉴에 액세스할 기호**+** 를 선택합니다.

1. **Northwind 인벤토리** 앱을 선택합니다.

1. 이제 **Customer**라는 새 탭을 볼 수 있습니다.

1. **Consolidated Holdings**를 검색하고 이 회사에서 주문한 제품을 확인합니다. 이전 작업에서 Copilot가 반환한 것과 일치할 것입니다.

    ![메시지 확장으로 사용되는 새 명령의 스크린샷.](../media/3-08-customer-message-extension.png)

## 작업 확인

연습을 완료한 후에는 Northwind 인벤토리 앱에서 회사별로 주문을 검색하는 새 명령이 있어야 합니다. 또한 Copilot와 함께 플러그 인을 성공적으로 사용하고 다른 앱에서 메시지 확장으로 사용할 수 있어야 합니다. 

다음 연습에서는 플러그 인 소스 코드 및 적응형 카드를 탐색하여 앱을 빌드하는 방법과 추가로 사용자 지정하는 방법에 대해 자세히 알아봅니다.

[이 연습의 다음 단계를 계속 진행합니다...](./6-exercise-4-explore-plugin-source-code.md)