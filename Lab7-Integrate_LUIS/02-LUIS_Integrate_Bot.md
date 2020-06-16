# 랩 7: 봇 대화 상자에 LUIS 통합

## 소개

이제 봇이 사용자의 입력을 받고 그에 따라 응답할 수 있습니다. 하지만 봇의 커뮤니케이션 기술이 취약하여 오타가 있거나 단어에 다른 표현을 사용할 경우 봇이 이해할 수 없습니다. 이로 인해 사용자가 답답하게 느낄 수 있습니다. [랩 6](../Lab6-Implement_LUIS/02-Implement_LUIS.md)에서 빌드한 LUIS 모델을 통해 자연어를 이해할 수 있게 함으로써 봇의 대화 능력을 크게 높일 수 있습니다.

LUIS를 사용하도록 봇을 업데이트해야 합니다.  "Startup.cs"와 "PictureBot.cs"를 수정하여 이 작업을 수행할 수 있습니다.

> 사전 요구 사항: 이 랩은 [랩 3](../Lab3-Basic_Filter_Bot/02-Basic_Filter_Bot.md)를 기반으로 합니다. 이 랩에서 다루는 로깅을 구현할 수 있도록 해당 랩을 완료하는 것이 좋습니다. 완료하지 않은 경우 필요에 따라 모든 연습을 주의 깊게 읽고 일부 코드를 살펴보거나 자체 응용 프로그램에서 사용하는 것으로 충분할 수 있습니다.

> 참고: Finished 폴더의 코드를 사용하려면 앱 특정 정보를 고유한 앱 아이디 및 엔드포인트로 바꿔야 합니다.

## 랩 7.1: 자연어 이해 추가

### Startup.cs에 LUIS 추가

1. 아직 열리지 않은 경우 Visual Studio에서 **PictureBot** 솔루션을 엽니다.

> **참고** 랩 1에서 시작하지 않은 경우 **{GitHubPath}/Lab7-Integrate_LUIS/code/Starter/PictureBot/PictureBot.sln** 솔루션으로 시작할 수도 있습니다.
> 모든 앱 설정 값을 교체해야 합니다.

1. **Startup.cs**를 열고 `ConfigureServices` 메서드를 찾습니다. 상태 접근자를 만들고 등록한 후 LUIS에 대한 추가적 서비스를 추가하여 여기에서 LUIS를 추가합니다.

다음 코드 아래에

```csharp
services.AddSingleton((Func<IServiceProvider, PictureBotAccessors>)(sp =>
{
    .
    .
    .
    return accessors;
});
```

다음을 추가합니다.

```csharp
// LUIS Recognizer를 만들고 등록합니다.
services.AddSingleton(sp =>
{
    var luisAppId = Configuration.GetSection("luisAppId")?.Value;
    var luisAppKey = Configuration.GetSection("luisAppKey")?.Value;
    var luisEndPoint = Configuration.GetSection("luisEndPoint")?.Value;

    // LUIS 정보를 얻습니다.
    var luisApp = new LuisApplication(luisAppId, luisAppKey, luisEndPoint);

    // LUIS 옵션을 지정합니다. 이는 봇에 따라 다를 수 있습니다.
    var luisPredictionOptions = new LuisPredictionOptions
    {
        IncludeAllIntents = true,
    };

    // Recognizer를 만듭니다.
    var recognizer = new LuisRecognizer(luisApp, luisPredictionOptions, true, null);
    return recognizer;
});
```

1. 다음 속성을 포함하도록 **appsettings.json**을 수정하고 LUIS 인스턴스 값으로 채워야 합니다.

```json
"luisAppId": "",
"luisAppKey": "",
"luisEndPoint": ""
```

> **참고** .NET SDK의 Luis 엔드포인트 URL은 **https://{region}.api.cognitive.microsoft.com** 같은 URL이어야 하며, 그 뒤에 API 또는 버전이 없어야 합니다.

## 랩 7.2: PictureBot의 MainDialog에 LUIS 추가

1. **PictureBot.cs**를 엽니다. 먼저, `PictureBotAccessors`에 대해 수행한 것과 마찬가지로 LUIS Recognizer를 초기화해야 합니다. 주석 줄 `// LUIS Recognizer를 초기화합니다.` 아래에 다음을 추가합니다.

```csharp
private LuisRecognizer _recognizer { get; } = null;
```

1. **PictureBot** 생성자로 이동합니다.

```csharp
public PictureBot(PictureBotAccessors accessors, ILoggerFactory loggerFactory /*, LuisRecognizer recognizer*/)
```

이전 랩에서 이것을 주석 처리했습니다. 아직 주석 처리하지 않았다면 지금 주석 처리해야 합니다. 지금까지는 LUIS를 호출하지 않았기 때문에 LUIS Recognizer가 PictureBot에 대한 입력이 될 필요가 없었기 때문입니다. 이제 Recognizer를 사용하고 있습니다.

1. 입력 요구 사항(매개 변수 `LuisRecognizer recognizer`)의 주석을 해제하고 `// LUIS Recognizer 인스턴스를 추가합니다` 아래에 다음 줄을 추가합니다.

```csharp
_recognizer = recognizer ?? throw new ArgumentNullException(nameof(recognizer));
```

다시 말하지만, 이것은 `_accessors`의 인스턴스를 초기화한 방법과 매우 유사합니다.

`MainDialog` 업데이트와 관련하여 초기 `GreetingAsync` 단계에 아무것도 추가할 필요가 없습니다. 사용자 입력에 관계없이 대화가 시작될 때 사용자에게 인사하려고 하기 때문입니다.

1. `MainMenuAsync`에서는 먼저 Regex를 시도하려고 하므로 대부분 그대로 둡니다. 그러나 Regex로 의도를 찾지 못하면 `기본` 작업을 변경해야 합니다. 즉, LUIS를 호출해야 합니다.

`MainMenuAsync` 스위치 블록 내에서 다음 코드를

```csharp
default:
    {
        await MainResponses.ReplyWithConfused(stepContext.Context);
        return await stepContext.EndDialogAsync();
    }
```

다음 코드로 바꿉니다.

```csharp
default:
{
    // LUIS Recognizer를 호출합니다.
    var result = await _recognizer.RecognizeAsync(stepContext.Context, cancellationToken);
    // 결과에서 최상위 의도를 얻습니다.
    var topIntent = result?.GetTopScoringIntent();
    // 의도에 따라 위 Regex와 마찬가지로 대화를 전환합니다.
    switch ((topIntent != null) ? topIntent.Value.intent : null)
    {
        case null:
            // 결과가 없는 경우 앱 논리를 추가합니다.
            await MainResponses.ReplyWithConfused(stepContext.Context);
            break;
        case "None":
            await MainResponses.ReplyWithConfused(stepContext.Context);
            // 각 문마다 테스트 목적으로 LuisScore를 추가하여 LUIS가 호출되었는지 여부를 알 수 있습니다.
            await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
            break;
        case "Greeting":
            await MainResponses.ReplyWithGreeting(stepContext.Context);
            await MainResponses.ReplyWithHelp(stepContext.Context);
            await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
            break;
        case "OrderPic":
            await MainResponses.ReplyWithOrderConfirmation(stepContext.Context);
            await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
            break;
        case "SharePic":
            await MainResponses.ReplyWithShareConfirmation(stepContext.Context);
            await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
            break;
        case "SearchPic":
            await MainResponses.ReplyWithSearchConfirmation(stepContext.Context);
            await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
            break;
        default:
            await MainResponses.ReplyWithConfused(stepContext.Context);
            break;
    }
    return await stepContext.EndDialogAsync();
}
```

새 코드 추가의 내용을 간단히 살펴보겠습니다. 먼저, 이해하지 못한다고 대답하는 대신 LUIS를 호출합니다. 즉, LUIS Recognizer를 사용하여 LUIS를 호출하고 최상위 의도를 변수에 저장합니다. 그런 다음 `스위치`를 사용하여 선택된 의도에 따라 다른 방식으로 응답합니다. 이는 Regex를 사용했을 때와 거의 동일합니다.

> **참고** LUIS에서 의도의 이름을 [랩 6](../Lab6-Implement_LUIS/02-Implement_LUIS.md)의 해당 코드에서 지정한 것과 다르게 지정한 경우 그에 따라 `case` 문을 수정해야 합니다.

또 다른 주목할 점은 LUIS를 호출한 모든 응답 이후에 LUIS 의도 값과 점수를 추가한다는 것입니다. 그 이유는 단지 Regex가 아닌 LUIS가 호출되는 경우를 보여주기 위한 것입니다. 최종 산물에서는 이러한 응답을 제거하지만 봇을 테스트할 때 좋은 지표입니다.

## 랩 7.3: 자연어 구문 테스트

1. **F5** 키를 눌러 앱을 실행합니다.

1. Bot Emulator로 전환합니다. 봇에게 다양한 방식으로 사진 검색을 요청해 보십시오. "물 사진을 보내 주세요" 또는 "개 사진을 보여 주세요"라고 말하면 어떻게 됩니까? 다른 방식으로 사진을 요청하고, 공유하고, 정렬해 보십시오.

여분의 시간이 있는 경우 LUIS가 예상대로 선택하지 않는 항목이 있는지 확인하십시오. 이제 luis.ai에서 [끝점 발화를 검토하고](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/label-suggested-utterances) 모델을 다시 학습/다시 게시하는 것이 좋습니다.

> **참고**: 끝점 발화를 검토하는 것은 매우 유용할 수 있습니다.  LUIS는 표시할 발화에 대해 스마트한 결정을 내립니다.  루프의 사람을 통해 수동으로 레이블이 지정되어 최대한 개선 가능한 발화를 선택합니다.  예를 들어 주어진 발화가 47% 신뢰 수준으로 Intent1에 매핑되고 48% 신뢰 수준으로 Intent2에 매핑될 것으로 LUIS 모델이 예측한 경우, 후자가 사람에게 표시하여 수동으로 매핑할 강력한 후보가 됩니다. 이 모델에서는 두 의도 간 차이가 거의 없기 때문입니다.

## 다음 단계

LUIS 구현을 사용자 지정하는 데 문제가 있는 경우 [여기](https://docs.microsoft.com/ko-kr/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&tabs=cs)에서 봇에 LUIS를 추가하는 것에 대한 설명서 지침을 검토하십시오.

> 문제가 발생할 경우 [code/Finished](./code/Finished)에서 이 지점까지 발생하는 랩 문제에 대한 해결 방법을 찾을 수 있습니다. `appsettings.json` 파일에 Azure Bot Service의 키를 삽입해야 합니다. 이 코드는 실행할 해결 방법이 아닌 참조로 사용하는 것이 좋지만, 실행하려는 경우 필요한 키를 추가해야 합니다(이 섹션에서는 필요 없음).

## **추가 크레딧**

이전 보충 LUIS model-with-search [교육](https://github.com/Azure/LearnAI-Bootcamp/tree/master/lab01.5-luis)을 기반으로 Azure Cognitive Search를 포함하여 LUIS 봇을 통합하려면 다음 교육을 완료하십시오. [Azure Cognitive Search](https://github.com/Azure/LearnAI-Bootcamp/tree/master/lab02.1-azure_search) 및 [Azure Cognitive Search 봇](https://github.com/Azure/LearnAI-Bootcamp/blob/master/lab02.2-building_bots/2_Azure_Search.md).

## 다음 단계

- [랩 08-01: 언어 감지](../Lab8-Detect_Language/01-Introduction.md)
