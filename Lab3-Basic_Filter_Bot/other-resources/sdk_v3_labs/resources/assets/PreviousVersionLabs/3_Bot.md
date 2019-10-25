## 3_Bot:
예상 시간: 30-40분

## 봇 빌드

Bot Framework를 사용해본 경험이 있을 줄로 생각하는데, 그렇다면 잘 된 일이고 그렇지 않더라도 너무 염려할 필요 없습니다. 이 섹션에서 많은 것을 배울 수 있습니다. [이 Microsoft Virtual Academy 과정](https://mva.microsoft.com/ko-kr/training-courses/creating-bots-in-the-microsoft-bot-framework-using-c-17590#!)을 이수하고 [설명서](https://docs.microsoft.com/ko-kr/bot-framework/)를 살펴보시기 바랍니다.

### 랩: 봇 개발 준비

우리는 C# SDK를 사용하여 봇을 개발할 것입니다.  시작하려면 다음 두 가지가 필요합니다.
1. [여기](http://aka.ms/bf-bc-vstemplate)에서 Bot Framework 프로젝트 템플릿을 다운로드합니다.  이 파일은 "Bot Application.zip"이라고 하며 \Documents\Visual Studio 2017\Templates\ProjectTemplates\Visual C#\ 디렉터리에 저장해야 합니다.  압축된 전체 파일을 이 디렉터리에 놓으면 되며, 압축을 풀 필요가 없습니다.  
2. 로컬에서 봇을 테스트하도록 [여기](https://github.com/Microsoft/BotFramework-Emulator/releases/download/v3.5.33/botframework-emulator-Setup-3.5.33.exe)에서 Bot Framework Emulator를 다운로드합니다.  이 에뮬레이터는 `c:\Users\`_your-username_`\AppData\Local\botframework\app-3.5.33\botframework-emulator.exe`에 설치됩니다. 

### 랩: 단순한 봇 만들기 및 실행

Visual Studio에서 파일 --> 새 프로젝트로 이동하여 "PictureBot"이라는 봇 응용 프로그램을 만듭니다. "PictureBot"으로 이름을 지정해야 합니다. 그러지 않으면 나중에 문제가 발생할 수 있습니다.  

![새로운 봇 응용 프로그램](./resources/assets/NewBotApplication.png) 

>**단순한 봇 만들기 및 실행** 랩의 나머지 부분은 선택 사항입니다. 전제 조건에 따라 Bot Framework를 사용한 경험이 있어야 합니다. F5 키를 눌러 올바르게 빌드된 것을 확인한 후 다음 랩으로 이동할 수 있습니다.

메시지를 반복하고 문자 길이를 알려주는 에코 봇인 샘플 봇 코드를 살펴봅니다.  특히 다음 사항을 참고하십시오.
+ App_Start 아래의 **WebApiConfig.cs** 에서 경로 템플릿은 api/{controller}/{id}(id는 선택 사항)입니다.  이 때문에 봇의 끝점을 호출할 때 항상 끝에 api/messages를 추가합니다.  
+ Controllers 아래의 **MessagesController.cs** 는 봇으로의 진입점입니다. 봇은 다양한 활동 유형에 응답할 수 있으며 메시지를 보내면 RootDialog가 호출됩니다.  
+ Dialogs 아래의 **RootDialog.cs** 에서 "StartAsync"는 사용자의 메시지를 기다리는 진입점이며, "MessageReceivedAsync"는 받은 메시지를 처리한 후 추가 메시지를 기다리는 메서드입니다.  "context.PostAsync"를 사용하여 봇에서 사용자에게 메시지를 다시 보낼 수 있습니다.  

F5 키를 클릭하여 샘플 코드를 실행합니다.  적절한 종속 항목을 다운로드하는 것은 NuGet을 통해 처리됩니다.  

코드는 기본 웹 브라우저를 사용하여 http://localhost:3979/와 유사한 URL에서 시작됩니다.  

> 참고: 왜 이 포트 번호를 사용할까요?  이는 프로젝트 속성에서 설정됩니다.  솔루션 탐색기에서 "속성"을 두 번 클릭하고 "웹" 탭을 선택합니다.  프로젝트 URL은 "서버" 섹션에 설정되어 있습니다.  

![봇 프로젝트 URL](./resources/assets/BotProjectUrl.jpg) 

프로젝트가 여전히 실행 중인지 확인하고(프로젝트 속성을 보기 위해 중지한 경우 F5 키를 다시 누름) Bot Framework Emulator를 시작합니다.  방금 설치한 경우 아직 인덱싱되지 않아 로컬 컴퓨터에서 검색에 표시되지 않을 수 있습니다. c:\User\User\AppData\Local\botframework\app-3.5.27\botframework-emulator.exe에 설치된다는 점을 기억하십시오.  봇 URL이 위에서 코드가 시작된 포트 번호와 일치하고 끝에 api/messages가 추가되어 있어야 합니다.  봇과 대화할 수 있습니다.  

![Bot Emulator](./resources/assets/BotEmulator.png) 

### 랩: LUIS를 사용하도록 봇 업데이트

LUIS를 사용하도록 봇을 업데이트해야 합니다.  [LuisDialog 클래스](https://docs.botframework.com/ko-kr/csharp/builder/sdkreference/d8/df9/class_microsoft_1_1_bot_1_1_builder_1_1_dialogs_1_1_luis_dialog.html)를 사용하여 이 작업을 수행할 수 있습니다.  

**RootDialog.cs** 파일에서 다음 네임스페이스에 참조를 추가합니다.

```csharp

using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

```

다음으로, RootDialog 클래스가 IDialog<object>가 아닌 LuisDialog<object>에서 파생되도록 변경합니다.  다음으로 LUIS 앱 ID 및 LUIS 키를 사용하여 이 클래스에 LuisModel 특성을 지정합니다.  이러한 값을 찾을 수 없는 경우 http://luis.ai로 돌아가서  응용 프로그램을 클릭하고 "앱 게시" 페이지로 이동합니다. 끝점 URL에서 LUIS 앱 ID 및 LUIS 키를 얻을 수 있습니다. (힌트: LUIS 앱 ID에는 하이픈이 있으며 LUIS 키에는 없습니다.)

```csharp

using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

namespace PictureBot.Dialogs
{
    [LuisModel("96f65e22-7dcc-4f4d-a83a-d2aca5c72b24", "1234bb84eva3481a80c8a2a0fa2122f0")]
    [Serializable]
    public class RootDialog : LuisDialog<object>
    {

```

> 참고: [Autofac](https://autofac.org/)을 사용하면 클래스에 LuisModel 특성을 하드 코딩하는 대신 동적으로 로드하여 구성 파일에 올바르게 저장할 수 있습니다.  [AlarmBot 샘플](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Samples/AlarmBot/Models/AlarmModule.cs#L24)에 이에 대한 예가 있습니다.  

다음으로 클래스의 두 기존 메서드(StartAsync 및 MessageReceivedAsync)를 삭제합니다.  LuisDialog에는 LUIS 서비스를 호출하고 응답을 기반으로 적절한 메서드로 라우팅하는 StartAsync의 구현이 이미 있습니다.  

마지막으로 각 의도에 대한 메서드를 추가합니다.  가장 높은 점수의 의도에 대해 해당 메서드가 호출됩니다.  먼저 각 의도에 대한 간단한 메시지를 표시해 보겠습니다.  

```csharp

        [LuisIntent("")]
        [LuisIntent("None")]
        public async Task None(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Hmmmm, I didn't understand that.  I'm still learning!");
        }

        [LuisIntent("Greeting")]
        public async Task Greeting(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Hello!  I am a Photo Organization Bot.  I can search your photos, share your photos on Twitter, and order prints of your photos.  You can ask me things like 'find pictures of food'.");
        }

        [LuisIntent("SearchPics")]
        public async Task SearchPics(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Searching for your pictures...");
        }

        [LuisIntent("OrderPic")]
        public async Task OrderPic(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Ordering your pictures...");
        }

        [LuisIntent("SharePic")]
        public async Task SharePic(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Posting your pictures to Twitter...");
        } 

```

코드를 수정한 후에는 F5 키를 눌러 Visual Studio에서 실행하고 Bot Framework Emulator에서 새로운 대화를 시작합니다.  봇과 채팅을 시도하고 예상 응답을 받을 수 있는지 확인합니다.  예기치 않은 결과가 발생하면 이를 기록하고 이후에 LUIS를 수정합니다.  

![봇 테스트 LUIS](./resources/assets/BotTestLuis.jpg) 

위의 스크린샷에서 봇에 "인쇄물 주문"이라고 말했을 때 다른 응답을 받을 것으로 예상했습니다. "OrderPic" 의도 대신 "SearchPics" 의도에 매핑된 것 같습니다.  http://luis.ai로 돌아가서 LUIS 모델을 업데이트할 수 있습니다.  해당 응용 프로그램을 클릭한 다음 왼쪽 사이드바에서 "의도"를 클릭합니다.  수동으로 새 발화로 이를 추가하거나 LUIS의 "발화 제안" 기능을 활용하여 모델을 개선할 수 있습니다.  "SearchPics" 의도(또는 발화 레이블이 잘못 지정된 의도)를 클릭한 다음 "발화 제안"을 클릭합니다.  레이블이 잘못 지정된 발화의 확인란을 클릭한 다음 "의도 재할당"을 클릭하고 올바른 의도를 선택합니다.  

![LUIS 의도 재할당](./resources/assets/LuisReassignIntent.jpg) 

봇에 이러한 변경 내용을 적용하려면 LUIS 모델을 다시 학습시키고 다시 게시해야 합니다.  왼쪽 사이드바에서 "앱 게시"를 클릭하고 "학습" 단추를 클릭한 다음 하단의 "게시" 단추를 클릭합니다.  그런 다음 에뮬레이터에서 봇으로 돌아가 다시 시도할 수 있습니다.  

> 참고: 발화 제안 기능은 매우 강력합니다.  LUIS는 표시할 발화에 대해 스마트한 결정을 내립니다.  루프의 사람을 통해 수동으로 레이블이 지정되어 최대한 개선 가능한 발화를 선택합니다.  예를 들어 주어진 발화가 47% 신뢰 수준으로 Intent1에 매핑되고 48% 신뢰 수준으로 Intent2에 매핑될 것으로 LUIS 모델이 예측한 경우, 후자가 사람에게 표시하여 수동으로 매핑할 강력한 후보가 됩니다. 이 모델에서는 두 의도 간 차이가 거의 없기 때문입니다.  

이제 LUIS 모델을 사용하여 사용자의 의도를 파악할 수 있으므로 Azure Search를 통합하여 사진을 찾아보겠습니다.  

### 랩: Azure Search를 사용하도록 봇 구성 

먼저 Azure Search 인덱스에 연결하기 위한 관련 정보를 봇에 제공해야 합니다.  연결 정보를 저장하는 가장 좋은 위치는 구성 파일입니다.  

Web.config를 열고 appSettings 섹션에서 다음을 추가합니다.

```xml    
    <!-- Azure Search Settings -->
    <add key="SearchDialogsServiceName" value="" />
    <add key="SearchDialogsServiceKey" value="" />
    <add key="SearchDialogsIndexName" value="images" />
```

SearchDialogsServiceName의 값을 앞에서 만든 Azure Search 서비스의 이름으로 설정합니다.  필요한 경우 [Azure Portal](https://portal.azure.com)로 돌아가서 이를 확인합니다.  

SearchDialogsServiceKey의 값을 이 서비스의 키로 설정합니다.  이 값은 [Azure Portal](https://portal.azure.com)의 Azure Search 키 섹션 아래에서 찾을 수 있습니다.  아래 스크린샷에서 SearchDialogsServiceName은 "aiimmersionsearch"이고 SearchDialogsServiceKey는 "375..."입니다.  

![Azure Search 설정](./resources/assets/AzureSearchSettings.jpg) 

### 랩: Azure Search를 사용하도록 봇 업데이트

다음으로 Azure Search를 호출하도록 봇을 업데이트하겠습니다.  먼저 도구-->NuGet Package Manager-->솔루션용 NuGet 패키지 관리를 선택합니다.  검색 상자에 "Microsoft.Azure.Search"를 입력합니다.  해당 라이브러리를 선택하고 프로젝트를 나타내는 확인란을 선택한 후 설치합니다.  다른 종속성 항목도 설치할 수 있습니다. 설치된 패키지에서 "Newtonsoft.Json" 패키지도 업데이트해야 할 수 있습니다.

![Azure Search NuGet](./resources/assets/AzureSearchNuGet.jpg) 

Visual Studio의 솔루션 탐색기에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 추가-->새 폴더를 선택합니다.  "Models"라는 폴더를 만듭니다.  "Models" 폴더를 마우스 오른쪽 단추로 클릭하고 추가-->기존 항목을 선택합니다.  이 작업을 두 번 수행하여 Models 폴더 아래에 다음 두 파일을 추가합니다. 필요한 경우 네임스페이스를 조정하십시오.
1. [ImageMapper.cs](./resources/code/Models/ImageMapper.cs)
2. [SearchHit.cs](./resources/code/Models/SearchHit.cs)

>이 리포지토리의 [resources/code/Models](./resources/code/Models) 아래에서 이러한 파일을 찾을 수 있습니다.

다음으로 Visual Studio의 솔루션 탐색기에서 Dialogs 폴더를 마우스 오른쪽 단추로 클릭하고 추가-->클래스를 선택합니다.  클래스 이름을 "SearchDialog.cs"로 지정합니다. [여기](./resources/code/SearchDialog.cs)에서 콘텐츠를 추가합니다.

마지막으로, SearchDialog를 호출하도록 RootDialog를 업데이트해야 합니다.  Dialogs 폴더의 RootDialog.cs에서 SearchPics 메서드를 업데이트하고 다음과 같은 "ResumeAfter" 메서드를 추가합니다.

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

        private async Task ResumeAfterSearchTopicClarification(IDialogContext context, IAwaitable<string> result)
        {
            string searchTerm = await result;
            context.Call(new SearchDialog(searchTerm), ResumeAfterSearchDialog);
        }

        private async Task ResumeAfterSearchDialog(IDialogContext context, IAwaitable<object> result)
        {
            await context.PostAsync("Done searching pictures");
        }

```

F5 키를 눌러 봇을 다시 실행합니다.  Bot Emulator에서 "개 사진 찾기" 또는 "행복에 대한 사진 검색"을 사용해 검색해 보십시오.  사진의 태그가 요청될 때 결과가 표시되는지 확인합니다.  


### [4_Bot_Enhancements](./4_Bot_Enhancements.md)로 계속 진행

[README](./0_README.md)로 돌아가기