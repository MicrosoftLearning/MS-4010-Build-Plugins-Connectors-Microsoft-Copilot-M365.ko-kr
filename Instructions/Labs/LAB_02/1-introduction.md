---
lab:
  title: 소개
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft 365 Copilot'
---

# 소개

이 프로젝트에서는 Microsoft 365 Microsoft Copilot에서 Teams 메시지 확장을 플러그 인으로 사용하는 방법을 알아봅니다. 이 프로젝트는 동일한 [GitHub 리포지토리](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/tree/main/samples/msgext-northwind-inventory-ts)에 포함된 "Northwind 인벤토리" 샘플을 기반으로 합니다. 유서 깊은 [Northwind 데이터베이스](https://learn.microsoft.com/dotnet/framework/data/adonet/sql/linq/downloading-sample-databases)를 사용하면 많은 시뮬레이션된 엔터프라이즈 데이터를 사용할 수 있습니다.

Northwind는 워싱턴 주 스포캔에서 식품 전문 전자상거래 비즈니스를 운영하고 있습니다. 이 랩에서는 제품 인벤토리 및 재무 정보에 대한 액세스를 제공하는 Northwind 인벤토리 애플리케이션으로 작업합니다.

이 연습을 완료하는 데 약 **60**분 정도 소요됩니다.

## 시작하기 전에

- 먼저 개발 환경을 설정하고 애플리케이션을 실행하여 [**준비**](./2-prepare-development-environment.md)합니다.

- [**연습 1**](./3-exercise-1-run-message-extension.md)에서는 Microsoft Teams 및 Outlook에서 [메시지 확장](https://learn.microsoft.com/microsoftteams/platform/messaging-extensions/what-are-messaging-extensions)과 동일한 애플리케이션을 실행합니다.

- [**연습 2**](./4-exercise-2-run-copilot-plugin.md)에서는 Microsoft 365 Copilot용 플러그 인으로 애플리케이션을 실행합니다. 다양한 프롬프트로 실험하고 여러 매개 변수를 사용하여 플러그인이 호출되는 방식을 관찰합니다. Copilot와 채팅할 때 개발자 콘솔에서 만들고 있는 쿼리를 볼 수 있습니다.

- [**연습 3**](./5-exercise-3-add-new-command.md)에서는 플러그 인 기능을 확장하고 더 많은 작업을 수행할 수 있도록 애플리케이션에 새 명령을 추가하는 방법을 알아봅니다.

  ![제품을 표시하는 적응형 카드의 스크린샷.](../media/1-00-product-card-only.png)

- 마지막으로, [**연습 4**](./6-exercise-4-explore-plugin-source-code.md)에서는 코드 둘러보기를 통해 코드가 어떻게 작동하는지 좀 더 자세히 알아봅니다. 아직 Copilot가 없는 경우 다른 모든 항목은 Microsoft 365의 메시지 확장으로 계속 작동합니다.

시작 준비가 되면 [다음 연습을 계속 진행합니다.](./2-prepare-development-environment.md)