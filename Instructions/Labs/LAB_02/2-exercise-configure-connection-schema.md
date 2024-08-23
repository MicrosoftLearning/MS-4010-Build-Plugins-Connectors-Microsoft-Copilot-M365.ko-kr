---
lab:
  title: 연습 1 - 외부 연결 구성 및 스키마 배포
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 연습 - 외부 연결 구성 및 스키마 배포

이 연습에서는 사용자 지정 Microsoft Graph 커넥터를 콘솔 애플리케이션으로 빌드합니다. 새 Microsoft Entra 앱 등록을 등록하고 코드를 추가하여 외부 연결을 만들고 해당 스키마를 배포합니다.

## 시작하기 전에

이 연습을 완료하는 데 약 **15분**이 소요됩니다.

## 작업 1 - 새 Microsoft Entra 앱 등록

먼저 사용자 지정 Graph 커넥터가 Microsoft 365로 인증하는 데 사용하는 새 Entra 앱 등록을 등록합니다.

웹 브라우저에서:

1. [https://portal.azure.com](https://portal.azure.com)에서 **Azure Portal**로 이동합니다.
1. 탐색에서 **Microsoft Entra ID** 아래의 **보기**를 선택합니다.
1. 측면 탐색에서 **관리**를 펼쳐 **앱 등록**을 선택합니다.
1. 상단 탐색 창에서 **새 등록**을 선택합니다.
1. 다음 값 중 하나를 지정합니다.
   1. **이름:** MSGraph docs Graph 커넥터
   1. **지원되는 계정 유형**: 이 조직 디렉터리의 계정만(단일 테넌트)
1. **등록**을 선택하여 항목을 확인합니다.
1. 개요 화면에서 **애플리케이션 ID** 및 **디렉터리(테넌트) ID** 속성의 값을 복사합니다. 나중에 필요하므로 잘 기억해 두세요.

## 작업 2 - 자격 증명 만들기

이 사용자 지정 Graph 커넥터는 사용자 상호 작용 없이 실행되므로 자동으로 인증하도록 구성해야 합니다. 간단히 하기 위해 암호를 만듭니다.

웹 브라우저에서 계속합니다:

1. 측면 탐색에서 **관리**를 펼쳐 **인증서 및 암호**를 선택합니다.
1. **클라이언트 암호** 탭을 선택한 다음, **새 클라이언트 암호**를 선택합니다.
1. **MSGraph docs Graph 커넥터 암호**에 대한 **설명**을 입력합니다.
1. **추가**를 선택하여 암호를 만듭니다.
1. 새로 만든 암호의 **값**을 복사합니다. 나중에 필요합니다.

## 작업 3 - API 권한 부여

Entra 앱 등록을 구성하는 마지막 단계는 외부 연결 및 스키마를 만들 수 있도록 API 권한을 부여하는 것입니다.

웹 브라우저에서 계속합니다:

1. 측면 탐색 창에서 **API 권한**을 선택합니다.
1. **권한 추가**를 선택합니다.
1. API 목록에서 **Microsoft Graph**를 선택합니다.
1. 다음으로 **애플리케이션 권한**을 선택합니다.
1. 필터 텍스트 상자에 **외부**를 입력합니다.
1. **ExternalConnection** 섹션을 확장하고 **ExternalConnection.ReadWrite.OwnedBy** 권한을 선택합니다.
1. **ExternalItem** 섹션을 확장하고 **ExternalItem.ReadWrite.OwnedBy** 권한을 선택합니다.
1. 선택 사항을 확인하려면 **권한 추가** 버튼을 선택합니다.
1. 구성을 완료하려면 **(테넌트)에 대한 관리자 동의 부여** 버튼을 선택하여 관리자 동의를 부여합니다.
1. **예**를 선택하여 대화 상자를 확인합니다.

## 작업 4 - 새 콘솔 앱 만들기 및 종속성 설치

Entra 앱 등록을 구성한 후 다음 단계는 Graph 커넥터의 코드를 구현하는 콘솔 앱을 만드는 것입니다.

Windows 터미널을 열어 새 콘솔 애플리케이션을 만듭니다.

1. 입력하여 새 폴더를 만든 다음, `mkdir documents\console_app`입력하여 새 폴더로 이동합니다.`cd .\documents\console_app`
1. `dotnet new console`을 실행하여 새 콘솔 애플리케이션을 만듭니다.
1. 커넥터를 빌드하는 데 필요한 종속성을 추가합니다.
   1. 패키지 원본으로 Nuget.org 추가하고 실행합니다. `dotnet nuget add source https://api.nuget.org/v3/index.json -n nuget.org` 
   1. Microsoft 365를 사용하여 인증하는 데 필요한 라이브러리를 추가하려면 `dotnet add package Azure.Identity`를 실행합니다.
   1. Graph API와 통신할 클라이언트 라이브러리를 추가하려면 `dotnet add package Microsoft.Graph`를 실행합니다.
   1. 다음 단계에서 구성할 사용자 암호를 사용하는 데 필요한 라이브러리를 추가하려면 `dotnet add package Microsoft.Extensions.Configuration.UserSecrets`를 실행합니다.
   1. Graph 커넥터는 외부 연결을 만들고 스키마를 배포하는 명령과 콘텐츠를 가져오는 명령 두 가지를 사용하여 콘솔 앱으로 구현합니다. 앱에서 명령어 정의를 지원하려면 `dotnet add package System.CommandLine --prerelease`을 실행하세요.

## 작업 5 - Entra 앱 등록 정보를 안전하게 저장하기

Entra 앱 등록을 생성한 후 애플리케이션 및 테넌트 ID, 비밀번호와 같은 정보를 기록했습니다. 커넥터에서 이 정보를 사용하여 Microsoft 365로 인증합니다. 코드에서 사용할 수 있도록 하려면 프로젝트에 사용자 비밀로 안전하게 저장합니다.

터미널에서:

1. 작업 디렉터리가 새로 만든 콘솔 앱으로 설정되어 있는지 확인합니다.
1. 사용자 비밀번호를 시작하려면 `dotnet user-secrets init`을 실행합니다.
1. 앱 등록에 대한 정보를 안전하게 저장하려면 토큰을 이전에 복사한 실제 값으로 바꾸고 실행합니다:

   ```dotnetcli
   dotnet user-secrets set "EntraId:ClientId" "[application id]"
   dotnet user-secrets set "EntraId:ClientSecret" "[secret value]"
   dotnet user-secrets set "EntraId:TenantId" "[directory (tenant) id]"
   ```

## 작업 6 - Microsoft Graph 클라이언트 만들기

사용자 지정 Graph 커넥터는 Microsoft Graph API를 사용하여 외부 연결 및 아이템을 관리합니다. 프로젝트에 설치한 **Microsoft.Graph** NuGet 패키지에서 `GraphServiceClient` 클래스의 인스턴스를 생성하여 시작합니다.

1. Visual Studio 2022에서 프로젝트를 엽니다.
1. 프로젝트에 ** GraphService.cs** 라는 새 코드 파일을 추가합니다.
1. 파일에 다음을 추가하여 사용할 네임스페이스에 대한 참조를 추가하는 것으로 시작합니다:

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   ```

1. 다음으로 GraphService라는 새 클래스를 정의합니다:

   ```csharp
   class GraphService
   {
   }
   ```

1. `GraphService` 클래스에서 `GraphServiceClient` 인스턴스를 저장하기 위한 싱글톤을 정의하여 Microsoft Graph API와 통신합니다:

   ```csharp
   class GraphService
   {
     static GraphServiceClient? _client;

     public static GraphServiceClient Client
     {
       get
       {
         // TODO: implement
       }
     }
   }
   ```

1. `Client` 싱글톤에서 아직 존재하지 않는 경우 `GraphServiceClient`의 새 인스턴스를 생성하도록 게터를 구현합니다.

   ```csharp
   public static GraphServiceClient Client
   {
     get
     {
       if (_client is null)
       {
         // TODO: implement
       }
       return _client;
     }
   }
   ```

1. 내부에 들어가면 이전에 저장한 Entra 앱 등록 정보를 자격 증명으로 사용하여 `GraphServiceClient`의 새 인스턴스를 생성합니다.

   ```csharp
   var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
   var config = builder.Build();

   var clientId = config["EntraId:ClientId"];
   var clientSecret = config["EntraId:ClientSecret"];
   var tenantId = config["EntraId:TenantId"];

   var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
   _client = new GraphServiceClient(credential);
   ```

   먼저 구성 빌더를 생성하여 사용자 비밀번호에 저장된 Entra 앱 등록 정보에 액세스합니다. 다음으로 작성기를 사용하여 앱 등록 정보를 검색합니다. 그런 다음 테넌트 및 클라이언트 ID와 클라이언트 암호 전달하는 새 클라이언트 암호 자격 증명을 만듭니다. 마지막으로 새로 만든 자격 증명을 전달하는 `GraphServiceClient` 인스턴스를 만듭니다.

1. 전체 코드는 다음과 같습니다:

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   
   class GraphService
   {
     static GraphServiceClient? _client;
   
     public static GraphServiceClient Client
     {
       get
       {
         if (_client is null)
         {
           var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
           var config = builder.Build();
     
           var clientId = config["EntraId:ClientId"];
           var clientSecret = config["EntraId:ClientSecret"];
           var tenantId = config["EntraId:TenantId"];
           
           var credential = new ClientSecretCredential(tenantId, clientId,    clientSecret);
           _client = new GraphServiceClient(credential);
         }
   
         return _client;
       }
     }
   }
   ```

1. 변경 내용을 저장합니다.

## 작업 7 - 외부 연결 및 스키마 구성 정의

다음 단계는 Graph 커넥터가 사용해야 하는 외부 연결 및 스키마를 정의하는 것입니다. 커넥터의 코드는 여러 위치에서 외부 연결의 ID에 액세스해야 하므로 코드의 중앙 위치에 저장하세요.

코드 편집기에서:

1. ** ConnectionConfiguration.cs** 라는 새 파일을 만듭니다.
1. Microsoft Graph 모델을 사용하여 네임스페이스에 참조를 추가합니다:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. 그런 다음 같은 파일에서 `ConnectionConfiguration`이라는 새 정적 클래스를 정의합니다:

   ```csharp
   static class ConnectionConfiguration
   {
   
   }
   ```

1. `ConnectionConfiguration` 클래스에서 `ExternalConnection`라는 새 속성을 추가합니다. 이를 구현하여 `ExternalConnection` Microsoft Graph 모델의 인스턴스를 반환합니다:

   ```csharp
   public static ExternalConnection ExternalConnection
   {
     get
     {
       return new ExternalConnection
       {
         Id = "msgraphdocs",
         Name = "Microsoft Graph documentation",
         Description = "Documentation for Microsoft Graph API which explains what Microsoft Graph is and how to use it."
       };
     }
   }
   ```

1. 다음으로 `Schema`이라는 속성을 하나 더 추가합니다. 이를 구현하여 `Schema` Graph 모델의 인스턴스를 반환합니다:

   ```csharp
   public static Schema Schema
   {
     get
     {
       return new Schema
       {
         BaseType = "microsoft.graph.externalItem",
         Properties = new()
         {
           // TODO: implement
         }
       };
     }
   }
   ```

1. 스키마에서 커넥터를 사용하여 수집하는 모든 외부 항목에 대해 추적하는 속성을 정의합니다:

   ```csharp
   new Property
   {
     Name = "title",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true,
     Labels = new() { Label.Title }
   },
   new Property
   {
     Name = "description",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true
   },
   new Property
   {
     Name = "iconUrl",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.IconUrl }
   },
   new Property
   {
     Name = "url",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.Url }
   }
   ```

   먼저 Microsoft 365로 가져온 외부 항목의 제목을 저장하는 ** title** 속성을 정의하는 것으로 시작합니다. 항목의 제목은 전체 텍스트 색인(`IsSearchable = true`)의 일부입니다. 사용자는 키워드 쿼리(`IsQueryable = true`)에서 해당 콘텐츠를 명시적으로 쿼리할 수도 있습니다. 제목을 검색하여 검색 결과에 표시할 수도 있습니다(`IsRetrievable = true`). ** title** 속성은 항목의 제목을 나타내며, `Title` 시맨틱 레이블을 사용하여 지정합니다.

   다음으로 외부 항목의 내용 요약을 저장하는 ** description** 속성을 정의합니다. 그 정의는 제목과 비슷합니다. 그러나 설명에 대한 의미론적 레이블이 없기 때문에 이를 정의하지 않습니다.

   다음으로 각 항목의 아이콘 URL을 저장할 속성을 정의합니다. Microsoft 365용 Copilot에는 이 속성이 필요하며 `IconUrl` 시맨틱 레이블을 사용하여 매핑해야 합니다.

   마지막으로 외부 항목의 원본 URL을 저장하는 ** URL** 속성을 정의합니다. 사용자는 이 URL을 사용하여 검색 결과 또는 Microsoft 365의 Copilot에서 외부 항목으로 이동합니다. URL은 Microsoft 365용 Copilot에 필요한 속성 중 하나이므로 `Url` 시맨틱 레이블을 사용하여 매핑합니다.

1. 전체 코드는 다음과 같습니다:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionConfiguration
   {
     public static ExternalConnection ExternalConnection
     {
       get
       {
         return new ExternalConnection
         {
           Id = "msgraphdocs",
           Name = "Microsoft Graph documentation",
           Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
         };
       }
     }
     public static Schema Schema
     {
       get
       {
         return new Schema
         {
           BaseType = "microsoft.graph.externalItem",
           Properties = new()
           {
             new Property
             {
               Name = "title",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true,
               Labels = new() { Label.Title }
             },
             new Property
             {
               Name = "description",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true
             },
             new Property
             {
               Name = "iconUrl",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.IconUrl }
             },
             new Property
             {
               Name = "url",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.Url }
             }
           }
         };
       }
     }
   }
   ```

1. 변경 내용을 저장합니다.

## 작업 8 - 외부 연결 만들기

이전 섹션에서 정의한 외부 연결에 대한 정보를 사용하여 Microsoft 365에서 외부 연결을 만드는 코드를 계속 추가합니다.

코드 편집기에서:

1. ** ConnectionService.cs**라는 새 파일을 만듭니다.
1. 파일에서 네임스페이스에 대한 참조를 추가하는 것으로 시작합니다:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. 그런 다음 같은 파일에서 `ConnectionService`이라는 새 정적 클래스를 정의합니다:

   ```csharp
   static class ConnectionService
   {
   
   }
   ```

1. `ConnectionService` 클래스에서 `CreateConnection`이라는 새 메서드를 추가합니다:

   ```csharp
   async static Task CreateConnection()
   {

   }
   ```

1. `CreateConnection` 메서드에서 Microsoft Graph 클라이언트 인스턴스를 사용하여 이전에 정의한 연결 정보를 사용하여 Microsoft Graph API를 호출하고 외부 연결을 생성합니다:

   ```csharp
   Console.Write("Creating connection...");
   
   await GraphService.Client.External.Connections
     .PostAsync(ConnectionConfiguration.ExternalConnection);
   
   Console.WriteLine("DONE");
   ```

1. 전체 코드는 다음과 같습니다:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   }
   ```

1. 변경 내용을 저장합니다.

이 코드를 테스트하기 전에 스키마를 생성하는 코드를 추가하세요. 이렇게 하면 외부 연결 생성 및 구성의 전체 흐름을 테스트할 수 있습니다.

## 작업 9 - 외부 연결 스키마 만들기

외부 연결 생성의 마지막 단계는 스키마를 만드는 것입니다.

코드 편집기에서:

1. ** ConnectionService.cs** 파일을 엽니다.
1. ConnectionService 클래스에서 `CreateSchema`라는 새 메서드를 추가합니다:

   ```csharp
   async static Task CreateSchema()
   {
   }
   ```

1. `CreateSchema` 메서드에서 Microsoft Graph 클라이언트 인스턴스를 사용하여 Microsoft Graph API를 호출하여 스키마를 생성합니다. 그런 다음 생성을 기다립니다.

   ```csharp
   Console.WriteLine("Creating schema...");
   
   await GraphService.Client.External
     .Connections[ConnectionConfiguration.ExternalConnection.Id]
     .Schema
     .PatchAsync(ConnectionConfiguration.Schema);
   
   do
   {
     var externalConnection = await GraphService.Client.External
       .Connections[ConnectionConfiguration.ExternalConnection.Id]
       .GetAsync();
   
     Console.Write($"State: {externalConnection?.State.ToString()}");
   
     if (externalConnection?.State != ConnectionState.Draft)
     {
       Console.WriteLine();
       break;
     }
   
     Console.WriteLine($". Waiting 60s...");
   
     await Task.Delay(60_000);
   }
   while (true);
   
   Console.WriteLine("DONE");
   ```

1. 같은 파일에 `ProvisionConnection`이라는 새 메서드를 추가합니다. 해당 코드에서 이전에 정의한 `CreateConnection` 및 `CreateSchema` 메서드를 호출합니다:

   ```csharp
   public static async Task ProvisionConnection()
   {
     try
     {
       await CreateConnection();
       await CreateSchema();
     }
     catch (Exception ex)
     {
       Console.WriteLine(ex.Message);
     }
   }
   ```

1. 전체 코드는 다음과 같습니다:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   
     async static Task CreateSchema()
     {
       Console.WriteLine("Creating schema...");
   
       await GraphService.Client.External
         .Connections[ConnectionConfiguration.ExternalConnection.Id]
         .Schema
         .PatchAsync(ConnectionConfiguration.Schema);
   
       do
       {
         var externalConnection = await GraphService.Client.External
           .Connections[ConnectionConfiguration.ExternalConnection.Id]
           .GetAsync();
   
         Console.Write($"State: {externalConnection?.State.ToString()}");
   
         if (externalConnection?.State != ConnectionState.Draft)
         {
           Console.WriteLine();
           break;
         }
   
         Console.WriteLine($". Waiting 60s...");
   
         await Task.Delay(60_000);
       }
       while (true);
   
       Console.WriteLine("DONE");
     }
   
     public static async Task ProvisionConnection()
     {
       try
       {
         await CreateConnection();
         await CreateSchema();
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

1. 변경 내용을 저장합니다.

## 작업 10 - 코드 테스트

남은 마지막 단계는 애플리케이션에서 연결과 스키마를 생성할 진입점을 만드는 것입니다. 명령줄에서 애플리케이션을 시작하여 호출하는 명령을 생성하여 이를 수행합니다.

코드 편집기에서:

1. **Program.cs** 파일을 엽니다.
1. 해당 내용을 다음 코드로 바꿉니다.

   ```csharp
   using System.CommandLine;
   
   var provisionConnectionCommand = new Command("provision-connection",    "Provisions external connection");
   provisionConnectionCommand.SetHandler(ConnectionService.   ProvisionConnection);
   
   var rootCommand = new RootCommand();
   rootCommand.AddCommand(provisionConnectionCommand);
   Environment.Exit(await rootCommand.InvokeAsync(args));
   ```

   먼저 `provision-connection.`이라는 명령을 정의합니다. 이 명령은 이전에 정의한 `ConnectionService.ProvisionConnection` 메서드를 호출합니다. 마지막으로 명령줄 프로세서에 명령을 등록하고 명령줄에 전달된 애플리케이션 모니터링 인수를 시작합니다.

1. 변경 내용을 저장합니다.

애플리케이션을 테스트합니다:

1. 터미널을 엽니다.
1. 작업 디렉터리를 프로젝트 폴더로 변경합니다.
1. `dotnet build`를 실행하여 프로젝트를 빌드합니다.
1. `dotnet run -- provision-connection`를 실행하여 앱을 실행합니다.
1. 연결 및 스키마가 생성될 때까지 몇 분 정도 기다립니다.

[ 다음 연습으로 계속...](./3-exercise-import-external-content.md)