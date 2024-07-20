---
lab:
  title: 연습 2 - 외부 콘텐츠 가져오기
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 연습 - 외부 콘텐츠 가져오기

이 연습에서는 코드로 사용자 지정 Microsoft Graph 커넥터를 확장하여 로컬 Markdown 파일을 Microsoft 365로 가져옵니다.

## 시작하기 전에

이 연습을 완료하는 데 약 **15분**이 소요됩니다.

## 작업 1 - 외부 콘텐츠 다운로드

이 연습을 수행하려면 [GitHub](https://pnp.github.io/download-partial/?url=https://github.com/pnp/graph-connectors-samples/tree/main/samples/dotnet-csharp-graphdocs/content)에서 이 연습에 사용된 샘플 콘텐츠 파일을 복사하여 프로젝트의 **콘텐츠**라는 폴더에 저장합니다.

:::image type="content" source="../media/6-content-files.png" alt-text="이 연습에 사용된 콘텐츠 파일을 보여 주는 코드 편집기의 스크린샷":::

코드가 제대로 작동하려면 **콘텐츠** 폴더와 해당 내용을 빌드 출력 폴더에 복사해야 합니다.

코드 편집기에서:

1. **.csproj** 파일을 열고, `</Project>` 태그 앞에 다음 코드를 추가합니다

   ```xml
   <ItemGroup>
     <ContentFiles Include="content\**"    CopyToOutputDirectory="PreserveNewest" />
   </ItemGroup>
   
   <Target Name="CopyContentFolder" AfterTargets="Build">
     <Copy SourceFiles="@(ContentFiles)" DestinationFiles="@   (ContentFiles->'$(OutputPath)\content\%(RecursiveDir)%(Filename)%   (Extension)')" />
   </Target>
   ```

1. 변경 내용을 저장합니다.

## 작업 2 - Markdown 및 YAML을 구문 분석하는 라이브러리 추가

빌드하는 Microsoft Graph 커넥터는 로컬 Markdown 파일을 Microsoft 365로 가져옵니다. 이러한 각 파일에는 YAML 형식의 메타데이터가 있는 헤더(frontmatter라고도 함)가 포함됩니다. 또한 각 파일의 내용은 Markdown으로 작성됩니다. 메타데이터를 추출하고 본문을 HTML로 변환하려면 사용자 지정 라이브러리를 사용합니다.

1. 터미널을 열고 작업 디렉터리를 프로젝트로 변경합니다.
1. Markdown 처리 라이브러리를 추가하려면 다음 명령을 실행합니다: `dotnet add package Markdig`.
1. YAML 처리 라이브러리를 추가하려면 다음 명령을 실행합니다: `dotnet add package YamlDotNet`.

## 작업 3 - 가져온 파일을 나타내는 클래스 정의

가져온 Markdown 파일 및 해당 콘텐츠 작업을 간소화하기 위해 필요한 속성을 사용하여 클래스를 정의해 보겠습니다.

코드 편집기에서:

1. **ContentService.cs**라는 새 파일을 만듭니다.
1. 다음 코드를 추가합니다.

   ```csharp
   using YamlDotNet.Serialization;
   
   public interface IMarkdown
   {
     string? Markdown { get; set; }
   }
   
   class DocsArticle : IMarkdown
   {
     [YamlMember(Alias = "title")]
     public string? Title { get; set; }
     [YamlMember(Alias = "description")]
     public string? Description { get; set; }
     public string? Markdown { get; set; }
     public string? Content { get; set; }
     public string? RelativePath { get; set; }
   }
   ```

   `IMarkdown` 인터페이스는 로컬 Markdown 파일의 내용을 나타냅니다. 파일 콘텐츠 역직렬화를 지원하려면 별도로 정의해야 합니다. `DocsArticle` 클래스는 구문 분석된 YAML 속성 및 HTML 콘텐츠가 포함된 최종 문서를 나타냅니다. `YamlMember` 속성은 각 문서 헤더의 메타데이터에 속성을 매핑합니다.

1. 변경 내용을 저장합니다.

## 작업 4 - 클래스 정의 `ContentService`

다음으로, 로컬 markdown 파일을 로드하고, 외부 항목으로 변환하고, Microsoft 365로 로드하는 코드가 포함된 클래스를 빌드합니다.

코드 편집기에서:

1. **ContentService.cs** 파일을 편집하고 있는지 확인합니다.
1. 파일 맨 위에 다음 using 문을 추가합니다.

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. 그런 다음 파일의 끝에 다음 코드를 추가합니다.

   ```csharp
   static class ContentService
   {
     static IEnumerable<DocsArticle> Extract()
     {}
   
     static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
     {}
   
     static async Task Load(IEnumerable<ExternalItem> items)
     {}
   
     public static async Task LoadContent()
     {
       var content = Extract();
       var transformed = Transform(content);
       await Load(transformed);
     }
   }
   ```

   `ContentService` 클래스는 콘텐츠 처리 프로세스를 나타내는 세 가지 메서드를 정의합니다:

   1. `Extract` - 로컬 Markdown 파일을 로드하고 `DocsArticle` 클래스의 인스턴스로 구문 분석하여 쉽게 처리할 수 있도록 합니다.
   1. `Transform` - `DocsArticle` 개체를 Microsoft Graph .NET SDK의 일부이며 Microsoft 365에 로드할 외부 항목을 나타내는 `ExternalItems` 클래스의 인스턴스로 변환합니다.
   1. `Load` - Microsoft Graph API를 사용하여 외부 항목을 Microsoft 365로 로드합니다.

   이러한 메서드는 `LoadContent` 메서드에서 이 특정 순서로 호출됩니다.

1. 변경 내용을 저장합니다.

## 작업 5 - Markdown 처리 구성

먼저 로컬 Markdown 파일에서 콘텐츠를 추출해 보겠습니다.

먼저 `Markdig` 및 `YamlDotNet` 라이브러리를 쉽게 사용할 수 있도록 도우미 메서드를 추가합니다.

코드 편집기에서:

1. **MarkdownExtensions.cs**라는 새 파일을 만듭니다.
1.  파일에서 다음 코드를 추가합니다.

   ```csharp
   // from: https://khalidabuhakmeh.com/parse-markdown-front-matter-with-csharp
   using Markdig;
   using Markdig.Extensions.Yaml;
   using Markdig.Syntax;
   using YamlDotNet.Serialization;
   
   public static class MarkdownExtensions
   {
     private static readonly IDeserializer YamlDeserializer =
         new DeserializerBuilder()
         .IgnoreUnmatchedProperties()
         .Build();
         
     private static readonly MarkdownPipeline Pipeline
         = new MarkdownPipelineBuilder()
         .UseYamlFrontMatter()
         .Build();
   }
   ```

   `YamlDeserializer` 속성은 추출하는 각 Markdown 파일에 있는 YAML 블록에 대한 새 역직렬 변환기를 정의합니다. 파일이 역직렬화되는 클래스의 일부가 아닌 모든 속성을 무시하도록 역직렬 변환기를 구성합니다.

   `Pipeline` 속성은 Markdown 파서의 처리 파이프라인을 정의합니다. YAML 헤더를 구문 분석하도록 구성합니다. 이 구성이 없으면 헤더의 정보가 삭제됩니다.

1. 다음 코드를 사용하여 `MarkdownExtensions` 클래스를 확장합니다.

   ```csharp
   public static T GetContents<T>(this string markdown) where T :    IMarkdown, new()
   {
     var document = Markdown.Parse(markdown, Pipeline);
     var block = document
         .Descendants<YamlFrontMatterBlock>()
         .FirstOrDefault();
   
     if (block == null)
       return new T { Markdown = markdown };
   
     var yaml =
         block
         // this is not a mistake
         // we have to call .Lines 2x
         .Lines // StringLineGroup[]
         .Lines // StringLine[]
         .OrderByDescending(x => x.Line)
         .Select(x => $"{x}\n")
         .ToList()
         .Select(x => x.Replace("---", string.Empty))
         .Where(x => !string.IsNullOrWhiteSpace(x))
         .Aggregate((s, agg) => agg + s);
   
     var t = YamlDeserializer.Deserialize<T>(yaml);
     t.Markdown = markdown.Substring(block.Span.End + 1);
     return t;
   }
   ```

   `GetContents` 메서드는 헤더에 YAML 메타데이터가 있는 Markdown 문자열을 `IMarkdown` 인터페이스를 구현하는 지정된 유형으로 변환합니다. Markdown 문자열에서 YAML 헤더를 추출하고 지정된 형식으로 역직렬화합니다. 그런 다음 글의 본문을 추출하고 추가 처리를 위해 `Markdown` 속성으로 설정합니다.

1. 변경 내용을 저장합니다.

## 작업 6 - Markdown 및 YAML 콘텐츠 추출

도우미 메서드가 준비되면 추출 메서드를 구현하여 로컬 Markdown 파일을 로드하고 해당 파일에서 정보를 추출합니다.

코드 편집기에서:

1. **ContentService.cs** 파일을 엽니다.
1. 파일 맨 위에 다음 using 문을 추가합니다.

   ```csharp
   using Markdig;
   ```

1. 다음으로, `ContentService` 클래스에 다음 코드를 추가하여 `Extract` 메서드를 구현합니다.

   ```csharp
   static IEnumerable<DocsArticle> Extract()
   {
     var docs = new List<DocsArticle>();
   
     var contentFolder = "content";
     var contentFolderPath = Path.Combine(Directory.GetCurrentDirectory(),    contentFolder);
     var files = Directory.GetFiles(contentFolder, "*.md", SearchOption.   AllDirectories);
   
     foreach (var file in files)
     {
       try
       {
         var contents = File.ReadAllText(file);
         var doc = contents.GetContents<DocsArticle>();
         doc.Content = Markdown.ToHtml(doc.Markdown ?? "");
         doc.RelativePath = Path.GetRelativePath(contentFolderPath, file);
         docs.Add(doc);
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   
     return docs;
   }
   ```

   이 메서드는 **콘텐츠** 폴더에서 Markdown 파일을 로드하는 것으로 시작합니다. 각 파일에 대해 해당 내용을 문자열로 로드합니다. `MarkdownExtensions` 클래스의 앞부분에서 정의된 `GetContents` 확장 메서드를 사용하여 메타데이터 및 콘텐츠가 별도의 속성에 저장된 개체로 문자열을 변환합니다. 다음으로 Markdown 문자열을 HTML로 변환합니다. 마지막으로 파일의 상대 경로를 저장하고 추가 처리를 위해 컬렉션에 개체를 추가합니다.

1. 변경 내용을 저장합니다.

## 작업 7 - 콘텐츠를 외부 항목으로 변환

외부 콘텐츠를 읽은 후 다음 단계는 외부 항목으로 변환하여 Microsoft 365로 로드하는 것입니다.

먼저 상대 파일 경로를 기반으로 각 외부 항목의 고유 ID를 생성하는 도우미 메서드를 추가합니다.

코드 편집기에서:

1. **ContentService.cs** 파일을 편집하고 있는지 확인합니다.
1. `ContentService` 클래스에서는 다음 메서드를 추가합니다.

   ```csharp
   static string GetDocId(string relativePath)
   {
     var id = relativePath.Replace(Path.DirectorySeparatorChar.ToString(),    "__").Replace(".md", "");
     return id;
   }
   ```

   `GetDocId` 메서드는 상대 파일 경로를 사용하고 모든 디렉터리 구분 기호를 이중 밑줄로 바꿉니다. 이는 외부 항목 ID에서 경로 구분 기호 문자를 사용할 수 없기 때문에 필요합니다.

1. 변경 내용을 저장합니다.

이제 로컬 Markdown 파일을 나타내는 개체를 Microsoft Graph의 외부 항목으로 변환하는 `Transform` 메서드를 구현합니다.

코드 편집기에서:

1. **ContentService.cs** 파일에 있는지 확인합니다.
1. 다음 코드를 사용하여 `Transform` 메서드를 구현합니다.

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var docId = GetDocId(a.RelativePath ?? '');
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

   먼저 기본 URL을 정의합니다. 이 URL을 사용하여 각 항목에 대한 전체 URL을 작성하므로 사용자에게 항목이 표시되면 원래 항목으로 이동할 수 있습니다. 다음으로 각 항목을 `DocsArticle`에서 `ExternalItem`로 변환합니다. 먼저 상대 파일 경로에 따라 각 항목의 고유 ID를 가져옵니다. 그런 다음 `ExternalItem`의 새 인스턴스를 만들고 해당 속성을 `DocsArticle`의 정보로 채웁니다. 그런 다음 항목의 콘텐츠를 로컬 파일에서 추출된 HTML 콘텐츠로 설정하고 항목 콘텐츠 형식을 HTML로 설정합니다. 마지막으로 조직의 모든 사용자가 볼 수 있도록 항목의 사용 권한을 구성합니다.

1. 변경 내용을 저장합니다.

## 작업 8 - Microsoft 365에 외부 항목 로드

콘텐츠를 처리하는 마지막 단계는 변환된 외부 항목을 Microsoft 365로 로드하는 것입니다.

코드 편집기에서:

1. **ContentService.cs** 파일을 편집하고 있는지 확인합니다.
1. `ContentService` 클래스에 다음 코드를 추가하여 `Load` 메서드를 구현합니다.

   ```csharp
   static async Task Load(IEnumerable<ExternalItem> items)
   {
     foreach (var item in items)
     {
       Console.Write(string.Format("Loading item {0}...", item.Id));
   
       try
       {
         await GraphService.Client.External
           .Connections[Uri.EscapeDataString(ConnectionConfiguration.   ExternalConnection.Id!)]
           .Items[item.Id]
           .PutAsync(item);
   
         Console.WriteLine("DONE");
       }
       catch (Exception ex)
       {
         Console.WriteLine("ERROR");
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

   각 외부 항목에 대해 Microsoft Graph .NET SDK를 사용하여 Microsoft Graph API를 호출하고 항목을 업로드합니다. 요청에서 이전에 만든 외부 연결의 ID, 업로드할 항목의 ID 및 전체 항목의 콘텐츠를 지정합니다.

1. 변경 내용을 저장합니다.

## 작업 9 - 콘텐츠 로드 명령 추가

코드를 테스트하려면 콘텐츠 로딩 논리를 호출하는 명령을 사용하여 콘솔 애플리케이션을 확장해야 합니다.

코드 편집기에서:

1. **Program.cs** 파일을 엽니다.
1. 다음 코드를 사용하여 콘텐츠를 로드하는 새 명령을 추가합니다.

    ```csharp
    var loadContentCommand = new Command("load-content", "Loads content   into the external connection");
    loadContentCommand.SetHandler(ContentService.LoadContent);
    ```

1. 다음 코드를 사용하여 새로 정의된 명령을 루트 명령에 등록하여 호출할 수 있도록 합니다.

     ```csharp
     rootCommand.AddCommand(loadContentCommand);
     ```

1. 변경 내용을 저장합니다.

## 작업 10 - 코드 테스트

마지막으로 Microsoft Graph 커넥터가 외부 콘텐츠를 올바르게 가져오는지 테스트합니다.

1. 터미널을 엽니다.
1. 작업 디렉터리를 프로젝트로 변경합니다.
1. `dotnet build` 명령을 사용하여 프로젝트를 빌드하고 실행합니다.
1. `dotnet run -- load-content` 명령을 실행하여 콘텐츠 로드를 시작합니다.
1. 명령이 완료되고 콘텐츠를 로드할 때까지 기다립니다.

[ 다음 연습으로 계속...](./4-exercise-ensure-secure-access.md)