---
lab:
    title: '랩 4: 채팅 로깅'
    module: '모듈 2: 봇 만들기'
---

# 랩 4: 채팅 로깅

##  소개

이 워크샵에서는 Microsoft Bot Framework를 사용하여 로깅을 수행하고 채팅 대화의 여러 측면을 저장하는 방법을 보여 줍니다. 이 랩을 완료하면 다음과 같은 역량을 갖추게 됩니다.

- 봇과 사용자 간의 메시지 활동을 가로채고 로깅하는 방법 이해
- 파일 저장소에 발화 로깅

## 사전 요구 사항

이 랩은 [랩 3](../Lab3-Basic_Filter_Bot/02-Basic_Filter_Bot.md)에서 봇을 빌드하고 게시했다는 전제하에 시작됩니다.

이 랩에서 다루는 로깅을 구현할 수 있도록 해당 랩을 완료하는 것이 좋습니다. 완료하지 않은 경우 필요에 따라 모든 연습을 주의 깊게 읽고 일부 코드를 살펴보거나 자체 응용 프로그램에서 사용하는 것으로 충분할 수 있습니다.

## 랩 4.0: 메시지 가로채기 및 분석

이 랩에서는 Bot Framework를 통해 봇이 사용자와 나눈 대화에서 데이터를 가로채고 로깅하는 몇 가지 방법을 살펴보겠습니다. 처음에는 메모리 내 스토리지를 사용할 것이며, 이는 테스트용으로는 좋지만 프로덕션 환경에는 적합하지 않습니다.

다음으로, 대화의 데이터를 Azure의 파일로 작성하는 방법에 대한 매우 간단한 구현을 살펴보겠습니다. 특히 사용자가 봇에 보내는 메시지를 목록에 넣고 몇 가지 다른 항목과 함께 목록을 임시 파일에 저장합니다(필요에 따라 특정 파일 경로로 변경할 수 있음).

#### Bot Framework Emulator 사용

봇에 아무것도 추가하지 않고 테스트 목적으로 수집할 수 있는 정보를 살펴보겠습니다.

1.  Visual Studio에서 **PictureBot.sln** 을 엽니다. 

> **참고** 랩 3을 수행하지 않은 경우 **시작** 솔루션을 사용할 수 있습니다.

1.  **F5** 키를 눌러 봇을 실행합니다.

1.  Bot Framework Emulator에서 봇을 열고 빠른 대화를 갖습니다.

1.  Bot Emulator 디버그 영역을 검토하고 다음 사항을 확인합니다.

- 메시지를 클릭하면 오른쪽에서 "Inspector-JSON" 도구와 연결된 JSON을 볼 수 있습니다. 메시지를 클릭하고 JSON을 검토하여 얻을 수 있는 정보를 확인합니다.

- 오른쪽 하단에 있는 "로그"에는 대화의 전체 로그가 포함되어 있습니다. 조금 더 깊이 살펴보겠습니다.

    - 가장 먼저 볼 수 있는 것은 에뮬레이터가 수신하는 포트입니다.

    - 또한 ngrok가 수신하는 위치도 확인할 수 있으며 "ngrok 트래픽 검사기"링크를 사용하여 ngrok에 대한 트래픽을 검사할 수 있습니다. 그러나 로컬 주소를 선택하면 ngrok을 바이패스할 수 있습니다. **원격 테스트는 이 워크샵에서 다루지 않으므로 ngrok는 여기에 정보 전달용으로 포함되어 있습니다.**

    - 호출에 오류가 있는 경우(POST 200 또는 POST 201 응답 이외의 응답) 오류를 클릭하고 "Inspector-JSON"에서 매우 자세한 로그를 볼 수 있습니다. 어떤 오류인지에 따라, 코드를 거치면서 오류가 발생한 위치를 지적하는 스택 추적을 얻을 수도 있습니다. 이는 봇 프로젝트를 디버깅할 때 매우 유용합니다.

    - 또한 LUIS를 호출할 때는 `Luis Trace` 가 있습니다. `trace` 링크를 클릭하면 LUIS 정보를 볼 수 있습니다. 이 랩에서는 설정되지 않았습니다.

![에뮬레이터](../images/emulator.png)

에뮬레이터를 사용한 테스트, 디버깅 및 로깅에 대한 자세한 내용은 [여기](https://docs.microsoft.com/ko-kr/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0) 에서 확인할 수 있습니다.

## 랩 4.1: Azure Storage에 로깅

기본 봇 스토리지 공급자는 메모리 내 스토리지를 사용하고, 이 스토리지는 봇이 다시 시작될 때 삭제됩니다. 이는 테스트 목적으로만 적합합니다. 데이터를 유지하고 싶지만 봇을 데이터베이스에 연결하지 않으려는 경우, Azure Storage 공급자를 사용하거나 SDK를 사용하여 자체 공급자를 빌드할 수 있습니다. 

1.  **Startup.cs** 파일을 엽니다.  모든 메시지에 이 프로세스를 사용하려고 하므로 Startup 클래스의 `ConfigureServices` 메서드를 사용하여 Azure Blob 파일에 저장 정보를 추가합니다. 현재 다음이 사용되고 있습니다.

```csharp
IStorage dataStore = new MemoryStorage();
```

위에서 볼 수 있듯이 현재의 구현은 메모리 내 스토리지를 사용합니다. 다시 말하지만, 이 메모리 스토리지는 로컬 봇 디버깅에만 권장됩니다. 봇이 다시 시작되면 메모리에 저장된 모든 것이 사라집니다.

1.  현재의 `IStorage` 줄을 다음으로 바꿔 FileStorage 기반 지속성으로 변경합니다.

```csharp
var blobConnectionString = Configuration.GetSection("BlobStorageConnectionString")?.Value;
var blobContainer = Configuration.GetSection("BlobStorageContainer")?.Value;
IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureBlobStorage(blobConnectionString, blobContainer);
```

1.  Azure Portal로 전환하고 Blob Storage 계정으로 이동합니다.

1.  **개요** 탭에서 **컨테이너** 를 클릭합니다.

1.  **chatlog** 컨테이너가 있는지 확인합니다. 그렇지 않을 경우 **+컨테이너** 를 클릭합니다.

    -   이름의 경우 **chatlog** 를 입력하고 **확인** 을 클릭합니다.

1.  아직 그렇게 하지 않은 경우, **액세스 키** 를 클릭하고 연결 문자열을 기록합니다.

1.  **appsettings.json** 을 열고 Blob 연결 문자열 세부 정보를 추가합니다.

```json
"BlobStorageConnectionString": "",
"BlobStorageContainer" :  "chatlog"
```

1.  **F5** 키를 눌러 봇을 실행합니다. 

1.  에뮬레이터에서 봇과의 샘플 대화를 진행합니다.

> **참고** 응답을 받지 못하면 Azure Storage 계정 연결 문자열을 확인합니다.

1.  Azure Portal로 전환하고 Blob Storage 계정으로 이동합니다.

1.  **컨테이너**를 클릭한 다음 **ChatLog** 컨테이너를 엽니다.

1.  채팅 로그 파일을 선택합니다. 이 파일은 **emulator...** 로 시작될 것입니다. 그런 다음 **Blob 편집** 을 선택합니다.  파일에 어떤 내용이 있습니까? 예상한 내용 중 없는 것은 무엇입니까?

다음과 비슷한 내용이 표시될 것입니다.

```json
{"$type":"System.Collections.Concurrent.ConcurrentDictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Collections.Concurrent","DialogState":{"$type":"Microsoft.Bot.Builder.Dialogs.DialogState, Microsoft.Bot.Builder.Dialogs","DialogStack":{"$type":"System.Collections.Generic.List`1[[Microsoft.Bot.Builder.Dialogs.DialogInstance, Microsoft.Bot.Builder.Dialogs]], System.Private.CoreLib","$values":[{"$type":"Microsoft.Bot.Builder.Dialogs.DialogInstance, Microsoft.Bot.Builder.Dialogs","Id":"mainDialog","State":{"$type":"System.Collections.Generic.Dictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Private.CoreLib","options":null,"values":{"$type":"System.Collections.Generic.Dictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Private.CoreLib"},"instanceId":"f80db88d-cdea-4b47-a3f6-a5bfa26ed60b","stepIndex":0}}]}},"PictureBotAccessors.PictureState":{"$type":"Microsoft.PictureBot.PictureState, PictureBot","Greeted":"greeted","Search":"","Searching":"no"}}
```

## 랩 4.2: 파일에 발화 로깅

이 랩의 목적상, 사용자가 봇에게 보내는 실제 발화에 초점을 맞출 것입니다. 이는 사용자가 봇과 수행하려는 대화 및 작업 유형을 결정하는 데 유용할 수 있습니다.

**PictureState.cs** 파일의 `UserData` 개체에 저장된 내용을 업데이트하고 **PictureBot.cs** 의 해당 개체에 정보를 추가하여 이 작업을 수행할 수 있습니다.

1.  **PictureState.cs** 를 엽니다.

1.  아래 코드 **다음에**

```csharp
public class PictureState
{
    public string Greeted { get; set; } = "not greeted";
```

다음을 추가합니다.

```csharp
// 사용자가 봇에게 말한 내용 목록
public List<string> UtteranceList { get; private set; } = new List<string>();
```

위에서는 단지 사용자가 봇에 보내는 메시지 목록을 저장하는 목록을 만들었습니다.

이 예제에서는 상태 관리자를 사용하여 데이터를 읽고 쓰도록 선택했지만 [상태 관리자를 사용하지 않고 저장소에서 직접 읽고 쓸 수도](https://docs.microsoft.com/ko-kr/azure/bot-service/bot-builder-howto-v4-storage?view=azure-bot-service-4.0&tabs=csharpechorproperty%2Ccsetagoverwrite%2Ccsetag) 있습니다.

> 저장소에 직접 쓰기로 선택한 경우 시나리오에 따라 eTag를 설정할 수 있습니다. eTag 속성을 `*` 로 설정하면 봇의 다른 인스턴스가 이전에 작성된 데이터를 덮어쓸 수 있어 마지막 데이터가 기록됩니다. 여기서는 살펴보지 않지만 [동시성 관리에 대한 자세한 내용](https://docs.microsoft.com/ko-kr/azure/bot-service/bot-builder-howto-v4-storage?view=azure-bot-service-4.0&tabs=csharpechorproperty%2Ccsetagoverwrite%2Ccsetag#manage-concurrency-using-etags)을 확인할 수 있습니다.

봇을 실행하기 전에 마지막으로 `OnTurn` 작업을 사용하여 목록에 메시지를 추가해야 합니다. 

1.  **PictureBot.cs** 에서 아래 코드 **다음에**

```csharp
public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
        {
            if (turnContext.Activity.Type is "message")
            {
```

다음을 추가합니다.

```csharp
var utterance = turnContext.Activity.Text;
var state = await _accessors.PictureState.GetAsync(turnContext, () => new PictureState());
state.UtteranceList.Add(utterance);
await _accessors.ConversationState.SaveChangesAsync(turnContext);
```

> **참고** 수정할 경우 상태를 저장해야 합니다.

첫 번째 줄은 사용자로부터 들어오는 메시지를 가져와 `utterance` 라는 변수에 저장합니다. 다음 줄은 PictureState.cs에 만든 기존 목록에 utterance를 추가합니다.

1.  **F5** 키를 눌러 봇을 실행합니다.

1.  봇과 다른 대화를 나누어 봅니다. 봇을 중지하고 최신 Blob 지속형 로그 파일을 확인합니다. 현재 어떤 내용을 볼 수 있습니까?

```json
{"$type":"System.Collections.Concurrent.ConcurrentDictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Collections.Concurrent","DialogState":{"$type":"Microsoft.Bot.Builder.Dialogs.DialogState, Microsoft.Bot.Builder.Dialogs","DialogStack":{"$type":"System.Collections.Generic.List`1[[Microsoft.Bot.Builder.Dialogs.DialogInstance, Microsoft.Bot.Builder.Dialogs]], System.Private.CoreLib","$values":[{"$type":"Microsoft.Bot.Builder.Dialogs.DialogInstance, Microsoft.Bot.Builder.Dialogs","Id":"mainDialog","State":{"$type":"System.Collections.Generic.Dictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Private.CoreLib","options":null,"values":{"$type":"System.Collections.Generic.Dictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Private.CoreLib"},"instanceId":"f80db88d-cdea-4b47-a3f6-a5bfa26ed60b","stepIndex":0}}]}},"PictureBotAccessors.PictureState":{"$type":"Microsoft.PictureBot.PictureState, PictureBot","Greeted":"greeted","UtteranceList":{"$type":"System.Collections.Generic.List`1[[System.String, System.Private.CoreLib]], System.Private.CoreLib","$values":["help"]},"Search":"","Searching":"no"}}
```

> 문제가 발생할 경우 [/code/PictureBot-FinishedSolution-File](./code/PictureBot-FinishedSolution-File)에서 이 지점까지 발생하는 랩 문제에 대한 해결 방법을 찾을 수 있습니다. `appsettings.json` 파일에 Azure Bot Service 및 Azure Storage 설정에 대한 키를 삽입해야 합니다. 이 코드는 실행할 솔루션이 아닌 참조로 사용하는 것이 좋지만, 실행하도록 선택하려는 경우 필요한 키를 추가해야 합니다.

## 다음 단계

데이터베이스 스토리지 및 테스트를 로깅 솔루션에 통합하려면 이 솔루션을 기반으로 하는 다음과 같은 자가 주도 자습서를 살펴보십시오. [Cosmos에 데이터 저장](https://github.com/Azure/LearnAI-Bootcamp/blob/master/lab02.5-logging_chat_conversations/3_Cosmos.md).

## 다음 단계

-   [랩 05-01: QnA Maker](../Lab5-QnA/01-Introduction.md)
