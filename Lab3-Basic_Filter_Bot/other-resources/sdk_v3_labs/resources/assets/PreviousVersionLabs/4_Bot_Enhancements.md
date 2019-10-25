## 4_봇 향상:
예상 시간: 20-30분

### 랩: 정규식 및 Scorable Groups

봇을 개선하기 위해 할 수 있는 일이 여러 가지 있습니다. 우선, 봇이 사용자로부터 상당히 자주 수신하는 간단한 "안녕하세요" 인사말을 위해 LUIS를 호출하는 것은 좋지 않습니다.  단순한 정규식이 이와 일치할 수 있으며 네트워크 대기 시간으로 인한 시간과 LUIS 서비스를 호출하는 데 드는 비용을 줄여 줍니다.  

또한 봇의 복잡성이 증가하는 동시에, 사용자의 입력을 받고 다양한 서비스를 사용하여 해석하기 때문에 이 흐름을 관리하는 프로세스가 필요합니다.  예를 들어 정규식을 먼저 시도하고 일치하지 않는 경우 LUIS를 호출하고, 이후에는 [QnA Maker](http://qnamaker.ai) 및 Azure Search 등의 다른 서비스를 사용해 볼 수 있습니다.  이를 관리하는 좋은 방법은 [ScorableGroups](https://blog.botframework.com/2017/07/06/Scorables/)를 사용하는 것입니다.  ScorableGroups는 이러한 서비스 호출에 순서를 설정하는 특성을 제공합니다.  코드에서 먼저 정규식과 일치시킨 다음, 발화 해석을 위해 LUIS를 호출하고, 마지막으로 일반적인 "무슨 말씀이신지 잘 모르겠습니다." 같은 응답을 제시하는 순서를 설정해 보겠습니다.    

ScorableGroups를 사용하려면 RootDialog가 LuisDialog 대신 DispatchDialog에서 상속해야 합니다(하지만 클래스에서 LuisModel 특성을 여전히 가질 수 있음).  또한 Microsoft.Bot.Builder.Scorables(및 기타)에 대한 참조가 필요합니다.  따라서 RootDialog.cs 파일에 다음을 추가합니다.

```csharp

using Microsoft.Bot.Builder.Scorables;
using System.Collections.Generic;

```

그리고 클래스 파생을 다음으로 변경합니다.

```csharp

    public class RootDialog : DispatchDialog<object>

```

그런 다음 ScorableGroup 0의 첫 번째 우선 순위로 정규식과 일치하는 몇 가지 새로운 메서드를 추가해 보겠습니다.  RootDialog 클래스의 시작 부분에 다음을 추가합니다.

```csharp

        [RegexPattern("^hello")]
        [RegexPattern("^hi")]
        [ScorableGroup(0)]
        public async Task Hello(IDialogContext context, IActivity activity)
        {
            await context.PostAsync("Hello from RegEx!  I am a Photo Organization Bot.  I can search your photos, share your photos on Twitter, and order prints of your photos.  You can ask me things like 'find pictures of food'.");
        }

        [RegexPattern("^help")]
        [ScorableGroup(0)]
        public async Task Help(IDialogContext context, IActivity activity)
        {
            // 단추 메뉴로 도움말 대화 상자를 시작합니다.  
            List<string> choices = new List<string>(new string[] { "Search Pictures", "Share Picture", "Order Prints" });
            PromptDialog.Choice<string>(context, ResumeAfterChoice, 
                new PromptOptions<string>("How can I help you?", options:choices));
        }

        private async Task ResumeAfterChoice(IDialogContext context, IAwaitable<string> result)
        {
            string choice = await result;
            
            switch (choice)
            {
                case "Search Pictures":
                    PromptDialog.Text(context, ResumeAfterSearchTopicClarification,
                        "What kind of picture do you want to search for?");
                    break;
                case "Share Picture":
                    await SharePic(context, null);
                    break;
                case "Order Prints":
                    await OrderPic(context, null);
                    break;
                default:
                    await context.PostAsync("I'm sorry. I didn't understand you.");
                    break;
            }
        }

```

이 코드는 "안녕하세요", "안녕하십니까"및 "도움말"로 시작하는 사용자의 표현과 일치시킵니다.  그리고 사용자가 도움을 요청할 때, 봇이 할 수 있는 3가지 핵심 작업인 사진 검색, 사진 공유, 인쇄물 주문에 대한 간단한 단추 메뉴를 표시합니다.  

> 참고: 봇이 할 수 있는 일에 대한 명확한 옵션의 메뉴를 얻기 위해 사용자가 "도움말"을 입력할 필요가 없이, 봇과 처음 접촉할 때 기본 환경에 이것이 포함되어야 한다고 생각할 수 있습니다. **검색 기능**, 즉 봇이 할 수 있는 일을 사용자가 알도록 하는 것은 봇의 최대 과제 중 하나입니다.  적절한 [봇 디자인 원칙](https://docs.microsoft.com/ko-kr/bot-framework/bot-design-principles)이 도움이 될 수 있습니다.   

이 설정을 사용하면 정규식이 일치하지 않을 경우 두 번째 시도로 Scorable Group 1에서 LUIS를 호출할 수 있습니다.  

LUIS의 "None" 의도는 발화가 의도에 매핑되지 않았음을 의미합니다.  이 경우 다음 수준의 ScorableGroup으로 이동합니다.  RootDialog 클래스에서 다음과 같이 "None" 메서드를 수정합니다.

```csharp

        [LuisIntent("")]
        [LuisIntent("None")]
        [ScorableGroup(1)]
        public async Task None(IDialogContext context, LuisResult result)
        {
            // LUIS가 최우선 의도로 "None"을 반환했으므로
            // 다음 수준의 ScorableGroups로 이동합니다.  
            ContinueWithNextGroup();
        }

```

"Greeting" 메서드에서 ScorableGroup 특성을 추가하고 "from LUIS"를 추가하여 차별화합니다.  코드를 실행할 때 "안녕하세요" 및 "안녕하십니까"(RegEx 일치 항목을 통해 포착할 수 있음)를 말한 다음 "반갑습니다" 또는 "여보세요"(어떻게 학습시키는지에 따라 LUIS에서 포착할 수 있음)라고 말하십시오.  어떤 메서드가 응답하는지 확인합니다.  

```csharp

        [LuisIntent("Greeting")]
        [ScorableGroup(1)]
        public async Task Greeting(IDialogContext context, LuisResult result)
        {
            // Scorables에 대해 학습시키기 위한 중복 논리입니다.  
            await context.PostAsync("Hello from LUIS!  I am a Photo Organization Bot.  I can search your photos, share your photos on Twitter, and order prints of your photos.  You can ask me things like 'find pictures of food'.");
        }

```

그런 다음 "SearchPics" 메서드 및 "OrderPic" 메서드에 ScorableGroup 특성을 추가합니다.  

```csharp

        [LuisIntent("SearchPics")]
        [ScorableGroup(1)]
        public async Task SearchPics(IDialogContext context, LuisResult result)
        {
            ...
        }

        [LuisIntent("OrderPic")]
        [ScorableGroup(1)]
        public async Task OrderPic(IDialogContext context, LuisResult result)
        {
            ...
        }

```

> 추가 크레딧(나중에 완료): "Dialogs" 폴더에 OrderDialog 클래스를 만듭니다.  [FormFlow](https://docs.botframework.com/ko-kr/csharp/builder/sdkreference/forms.html)를 사용하여 봇을 통해 인쇄물을 주문하는 프로세스를 만듭니다.  봇은 다음 정보를 수집해야 합니다. 사진 크기(8x10, 5x7, 지갑 등), 인쇄물 수, 광택 또는 무광택 마감, 사용자의 전화 번호 및 사용자의 전자 메일.

"SharePic" 메서드도 업데이트할 수 있습니다.  아래에는 예/아니요 확인을 위한 메시지를 표시하는 방법과 ScorableGroup을 설정하는 방법을 보여주는 작은 코드가 포함되어 있습니다.  이 코드는 실제로 트윗을 게시하지는 않습니다. 왜냐하면 Twitter 개발자 계정 등을 설정하는 데 시간을 할애하는 것을 원치 않기 때문입니다. 하지만 원한다면 구현할 수 있습니다.  

```csharp

        [LuisIntent("SharePic")]
        [ScorableGroup(1)]
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

### 랩: 봇 게시

Microsoft Bot을 사용하여 만든 봇은 공개적으로 액세스할 수 있는 모든 URL에서 호스트할 수 있습니다.  이 랩에서는 Azure 웹 사이트/App Service에서 봇을 호스팅합니다.  

Visual Studio의 솔루션 탐색기에서 봇 응용 프로그램 프로젝트를 마우스 오른쪽 단추로 클릭하고 "게시"를 선택합니다.  이렇게 하면 Azure에 봇을 게시하는 과정을 안내하는 마법사가 시작됩니다.  

게시 대상을 "Microsoft Azure App Service"로 선택합니다.  

![Azure App Service에 봇 게시](./resources/assets/PublishBotAzureAppService.png) 

App Service 화면에서 적절한 구독을 선택하고 "새로 만들기"를 클릭합니다. 그런 다음 API 앱 이름, 구독, 지금까지 사용한 것과 동일한 리소스 그룹 및 App Service 계획을 입력합니다.  

![App Service 만들기](./resources/assets/CreateAppService.jpg) 

마지막으로 웹 배포 설정이 표시되고 "게시"를 클릭할 수 있습니다.  Visual Studio의 출력 창에 배포 프로세스가 표시되고  http://testpicturebot.azurewebsites.net/ 같은 URL에 봇이 호스팅됩니다. 여기서 "testpicturebot"은 App Service API 앱 이름입니다.  

### 랩: 봇 커넥터에 봇 등록

웹 브라우저에서 [http://dev.botframework.com](http://dev.botframework.com)으로 이동합니다.  [봇 등록](https://dev.botframework.com/bots/new)을 클릭하고 봇의 이름, 핸들 및 설명을 입력합니다.  메시지 끝점은 끝에 "api/messages"가 추가된 Azure 웹 사이트 URL(예: https://testpicturebot.azurewebsites.net/api/messages)입니다.  

![봇 등록](./resources/assets/BotRegistration.jpg) 

그런 다음 Microsoft 앱 ID 와 암호를 만드는 단추를 클릭합니다.  Web.config에 필요한 봇 앱 ID 및 암호입니다.  Bot 앱 이름, 앱 ID 및 앱 암호를 안전한 장소에 저장하십시오!  암호에 대해 "확인"을 클릭한 후에는 다시 돌아갈 방법이 없습니다.  그런 다음 "완료하고 Bot Framework로 돌아가기"를 클릭합니다.  

![봇 앱 이름, ID 및 암호 생성](./resources/assets/BotGenerateAppInfo.jpg) 

봇 등록 페이지에 앱 ID가 자동으로 채워져 있을 것입니다. 봇에서 로깅하기 위해 선택적으로 AppInsights 계측 키를 추가할 수 있습니다. 서비스 약관에 동의하는 경우 확인란을 선택하고 "등록"을 클릭합니다.  

그러면 https://dev.botframework.com/bots?id=TestPictureBot과 비슷하지만 실제 봇 이름을 포함한 URL의 봇 대시보드 페이지가 표시됩니다. 여기서는 다양한 채널을 사용하도록 설정할 수 있습니다.  Skype와 웹 채팅의 두 채널이 자동으로 사용하도록 설정되어 있습니다.  

마지막으로, 등록 정보를 사용하여 봇을 업데이트해야 합니다.  Visual Studio로 돌아가 Web.config를 엽니다.  BotId를 앱 이름으로, MicrosoftAppId를 앱 ID로, MicrosoftAppPassword를 봇 등록 사이트에서 얻은 앱 암호로 업데이트합니다.  

```xml

    <add key="BotId" value="TestPictureBot" />
    <add key="MicrosoftAppId" value="95b76ae6-8643-4d94-b8a1-916d9f753a30" />
    <add key="MicrosoftAppPassword" value="kC200000000000000000000" />

```

프로젝트를 다시 빌드한 다음 솔루션 탐색기에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 다시 "게시"를 선택합니다.  마지막 설정이 기억되므로 "게시"를 누르기만 하면 됩니다. 

> 오류가 발생하여 MicrosoftAppPassword를 입력하라는 메시지가 표시됩니까? XML이므로 키에 "&", "<", ">", "'" 또는 '"'가 포함된 경우 이러한 기호를 다음과 같은 [이스케이프 기능](https://en.wikipedia.org/wiki/XML#Characters_and_escaping)으로 바꿔야 합니다. "&amp;", "&lt;", "&gt;", "&apos;", "&quot;". 

봇의 대시보드(https://dev.botframework.com/bots?id=TestPictureBot)로 돌아갑니다.  채팅 창에서 대화해 보십시오.  웹 채팅에서는 캐러셀이 에뮬레이터에서와 다르게 보일 수 있습니다.  다양한 채널에서 다양한 컨트롤의 사용자 경험을 확인할 수 있는 Channel Inspector라는 훌륭한 도구가 https://docs.botframework.com/ko-kr/channel-inspector/channels/Skype/#navtitle에 있습니다.  
봇의 대시보드에서 다른 채널을 추가하고 Skype, Facebook Messenger 또는 Slack에서 봇을 사용해 볼 수 있습니다.  봇 대시보드의 채널 이름 오른쪽에 있는 "추가" 단추를 클릭하고 지침을 따르기만 하면 됩니다.


### [5_Challenge_and_Closing](./5_Challenge_and_Closing.md)로 계속 진행

[README](./0_README.md)로 돌아가기