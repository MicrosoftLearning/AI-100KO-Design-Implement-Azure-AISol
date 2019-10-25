## 3_LUIS:
예상 시간: 15-20분

### 랩 3.1: LUIS를 사용하도록 봇 업데이트

LUIS를 사용하도록 봇을 업데이트해야 합니다.  [LuisDialog 클래스](https://docs.botframework.com/ko-kr/csharp/builder/sdkreference/d8/df9/class_microsoft_1_1_bot_1_1_builder_1_1_dialogs_1_1_luis_dialog.html)를 사용하여 이 작업을 수행할 수 있습니다.  

**RootDialog.cs** 파일에서 다음 네임스페이스에 참조를 추가합니다.

```csharp

using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

```

다음으로 LUIS 앱 ID 및 LUIS 키를 사용하여 이 클래스에 LuisModel 특성을 지정합니다.  이러한 값을 찾을 수 없는 경우 http://luis.ai로 돌아가서  응용 프로그램을 클릭하고 "앱 게시" 페이지로 이동합니다. 끝점 URL에서 LUIS 앱 ID 및 LUIS 키를 얻을 수 있습니다. (힌트: LUIS 앱 ID에는 하이픈이 있으며 LUIS 키에는 없습니다.) LUIS 앱 ID 및 LUIS 키를 사용하여 `[Serializable]` 바로 위에 다음 코드 줄을 배치해야 합니다.

```csharp

namespace PictureBot.Dialogs
{
    [LuisModel("YOUR-APP-ID", "YOUR-SUBSCRIPTION-KEY")]
    [Serializable]

...
```

> 참고: [Autofac](https://autofac.org/)을 사용하면 클래스에 LuisModel 특성을 하드 코딩하는 대신 동적으로 로드하여 구성 파일에 올바르게 저장할 수 있습니다.  [AlarmBot 샘플](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Samples/AlarmBot/Models/AlarmModule.cs#L24)에 이에 대한 예가 있습니다.  


먼저 "Greeting" 의도를 추가하겠습니다(DispatchDialog 내에 이미 배치한 모든 코드 아래).

```csharp
        [LuisIntent("Greeting")]
        [ScorableGroup(1)]
        public async Task Greeting(IDialogContext context, LuisResult result)
        {
            // Scorables에 대해 학습시키기 위한 중복 논리입니다.  
            await context.PostAsync("Hello from LUIS!  I am a Photo Organization Bot.  I can search your photos, share your photos on Twitter, and order prints of your photos.  You can ask me things like 'find pictures of food'.");
        }
```

F5 키를 눌러 앱을 실행합니다. Bot Emulator에서 봇에게 다양한 인사말을 보내 보십시오. "안녕하십니까" 대신 "잘 지내세요?"를 보내면 어떻게 되나요?

LUIS의 "None" 의도는 발화가 의도에 매핑되지 않았음을 의미합니다.  이 경우 다음 수준의 ScorableGroup으로 이동합니다.  RootDialog 클래스에서 다음과 같이 "None" 메서드를 추가합니다.
```csharp
        [LuisIntent("")]
        [LuisIntent("None")]
        public async Task None(IDialogContext context, LuisResult result)
        {
            // LUIS가 최우선 의도로 "None"을 반환했으므로
            // 다음 수준의 ScorableGroups로 이동합니다.  
            ContinueWithNextGroup();
        }
```

마지막으로 나머지 의도에 대한 메서드를 추가합니다.  가장 높은 점수의 의도에 대해 해당 메서드가 호출됩니다. "SearchPics" 및 "SharePics" 코드에 대해 동료와 이야기합니다. 코드가 수행하는 기타 작업은 무엇입니까? 


```csharp
        [LuisIntent("SearchPics")]
        public async Task SearchPics(IDialogContext context, LuisResult result)
        {
            // LUIS에서 찾아야 할 검색어를 식별했는지 확인합니다.  
            string facet = null;
            EntityRecommendation rec;
            if (result.TryFindEntity("facet", out rec)) facet = rec.Entity;

            // 검색할 항목을 알 수 없는 경우(예: 사용자가
            // "x의 사진 찾기" 대신 "사진 찾기" 또는 "검색"이라고 말함)
            // 검색어를 입력하라는 메시지를 표시합니다.  
            if (string.IsNullOrEmpty(facet))
            {
                PromptDialog.Text(context, ResumeAfterSearchTopicClarification,
                    "What kind of picture do you want to search for?");
            }
            else
            {
                await context.PostAsync("Searching pictures...");
                context.Call(new SearchDialog(facet), ResumeAfterSearchDialog);
            }
        }

        [LuisIntent("OrderPic")]
        public async Task OrderPic(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Ordering your pictures...");
        }

        [LuisIntent("SharePic")]
        public async Task SharePic(IDialogContext context, LuisResult result)
        {
            PromptDialog.Confirm(context, AfterShareAsync,
                "Are you sure you want to tweet this picture?");
        }

        private async Task AfterShareAsync(IDialogContext context, IAwaitable<bool> result)
        {
            if (result.GetAwaiter().GetResult() == true)
            {
                // 예, 사진을 공유합니다.
                await context.PostAsync("Posting tweet.");
            }
            else
            {
                // 아니요, 사진을 공유하지 않습니다.  
                await context.PostAsync("OK, I won't share it.");
            }
        }


```



> 힌트: "SharePic" 메서드에는 예/아니요 확인을 위한 메시지를 표시하는 방법과 ScorableGroup을 설정하는 방법을 보여주는 작은 코드가 포함되어 있습니다. 이 코드는 실제로 트윗을 게시하지는 않습니다. 왜냐하면 Twitter 개발자 계정 등을 설정하는 데 시간을 할애하는 것을 원치 않기 때문입니다. 하지만 원한다면 구현할 수 있습니다.

방금 추가한 의도에 대해서는 Scorable Groups를 구현하지 않았습니다. 모든 `LuisIntent`가 Scorable Groups 우선 순위 1이 되도록 코드를 수정하십시오. LUIS 기능을 추가한 후에는 `ResumeAfterChoice` 메서드에서 두 개의 `await` 줄에 대한 주석을 해제할 수 있습니다.  

코드를 수정한 후에는 F5 키를 눌러 Visual Studio에서 실행하고 Bot Framework Emulator에서 새로운 대화를 시작합니다.  봇과 채팅을 시도하고 예상 응답을 받을 수 있는지 확인합니다.  예기치 않은 결과가 발생하면 이를 기록하고 이후에 [LUIS를 수정합니다](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/Train-Test).

LUIS 모델을 다시 학습시키고 다시 게시해야 한다는 것을 잊지 마십시오.  그런 다음 에뮬레이터에서 봇으로 돌아가 다시 시도할 수 있습니다.  

> 참고: 발화 제안 기능은 매우 강력합니다.  LUIS는 표시할 발화에 대해 스마트한 결정을 내립니다.  루프의 사람을 통해 수동으로 레이블이 지정되어 최대한 개선 가능한 발화를 선택합니다.  예를 들어 주어진 발화가 47% 신뢰 수준으로 Intent1에 매핑되고 48% 신뢰 수준으로 Intent2에 매핑될 것으로 LUIS 모델이 예측한 경우, 후자가 사람에게 표시하여 수동으로 매핑할 강력한 후보가 됩니다. 이 모델에서는 두 의도 간 차이가 거의 없기 때문입니다.  


> 추가 크레딧(나중에 완료): "Dialogs" 폴더에 OrderDialog 클래스를 만듭니다.  [FormFlow](https://docs.botframework.com/ko-kr/csharp/builder/sdkreference/forms.html)를 사용하여 봇을 통해 인쇄물을 주문하는 프로세스를 만듭니다.  봇은 다음 정보를 수집해야 합니다. 사진 크기(8x10, 5x7, 지갑 등), 인쇄물 수, 광택 또는 무광택 마감, 사용자의 전화 번호 및 사용자의 전자 메일.



마지막으로, 위의 서비스 중 어느 것도 파악하지 못하는 경우를 위한 기본 처리기를 추가합니다.  이 Scorable Group은 LuisIntent 또는 RegexPattern 특성(MethodBind 포함)이 추가되지 않으므로 명시적인 MethodBind가 필요합니다.

```csharp

        // 이전 그룹의 Scorables 중 어느 것도 파악하지 못했으므로 대화 상자에 도움말 메시지를 보냅니다.
        [MethodBind]
        [ScorableGroup(2)]
        public async Task Default(IDialogContext context, IActivity activity)
        {
            await context.PostAsync("I'm sorry. I didn't understand you.");
            await context.PostAsync("You can tell me to find photos, tweet them, and order prints.  예를 들면 다음과 같습니다. \"find pictures of food\".");
        }

```

F5 키를 눌러 봇을 실행하고 Bot Emulator에서 테스트합니다.  



### [4_Publish_and_Register](./4_Publish_and_Register.md)로 계속 진행  
[README](./0_README.md)로 돌아가기