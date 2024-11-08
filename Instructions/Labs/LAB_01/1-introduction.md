---
lab:
  title: 소개
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 소개

메시지 확장을 사용하면 사용자가 Microsoft Teams 및 Microsoft Outlook의 외부 시스템을 사용하여 작업할 수 있습니다. 사용자는 메시지 확장을 사용하여 데이터를 조회 및 변경하고 메시지 및 이메일에서 이러한 시스템의 정보를 서식 있는 카드로 공유할 수 있습니다.

현재 조직과 관련된 제품 정보에 액세스하는 데 사용하는 사용자 지정 API가 있다고 가정합니다. Microsoft 365에서 이 정보를 검색하고 공유하려고 합니다. 또한 Microsoft 365용 Copilot에서 이 정보를 답변에 사용하려고 합니다.

이 모듈에서는 메시지 확장을 만듭니다. 메시지 확장은 봇을 사용하여 Microsoft Teams, Microsoft Outlook 및 Microsoft 365용 Copilot과 통신합니다.

![Microsoft Teams의 검색 기반 메시지 확장 프로그램에서 반환된 검색 결과의 스크린샷](../media/1-search-results.png)

Microsoft Entra를 통해 사용자를 인증하므로 사용자 대신 API에서 데이터를 반환할 수 있습니다.

사용자가 인증하면 메시지 확장 프로그램은 API에서 데이터를 가져와 메시지 및 이메일에 서식 있는 카드로 포함할 수 있는 검색 결과를 반환한 다음 공유합니다.

![Microsoft Teams의 외부 API에서 데이터를 사용하는 검색 결과의 스크린샷](../media/3-search-results-api.png)

![Microsoft Teams의 메시지에 포함된 검색 결과의 스크린샷입니다.](../media/4-adaptive-card.png)

Microsoft 365용 Copilot을 플러그 인으로 사용하여 사용자 대신 제품 데이터를 쿼리하고 반환된 데이터를 답변에 사용할 수 있습니다.

![메시지 확장 플러그 인에서 반환된 정보를 포함하는 Microsoft 365용 Copilot의 답변 스크린샷입니다. 제품 정보를 보여 주는 적응형 카드가 표시됩니다.](../media/5-copilot-answer.png)

이 모듈을 마치면 C#(.NET에서 실행)으로 작성된 메시지 확장을 만들 수 있습니다. Microsoft Teams, Microsoft Outlook 및 Microsoft 365용 Copilot에서 사용할 수 있습니다. 보호된 API 뒤에 있는 데이터를 쿼리하고 결과를 서식 있는 카드로 반환할 수 있습니다.

## 필수 조건

- C#에 대한 기본 지식
- Bicep에 대한 기본 지식
- 인증에 대한 기본 지식
- Microsoft 365 테넌트에 대한 관리자 액세스
- Azure 구독에 대한 액세스
- Microsoft 365용 Copilot에 대한 액세스는 선택 사항이며 **연습 4: 작업 5**를 완료하는 데만 필요합니다.
- [Teams 도구 키트](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs)(Microsoft Teams 개발 도구 구성 요소)가 설치된 Visual Studio 2022 17.10
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)
- [개발 프록시 0.19.1 이상](https://aka.ms/devproxy)

> [!NOTE]
> 이 랩에서 Microsoft 365 Copilot 라이선스가 필요한 연습은 **연습 4: 작업 5**뿐입니다. 테넌트에 Copilot이 있는지 여부에 관계없이 해당 시점까지 모든 작업을 수행해야 합니다.

## 랩 기간

  - **예상 완료 시간**: 150분

## 학습 목표

이 모듈을 마치면 다음을 수행할 수 있습니다.

- 메시지 확장이란 무엇이며 이를 빌드하는 방법을 이해합니다.
- 메시징 확장을 만듭니다.
- Single Sign-On을 사용하여 사용자를 인증하고 Microsoft Entra 인증으로 보호되는 사용자 지정 API를 호출하는 방법을 이해합니다.
- Microsoft 365용 Copilot에서 사용할 수 있도록 메시지 확장을 확장하고 최적화하는 방법을 .

시작 준비가 되면 [첫 번째 연습을 진행합니다.](./2-exercise-create-a-message-extension.md)
