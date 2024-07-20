---
lab:
  title: 연습 3 - 액세스 제어 목록을 사용하여 보안 액세스 보장
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 연습 3 - 액세스 제어 목록을 사용하여 보안 액세스 보장

이 연습에서는 선택한 항목에서 ACL을 구성하여 로컬 Markdown 파일을 가져오는 코드를 업데이트합니다.

## 시작하기 전에

이 연습을 완료하는 데 약 **15분**이 소요됩니다.

## 작업 1 - 조직의 모든 사용자가 사용할 수 있는 콘텐츠 가져오기

이전 연습에서 외부 콘텐츠를 가져오기 위한 코드를 구현할 때 조직의 모든 사용자가 사용할 수 있도록 구성했습니다. 사용한 코드는 다음과 같습니다.

```csharp
static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
{
  var baseUrl = new Uri("https://learn.microsoft.com/graph/");

  return content.Select(a =>
  {
    var docId = GetDocId(a.RelativePath ?? "");

    return new ExternalItem
    {
      Id = docId,
      Properties = new()
      {
        AdditionalData = new Dictionary<string, object> {
            { "title", a.Title ?? "" },
            { "description", a.Description ?? "" },
            { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).ToString() }
        }
      },
      Content = new()
      {
        Value = a.Content ?? "",
        Type = ExternalItemContentType.Html
      },
      Acl = new()
      {
          new()
          {
            Type = AclType.Everyone,
            Value = "everyone",
            AccessType = AccessType.Grant
          }
      }
    };
  });
}
```

모든 사용자에게 액세스 권한을 부여하도록 ACL을 구성했습니다. 가져오고 있는 선택한 Markdown 페이지에 맞게 조정해 보겠습니다.

## 작업 2 - 일부 사용자에게 제공되는 콘텐츠 가져오기

먼저 가져오려는 페이지 중 하나를 특정 사용자만 액세스할 수 있도록 구성합니다.

웹 브라우저에서:

1. [https://portal.azure.com](https://portal.azure.com)에서 Azure Portal로 이동하고 회사 또는 학교 계정으로 로그인합니다.
1. 사이드바에서 **Microsoft Entra ID**를 선택합니다.
1. 탐색 창에서 **사용자**를 선택합니다.
1. 사용자 목록에서 이름을 선택하여 사용자 중 하나를 엽니다.
1. **개체 ID** 속성 값을 복사합니다.

   :::image type="content" source="../media/8-user.png" alt-text="사용자 프로필이 열려 있는 Azure Portal의 스크린샷":::

이 값을 사용하여 특정 Markdown 페이지에 대한 새 ACL을 정의합니다.

코드 편집기에서:

1. **ContentService.cs** 파일을 열고 `Transform` 메서드를 찾습니다.
1. `Select` 대리자 내에서 가져온 모든 항목에 적용되는 기본 ACL을 정의합니다.

   ```csharp
   var acl = new Acl
   {
     Type = AclType.Everyone,
     Value = "everyone",
     AccessType = AccessType.Grant
   };
   ```

1. 다음으로, markdown 파일의 기본 ACL을 `use-the-api.md`로 끝나는 이름으로 재정의합니다.

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   ```

1. 마지막으로, 정의된 ACL을 사용하도록 외부 항목을 반환하는 코드를 업데이트합니다.

   ```csharp
   return new ExternalItem
   {
     Id = docId,
     Properties = new()
     {
       AdditionalData = new Dictionary<string, object> {
         { "title", a.Title ?? "" },
         { "description", a.Description ?? "" },
         { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
       }
     },
     Content = new()
     {
       Value = a.Content ?? "",
       Type = ExternalItemContentType.Html
     },
     Acl = new()
     {
       acl
     }
   };
   ```

1. 업데이트된 `Transform` 메서드는 다음과 같습니다.

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
             { "title", a.Title ?? "" },
             { "description", a.Description ?? "" },
             { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
           acl
         }
       };
     });
   }
   ```

1. 변경 내용을 저장합니다.

## 작업 3 - 선택 그룹에서 사용할 수 있는 콘텐츠 가져오기

이제 선택한 사용자 그룹에서만 다른 페이지에 액세스할 수 있도록 코드를 확장해 보겠습니다.

웹 브라우저에서:

1. [https://portal.azure.com](https://portal.azure.com)에서 Azure Portal로 이동하고 회사 또는 학교 계정으로 로그인합니다.
1. 사이드바에서 **Microsoft Entra ID**를 선택합니다.
1. 탐색 창에서 **그룹**을 선택합니다.
1. 그룹 목록에서 이름을 선택하여 그룹 중 하나를 엽니다.
1. **개체 ID** 속성 값을 복사합니다.

:::image type="content" source="../media/8-group.png" alt-text="그룹 페이지가 열려 있는 Azure Portal의 스크린샷":::

이 값을 사용하여 특정 Markdown 페이지에 대한 새 ACL을 정의합니다.

코드 편집기에서:

1. **ContentService.cs** 파일을 열고 `Transform` 메서드를 찾습니다.
1. 이름이 `traverse-the-graph.md`로 끝나는 markdown 파일의 ACL을 정의하는 추가 조건으로 이전에 정의된 `if` 절을 확장합니다.

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
   {
     acl = new()
     {
       Type = AclType.Group,
       // Sales and marketing
       Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
       AccessType = AccessType.Grant
     };
   }
   ```

1. 업데이트된 `Transform` 메서드는 다음과 같습니다.

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
       else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
       {
         acl = new()
         {
           Type = AclType.Group,
           // Sales and marketing
           Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
               { "title", a.Title ?? "" },
               { "description", a.Description ?? "" },
               { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md",    "")).ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
             acl
         }
       };
     });
   }
   ```

1. 변경 내용을 저장합니다.

## 작업 4 - 새 ACL 적용

마지막 단계는 새로 구성된 ACL을 적용하는 것입니다.

1. 터미널을 열고 작업 디렉터리를 프로젝트로 변경합니다.
1. `dotnet build` 명령을 사용하여 프로젝트를 빌드하고 실행합니다.
1. `dotnet run -- load-content` 명령을 실행하여 콘텐츠 로드를 시작합니다.

[이 연습의 다음 단계를 계속 진행합니다...](./5-exercise-enable-inline-results.md)