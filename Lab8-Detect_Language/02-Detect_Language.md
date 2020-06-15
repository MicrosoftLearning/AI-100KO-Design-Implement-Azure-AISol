# 랩 8 - 언어 감지

이 실습에서는 Cognitive Services의 언어 감지 기능을 봇에 통합할 것입니다.

## 랩 8.1: Cognitive Services URL 및 키 검색

1. [Azure Portal](https://portal.azure.com)을 엽니다.

1. 리소스 그룹으로 이동하여 일반적인(즉, 모든 엔드포인트가 포함되어 있음) Cognitive Services 리소스를 선택합니다.

1. **리소스 관리**에서 **빠른 시작** 탭을 선택하고 Cognitive Services 리소스의 URL과 키를 기록합니다.

## 랩 8.2: 봇에 언어 지원 추가

1. 아직 열리지 않은 경우 **PictureBot** 솔루션을 엽니다.

1. 프로젝트를 마우스 오른쪽 단추로 클릭하고 **Nuget 패키지 관리**를 선택합니다.

1. **찾아보기**를 선택합니다.

1. **Microsoft.Azure.CognitiveServices.Language.TextAnalytics**를 검색하고 선택한 다음 **설치**와 **동의함**을 차례로 클릭합니다.

1. **Startup.cs** 파일을 열고 다음 using 문을 추가합니다.

```csharp
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
```

1. **ConfigureServices**메서드에 다음 코드를 추가합니다.

```csharp
services.AddSingleton(sp =>
{
    string cogsBaseUrl = Configuration.GetSection("cogsBaseUrl")?.Value;
    string cogsKey = Configuration.GetSection("cogsKey")?.Value;

    var credentials = new ApiKeyServiceClientCredentials(cogsKey);
    TextAnalyticsClient client = new TextAnalyticsClient(credentials)
    {
        Endpoint = cogsBaseUrl
    };

    return client;
});
```

1. **PictureBot.cs** 파일을 열고 다음 using 문을 추가합니다.

```csharp
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
```

1. 다음 클래스 변수를 추가합니다.

```csharp
private TextAnalyticsClient _textAnalyticsClient;
```

1. 새 TextAnalyticsClient를 포함하도록 생성자를 수정합니다.

```csharp
public PictureBot(PictureBotAccessors accessors, ILoggerFactory loggerFactory,LuisRecognizer recognizer, TextAnalyticsClient analyticsClient)
```

1. 생성자 내에서 클래스 변수를 초기화합니다.

```csharp
_textAnalyticsClient = analyticsClient;
```

1. **OnTurnAsync** 메서드로 이동하여 다음 코드 줄을 찾습니다.

```csharp
var utterance = turnContext.Activity.Text;
var state = await _accessors.PictureState.GetAsync(turnContext, () => new PictureState());
state.UtteranceList.Add(utterance);
await _accessors.ConversationState.SaveChangesAsync(turnContext);
```

1. 다음 코드 줄을 뒤에 추가합니다.

```csharp
//언어를 확인합니다.
var result = _textAnalyticsClient.DetectLanguage(turnContext.Activity.Text, "us");

switch (result.DetectedLanguages[0].Name)
{
    case "English":
        break;
    default:
        //오류를 throw합니다.
        await turnContext.SendActivityAsync($"I'm sorry, I can only understand English. [{result.DetectedLanguages[0].Name}]");
        return;
        break;
}
```

1. **appsettings.json** 파일을 열고 Cognitive Services 설정이 입력되었는지 확인합니다.

```csharp
"cogsBaseUrl": "",
"cogsKey" :  ""
```

1. **F5** 키를 눌러 봇을 시작합니다.

1. Bot Emulator를 사용하여 몇 가지 문구를 보내고 무슨 일이 일어나는지 확인합니다.

- Como Estes?
- Bon Jour!
- Привет
- Hello

## 다음 단계

이전 랩에서 LUIS를 이미 소개했기 때문에 LUIS를 사용하여 여러 언어를 지원하기 위해 어떤 것을 변경해야 하는지 생각해 보십시오.  몇 가지 유용한 문서:

- [LUIS의 언어 및 지역 지원](https://docs.microsoft.com/ko-kr/azure/cognitive-services/luis/luis-language-support)

## 리소스

- [예제: Text Analytics로 언어 감지](https://docs.microsoft.com/ko-kr/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-language-detection)
- [빠른 시작: .NET용 Text Analytics 클라이언트 라이브러리](https://docs.microsoft.com/ko-kr/azure/cognitive-services/text-analytics/quickstarts/csharp)

## 다음 단계

- [랩 09-01: 봇 DirectLine 테스트](../Lab9-Test_Bots_DirectLine/01-Introduction.md)