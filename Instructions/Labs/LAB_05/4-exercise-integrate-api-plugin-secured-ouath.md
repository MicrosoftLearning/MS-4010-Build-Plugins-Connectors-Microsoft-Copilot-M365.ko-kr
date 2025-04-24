---
lab:
  title: 연습 3 - OAuth로 보호된 API와 API 플러그 인 통합
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# 연습 3 - OAuth로 보호된 API와 API 플러그 인 통합

Microsoft 365 Copilot용 API 플러그인을 사용하면 OAuth로 보호되는 API와 통합할 수 있습니다. API를 보호하는 앱의 클라이언트 ID와 비밀을 Teams 자격 증명 모음에 등록하여 안전하게 유지합니다. 런타임에 Microsoft 365 Copilot는 플러그 인을 실행하고, 자격 증명 모음에서 정보를 검색하고, 이를 사용하여 액세스 토큰을 가져오고 API를 호출합니다. 이 프로세스에 따라 클라이언트 ID와 비밀은 안전하게 유지되며 클라이언트에 노출되지 않습니다.

### 연습 기간

- **예상 완료 시간**: 10분

## 작업 1 - 샘플 프로젝트를 열고 인증 구성 검사

먼저 샘플 프로젝트를 다운로드합니다.

1. 웹 브라우저에서 [https://aka.ms/learn-da-api-ts-repairs](https://aka.ms/learn-da-api-ts-repairs)으로 이동합니다. 샘플 프로젝트와 함께 ZIP 파일을 다운로드하라는 메시지가 표시됩니다.
1. ZIP 파일을 컴퓨터에 저장합니다.
1. ZIP 파일의 압축을 풉니다.
1. Visual Studio Code에서 폴더를 엽니다.

샘플 프로젝트는 선언적 에이전트, API 플러그인 및 Microsoft Entra ID로 보호되는 API가 포함된 Teams 도구 키트 프로젝트입니다. 이 API는 Azure Functions에서 실행되며 Azure Functions의 기본 제공 인증 및 권한 부여 기능(Easy Auth라고도 함)을 사용하여 보안을 구현합니다.

## 작업 2 – OAuth2 권한 부여 구성 검사

계속하기 전에 샘플 프로젝트에서 OAuth2 권한 부여 구성을 검토합니다.

### API 정의 검사

먼저 프로젝트에 포함된 API 정의의 보안 구성을 살펴.

Visual Studio Code:

1. **appPackage/apiSpecificationFile/repair.yml** 파일을 엽니다.
1. **components.securitySchemes** 섹션에서 **oAuth2AuthCode** 속성을 확인합니다.

  ```yml
  components:
    securitySchemes:
    oAuth2AuthCode:
      type: oauth2
      description: OAuth configuration for the repair service
      flows:
      authorizationCode:
        authorizationUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/authorize
        tokenUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/token
        scopes:
        api://${{AAD_APP_CLIENT_ID}}/repairs_read: Read repair records 
  ```

  이 속성은 OAuth2 보안 체계를 정의하며 액세스 토큰을 얻기 위해 호출할 URL과 API가 사용하는 범위에 대한 정보를 포함합니다.

  > **중요**: 범위는 애플리케이션 ID URI **(api://...**)로 완전히 정규화된 상태입니다. Microsoft Entra로 작업할 때는 사용자 지정 범위를 완전히 정규화해야 합니다. Microsoft Entra에서 정규화되지 않은 범위가 표시되는 경우 Microsoft Graph에 속한다고 가정하므로 권한 부여 흐름 오류가 발생합니다.

1. **paths./repairs.get.security** 속성을 찾습니다. 클라이언트가 작업을 수행하는 데 필요한 **oAuth2AuthCode** 보안 체계와 범위를 참조한다는 점에 유의.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - oAuth2AuthCode:
        - api://${{AAD_APP_CLIENT_ID}}/repairs_read
  [...]
  ```

  > **중요** API 사양에 필요한 범위를 나열하는 것은 순전히 정보 제공을 위한 것입니다. API를 구현할 때는 토큰의 유효성을 검사하고 토큰에 필요한 범위가 포함되어 있는지 확인해야 합니다.

### API 구현 검사

다음으로, API 구현을 살펴보겠습니다.

Visual Studio Code:

1. **src/functions/repairs.ts** 파일을 엽니다.
1. **repairs** 처리기 함수에서 요청에 필요한 범위의 액세스 토큰이 포함되어 있는지 확인하는 다음 줄을 찾습니다.

  ```typescript
  if (!hasRequiredScopes(req, 'repairs_read')) {
    return {
    status: 403,
    body: "Insufficient permissions",
    };
  }
  ```

1. **hasRequiredScopes** 함수는 **repairs.ts** 파일에 추가로 구현되어 있습니다.

  ```typescript
  function hasRequiredScopes(req: HttpRequest, requiredScopes: string[] | string): boolean {
    if (typeof requiredScopes === 'string') {
    requiredScopes = [requiredScopes];
    }
  
    const token = req.headers.get("Authorization")?.split(" ");
    if (!token || token[0] !== "Bearer") {
    return false;
    }
  
    try {
    const decodedToken = jwtDecode<JwtPayload & { scp?: string }>(token[1]);
    const scopes = decodedToken.scp?.split(" ") ?? [];
    return requiredScopes.every(scope => scopes.includes(scope));
    }
    catch (error) {
    return false;
    }
  }
  ```

  이 함수는 권한 부여 요청 헤더에서 전달자 토큰을 추출하여 시작합니다. 다음으로, **jwt-decode** 패키지를 사용하여 토큰을 디코딩하고 **scp** 클레임에서 범위 목록을 가져옵니다. 마지막으로 **scp** 클레임에 필요한 모든 범위가 포함되어 있는지 확인합니다.

  함수가 액세스 토큰의 유효성을 검사하지 않습니다. 대신 액세스 토큰에 필요한 범위가 포함되어 있는지 확인합니다. 이 템플릿에서 API는 Azure Functions에서 실행되며 액세스 토큰의 유효성 검사를 담당하는 Easy Auth를 사용하여 보안을 구현합니다. 요청에 유효한 액세스 토큰이 포함되어 있지 않은 경우 Azure Functions 런타임은 코드에 도달하기 전에 요청을 거부합니다. 간편 인증은 토큰의 유효성을 검사하지만, 필요한 범위는 직접 확인하지 않으므로 사용자가 직접 확인해야 합니다.

### 자격 증명 모음 작업 구성 검사

이 프로젝트에서는 Teams 도구 키트를 사용하여 OAuth 정보를 자격 증명 모음에 추가합니다. Teams 도구 키트는 프로젝트 구성의 특수 작업을 사용하여 자격 증명 모음에 OAuth 정보를 등록합니다.

Visual Studio Code:

1. **./teampsapp.local.yml** 파일을 엽니다.
1. **프로비전** 섹션에서 **OAuth/register** 작업을 찾습니다.

  ```yml
  - uses: oauth/register
    with:
    name: oAuth2AuthCode
    flow: authorizationCode
    appId: ${{TEAMS_APP_ID}}
    clientId: ${{AAD_APP_CLIENT_ID}}
    clientSecret: ${{SECRET_AAD_APP_CLIENT_SECRET}}
    isPKCEEnabled: true
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    writeToEnvironmentFile:
    configurationId: OAUTH2AUTHCODE_CONFIGURATION_ID
  ```

  이 작업은 **env/.env.local** 및 **env/.env.local.user** 파일에 저장된 **TEAMS_APP_ID**, **AAD_APP_CLIENT_ID**, **SECRET_AAD_APP_CLIENT_SECRET** 프로젝트 변수의 값을 가져와 자격 증명 모음에 등록합니다. 또한 추가 보안 조치로 PKCE(코드 교환용 증명 키)를 사용할 수 있습니다. 그런 다음 자격 증명 모음 항목 ID를 가져와 환경 파일 **env/.env.local**에 씁니다. 이 작업의 결과는 **OAUTH2AUTHCODE_CONFIGURATION_ID**라는 환경 변수입니다. Teams 도구 키트는 이 변수 값을 플러그인 정의가 포함된 **appPackages/ai-plugin.json** 파일에 씁니다. 런타임에 API 플러그인을 로드하는 선언적 에이전트는 이 ID를 사용하여 자격 증명 모음에서 OAuth 정보를 검색하고, 액세스 토큰을 얻기 위해 흐름을 시작 및 인증합니다.

  > **중요** **oauth/register** 작업은 OAuth 정보가 아직 없는 경우에만 자격 증명 모음에 등록하는 역할을 담당합니다. 정보가 이미 있는 경우 Teams 도구 키트는 이 작업 실행을 건너뜁니다.

1. 다음으로 **OAuth/update** 작업을 찾습니다.

  ```yml
  - uses: oauth/update
    with:
    name: oAuth2AuthCode
    appId: ${{TEAMS_APP_ID}}
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    configurationId: ${{OAUTH2AUTHCODE_CONFIGURATION_ID}}
    isPKCEEnabled: true
  ```

  이 작업은 프로젝트와 동기화된 자격 증명 모음의 OAuth 정보를 유지합니다. 프로젝트가 제대로 작동해야 합니다. 주요 속성 중 하나는 API 플러그 인을 사용할 수 있는 URL입니다. 프로젝트를 시작할 때마다 Teams 도구 키트는 새 URL에 개발 터널을 엽니다. 자격 증명 모음의 OAuth 정보는 Copilot가 API에 도달할 수 있도록 이 URL을 참조해야 합니다.

### 인증 및 권한 부여 구성 검사

다음으로 살펴볼 부분은 Azure Functions의 인증 및 권한 부여 설정입니다. 이 연습의 API는 Azure Functions의 기본 제공 인증 및 권한 부여 기능을 사용합니다. Teams 도구 키트는 Azure Functions를 Azure에 프로비전하는 동안 이러한 기능을 구성합니다.

Visual Studio Code:

1. **infra/azure.bicep** 파일을 엽니다.
1. **인증 설정** 리소스를 찾습니다.

  ```bicep
  resource authSettings 'Microsoft.Web/sites/config@2021-02-01' = {
    parent: functionApp
    name: 'authsettingsV2'
    properties: {
    globalValidation: {
      requireAuthentication: true
      unauthenticatedClientAction: 'Return401'
    }
    identityProviders: {
      azureActiveDirectory: {
      enabled: true
      registration: {
        openIdIssuer: oauthAuthority
        clientId: aadAppClientId
      }
      validation: {
        allowedAudiences: [
        aadAppClientId
        aadApplicationIdUri
        ]
      }
      }
    }
    }
  }
  ```

  이 리소스를 사용하면 Azure Functions 앱에서 기본 제공 인증 및 권한 부여 기능을 사용할 수 있습니다. 먼저, **globalValidation** 섹션에서 앱이 인증된 요청만 허용하도록 정의합니다. 앱이 인증되지 않은 요청을 받으면 401 HTTP 오류로 거부합니다. 그런 다음 **identityProviders** 섹션에서 구성은 요청을 승인하는 데 Microsoft Entra ID(이전의 Azure Active Directory)를 사용하도록 정의합니다. API를 보호하기 위해 사용하는 Microsoft Entra 앱 등록과 API를 호출할 수 있는 대상 그룹을 지정합니다.

### Microsoft Entra 애플리케이션 등록 검사

마지막으로 살펴봐야 할 부분은 프로젝트에서 API를 보호하는 데 사용하는 Microsoft Entra 애플리케이션 등록입니다. OAuth를 사용하는 경우 애플리케이션을 사용하여 리소스에 대한 액세스를 보호합니다. 애플리케이션은 일반적으로 클라이언트 암호 또는 인증서와 같은 액세스 토큰을 가져오는 데 필요한 자격 증명을 정의합니다. 또한 클라이언트가 API를 호출할 때 요청할 수 있는 다양한 권한(범위라고도 함)을 지정합니다. Microsoft Entra 애플리케이션 등록은 Microsoft Cloud의 애플리케이션을 나타내며 OAuth 권한 부여 흐름에 사용할 애플리케이션을 정의합니다.

Visual Studio Code:

1. **./aad.manifest.json** 파일을 엽니다.
1. **oauth2Permissions** 속성을 찾습니다.

  ```json
  "oauth2Permissions": [
    {
    "adminConsentDescription": "Allows Copilot to read repair records on your behalf.",
    "adminConsentDisplayName": "Read repairs",
    "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
    "isEnabled": true,
    "type": "User",
    "userConsentDescription": "Allows Copilot to read repair records.",
    "userConsentDisplayName": "Read repairs",
    "value": "repairs_read"
    }
  ],
  ```

  이 속성은 **repairs_read**라는 사용자 지정 범위를 정의하여, 클라이언트에게 repairs API에서 수리 정보를 읽을 수 있는 권한을 부여합니다.

1. **identifierUris** 속성을 찾습니다.

  ```json
  "identifierUris": [
    "api://${{AAD_APP_CLIENT_ID}}"
  ]
  ```

  **identifierUris** 속성은 범위를 완전히 한정하는 데 사용되는 식별자를 정의합니다.
