---
lab:
  title: 연습 1 - API 플러그 인을 키로 보호된 API와 통합
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# 연습 1 - API 플러그 인을 키로 보호된 API와 통합

Microsoft 365 Copilot용 API 플러그 인을 사용하면 키로 보호된 API와 통합할 수 있습니다. Teams 자격 증명 모음에 등록하여 API 키를 안전하게 유지합니다. 런타임 시 Microsoft 365 Copilot는 플러그 인을 실행하고, 자격 증명 모음에서 API 키를 검색하고, 이를 사용하여 API를 호출합니다. 이 프로세스에 따라 API 키는 안전하게 유지되며 클라이언트에 노출되지 않습니다.

이 연습에서는 생성한 API 키로 인증하는 API 플러그 인을 사용하여 새 선언적 에이전트를 만듭니다.

### 연습 기간

- **예상 완료 시간**: 10분

## 작업 1 - 새 프로젝트 만들기

Microsoft 365 Copilot용 새 API 플러그 인을 만드는 것으로 시작합니다. Visual Studio Code를 엽니다.

Visual Studio Code:

1. **활동 모음**(사이드 바)에서 Teams 도구 키트 확장을 활성화합니다.
1. **Teams 도구 키트** 확장 패널에서 **새 앱 만들기**를 선택합니다.
1. 프로젝트 템플릿 목록에서 **Copilot 에이전트**를 선택합니다.
1. 앱 기능 목록에서 **선언적 에이전트**를 선택합니다.
1. **플러그인 추가** 옵션을 선택합니다.
1. **새 API로 시작** 옵션을 선택합니다.
1. 인증 유형 목록에서 **API 키(전달자 토큰 인증)** 를 선택합니다.
1. 프로그래밍 언어로 **TypeScript**를 선택합니다.
1. 프로젝트를 저장할 폴더를 선택합니다.
1. 프로젝트 이름을 **da-repairs-key**로 지정합니다.

Teams 도구 키트는 선언적 에이전트, API 플러그 인 및 키로 보호된 API를 포함하는 새 프로젝트를 만듭니다.

## 작업 2 - API 키 인증 구성 검사

계속하기 전에 생성된 프로젝트에서 API 키 인증 구성을 검사합니다.

### API 정의 검사

먼저 API 정의에서 API 키 인증이 정의되는 방법을 살펴.

Visual Studio Code:

1. **appPackage/apiSpecificationFile/repair.yml** 파일을 엽니다. 이 파일에는 API에 대한 OpenAPI 정의가 포함되어 있습니다.
1. **components.securitySchemes** 섹션에서 **apiKey** 속성을 확인합니다.

  ```yml
  components:
    securitySchemes:
    apiKey:
      type: http
      scheme: bearer
  ```

  이 속성은 권한 요청 헤더에서 API 키를 전달자 토큰으로 사용하는 보안 체계를 정의합니다.

1. **paths./repairs.get.security** 속성을 찾습니다. **apiKey** 보안 체계를 참조한다는 점에 유의.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - apiKey: []
  [...] 
  ```

### API 구현 검사

다음으로, API가 각 요청에서 API 키의 유효성을 검사하는 방법을 알아봅니다.

Visual Studio Code:

1. **src/functions/repairs.ts** 파일을 엽니다.
1. **repairs** 처리기 함수에서 승인되지 않은 모든 요청을 거부하는 다음 줄을 찾습니다.

  ```typescript
  if (!isApiKeyValid(req)) {
    // Return 401 Unauthorized response.
    return {
    status: 401,
    };
  } 
  ```

1. **isApiKeyValid** 함수는 repairs.ts 파일에 추가로 구현되어 있습니다.

  ```typescript
  function isApiKeyValid(req: HttpRequest): boolean {
    const apiKey = req.headers.get("Authorization")?.replace("Bearer ", "").trim();
    return apiKey === process.env.API_KEY;
  }
  ```

  이 함수는 인증 헤더에 전달자 토큰이 포함되어 있는지 확인하고 **API_KEY** 환경 변수에 정의된 API 키와 비교합니다.

이 코드는 API 키 보안의 간단한 구현을 보여 주지만 실제로 API 키 보안이 어떻게 작동하는지 보여줍니다.

### 자격 증명 모음 작업 구성 검사

이 프로젝트에서는 Teams 도구 키트를 사용하여 자격 증명 모음에 API 키를 추가합니다. Teams 도구 키트는 프로젝트 구성에서 특수 작업을 사용하여 자격 증명 모음에 API 키를 등록합니다.

Visual Studio Code:

1. **./teampsapp.local.yml** 파일을 엽니다.
1. **프로비전** 섹션에서 **apiKey/register** 작업을 찾습니다.

  ```yml
  # Register API KEY
  - uses: apiKey/register
    with:
    # Name of the API Key
    name: apiKey
    # Value of the API Key
    primaryClientSecret: ${{SECRET_API_KEY}}
    # Teams app ID
    appId: ${{TEAMS_APP_ID}}
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    # Write the registration information of API Key into environment file for
    # the specified environment variable(s).
    writeToEnvironmentFile:
    registrationId: APIKEY_REGISTRATION_ID
  ```

  이 작업은 **env/.env.local.user** 파일에 저장된 **SECRET_API_KEY** 프로젝트 변수 값을 가져와서 자격 증명 모음에 등록합니다. 그런 다음 자격 증명 모음 항목 ID를 가져와 환경 파일 **env/.env.local**에 씁니다. 이 작업의 결과는 **APIKEY_REGISTRATION_ID**라는 환경 변수입니다. Teams 도구 키트는 이 변수 값을 플러그인 정의가 포함된 **appPackages/ai-plugin.json** 파일에 씁니다. 런타임에 API 플러그 인을 로드하는 선언적 에이전트는 이 ID를 사용하여 자격 증명 모음에서 API 키를 검색하고 API를 안전하게 호출합니다.

## 작업 3 - 로컬 개발을 위한 API 키 구성

프로젝트를 테스트하려면 API에 대한 API 키를 정의해야 합니다. 그런 다음, API 키를 자격 증명 모음에 저장하고 API 플러그 인에 자격 증명 모음 항목 ID를 기록합니다. 로컬 개발의 경우 프로젝트에 API 키를 저장하고 Teams 도구 키트를 사용하여 자격 증명 모음에 등록합니다.

Visual Studio Code:

1. **터미널** 창을 엽니다(Ctrl + `).
1. 명령줄
  1. `npm install`을(를) 실행하여 프로젝트의 종속성을 복원합니다.
  1. `npm run keygen`을(를) 실행하여 새 API 키를 생성합니다.
  1. 생성된 키를 클립보드에 복사합니다.
1. **env/.env.local.user** 파일을 엽니다.
1. **SECRET_API_KEY** 속성을 새로 생성된 API 키로 업데이트합니다. 업데이트된 속성은 다음과 같습니다.

  ```text
  SECRET_API_KEY='your_key'
  ```

1. 변경 내용을 저장합니다.

프로젝트를 빌드할 때마다 Teams 도구 키트는 자격 증명 모음의 API 키를 자동으로 업데이트하고 자격 증명 모음 항목 ID로 프로젝트를 업데이트합니다.