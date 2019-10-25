---
lab:
    title: '랩 9: DirectLine에서 봇 테스트'
    module: '모듈 2: 봇 만들기'
---

# 랩 9: DirectLine에서 봇 테스트

## 랩 9.0: 사전 요건

이 랩은 [랩 2](../Lab2-Basic_Filter_Bot/02-Basic_Filter_Bot.md)에서 봇을 빌드하고 게시했다는 전제하에 시작됩니다. 이어지는 랩에서 성공하려면 해당 랩을 완료하는 것이 좋습니다. 완료하지 않은 경우 필요에 따라 모든 연습을 주의 깊게 읽고 일부 코드를 살펴보거나 자체 응용 프로그램에서 사용하는 것으로 충분할 수 있습니다.

또한 [랩 8](../Lab8-Log_Chat/02-Logging_Chat.md)을 완료했다고 가정하지만 로깅 랩을 완료하지 않고도 랩을 완료할 수 있어야 합니다.

> 참고: 이러한 랩에서는 Microsoft Bot Framework SDK의 v4를 사용합니다. v3 SDK를 사용하여 유사한 랩을 수행하려면 [여기](./sdk_v3_labs)를 참조하십시오.


### 키 수집

이 랩에서는 다양한 키를 수집합니다. 워크샵 전반에서 쉽게 액세스할 수 있도록 모든 키를 텍스트 파일에 저장하는 것이 좋습니다.

> 키_
>- Microsoft 봇 리소스 ID:
>- Microsoft 봇 AppId:
>- Microsoft Bot AppPassword:
>- Direct Line 비밀 키:

## 랩 9.1 봇에 직접 연결

경우에 따라 봇과 직접 통신해야 할 수 있습니다. 예를 들어 호스팅된 봇을 사용하여 기능 테스트를 수행할 수 있습니다. 봇과 사용자 고유의 클라이언트 응용 프로그램 간의 통신은 [Direct Line API](https://docs.microsoft.com/ko-kr/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts)를 사용하여 수행할 수 있습니다.

또한 다른 채널을 사용하거나 봇을 추가로 사용자 지정할 수도 있습니다. 이 경우 Direct Line을 사용하여 사용자 지정 봇과 통신할 수 있습니다.

Microsoft Bot Framework Direct Line 봇은 고유한 사용자 지정 클라이언트에서 작동할 수 있는 봇입니다. Direct Line 봇은 일반 봇과 유사한데 제공된 채널을 사용할 필요가 없습니다. Direct Line 클라이언트는 원하는 대로 작성할 수 있습니다. Android 클라이언트, iOS 클라이언트 또는 콘솔 기반 클라이언트 응용 프로그램을 작성할 수 있습니다.

이 실습 랩에서는 Direct Line API와 관련된 주요 개념을 소개합니다.

## 랩 9.2: Direct Line 채널 설정

포털에서 게시된 PictureBot을 찾고 "채널" 탭으로 이동합니다. Direct Line 아이콘(지구본 모양)을 선택합니다. "기본 사이트"가 표시됩니다. "표시"를 클릭하고 메모장이나 키를 추적하는 위치에 비밀 키 중 하나를 저장합니다. 포털에 있는 동안 봇의 이름 또는 "botId"도 기록하는 것이 좋습니다.

[Direct Line 채널 사용](https://docs.microsoft.com/ko-kr/azure/bot-service/bot-service-channel-connect-directline?view=azure-bot-service-4.0) 및 [비밀 및 토큰](https://docs.microsoft.com/ko-kr/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-3.0#secrets-and-tokens)에 대한 자세한 지침을 확인할 수 있습니다.


## 랩 9.3: 콘솔 응용 프로그램 만들기

Visual Studio에서 게시된 PictureBot 솔루션을 엽니다. Direct Line을 통해 봇에 직접 연결하는 방법을 이해하는 데 도움이 되는 콘솔 응용 프로그램을 만들겠습니다.

우리가 만들 콘솔 클라이언트 응용 프로그램은 두 스레드로 작동합니다. 기본 스레드는 사용자 입력을 받고 봇에 메시지를 보냅니다. 보조 스레드는 봇을 초당 한 번 폴링하여 봇에서 메시지를 검색한 다음 받은 메시지를 표시합니다.

> 참고: 여기에 있는 지침 및 코드는 [설명서의 모범 사례](https://docs.microsoft.com/ko-kr/azure/bot-service/bot-builder-howto-direct-line?view=azure-bot-service-4.0&tabs=cscreatebot%2Ccsclientapp%2Ccsrunclient)에서 수정되었습니다.

**1단계** - 콘솔 프로젝트 만들기

솔루션 탐색기에서 솔루션을 마우스 오른쪽 단추로 클릭한 다음 **추가 > 새 프로젝트** 를 선택합니다. **Visual C# > Windows 클래식 데스크톱** 에서 "PictureBotDL"이라는 제목의 새 **콘솔 앱(.NET Framework)** 을 만들고 **확인** 을 선택합니다.

**2단계** - PictureBotDL에 NuGet 패키지 추가

PictureBotDL 프로젝트를 마우스 오른쪽 단추로 클릭하고 **NuGet 패키지 관리** 를 선택합니다. "찾아보기" 탭(그리고 "시험판 포함" 선택)에서 "Microsoft.Bot.Connector.DirectLine" 및 "Newtonsoft.Json"을 검색하고 설치/업데이트합니다.

**3단계** Program.cs 업데이트

Program.cs 내용(PictureBotDL 내)을 다음과 같은 내용으로 바꿉니다.
```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Connector.DirectLine;
using Newtonsoft.Json;

namespace PictureBotDL
{
    class Program
    {
        // ************
        // 다음 값을 Direct Line 암호 및 봇 리소스 ID 이름으로 바꿉니다.
        //*************
        private static string directLineSecret = "YourDLSecret";
        private static string botId = "YourBotServiceName";

        // 이를 통해 봇 사용자에게 이름을 지정합니다.
        private static string fromUser = "PictureBotSampleUser";

        static void Main(string[] args)
        {
            StartBotConversation().Wait();
        }


        /// <summary>
        /// 봇과 사용자의 대화를 유도합니다.
        /// </summary>
        /// <returns></returns>
        private static async Task StartBotConversation()
        {
            // 새 Direct Line 클라이언트를 만듭니다.
            DirectLineClient client = new DirectLineClient(directLineSecret);

            // 대화를 시작합니다.
            var conversation = await client.Conversations.StartConversationAsync();

            // 별도의 스레드로 봇 메시지 리더를 시작합니다.
            new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

            // 사용자에게 봇과 대화를 시작하라는 메시지를 표시합니다.
            Console.Write("Conversation ID: " + conversation.ConversationId + Environment.NewLine);
            Console.Write("Type your message (or \"exit\" to end): ");

            // 사용자가 이 루프를 종료하도록 선택할 때까지 반복합니다.
            while (true)
            {
                // 사용자의 입력을 받습니다.
                string input = Console.ReadLine().Trim();

                // 사용자가 종료하길 원하는지 확인합니다.
                if (input.ToLower() == "exit")
                {
                    // 사용자가 요청하는 경우 앱을 종료합니다.
                    break;
                }
                else
                {
                    if (input.Length > 0)
                    {
                        // 사용자가 입력한 텍스트로 메시지 활동을 만듭니다.
                        Activity userMessage = new Activity
                        {
                            From = new ChannelAccount(fromUser),
                            Text = input,
                            Type = ActivityTypes.Message
                        };

                        // 봇에 메시지 활동을 보냅니다.
                        await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                    }
                }
            }
        }


        /// <summary>
        /// 봇을 지속적으로 폴링하고 봇이 클라이언트에 보낸 메시지를 검색합니다.
        /// </summary>
        /// <param name="client">Direct Line 클라이언트입니다.</param>
        /// <param name="conversationId">대화 ID입니다.</param>
        /// <returns></returns>
        private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
        {
            string watermark = null;

            // 봇에 초당 한 번 회신을 폴링합니다.
            while (true)
            {
                // 봇에서 활동 집합을 검색합니다.
                var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                watermark = activitySet?.Watermark;

                // 봇에서 전송된 활동을 추출합니다.
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // 활동 집합의 각 활동을 분석합니다.
                foreach (Activity activity in activities)
                {
                    // 활동의 텍스트를 표시합니다.
                    Console.WriteLine(activity.Text);

                    // 첨부 파일이 있습니까?
                    if (activity.Attachments != null)
                    {
                        // 활동에서 각 첨부 파일을 추출합니다.
                        foreach (Attachment attachment in activity.Attachments)
                        {
                            switch (attachment.ContentType)
                            {
                                // 영웅 카드를 표시합니다.
                                case "application/vnd.microsoft.card.hero":
                                    RenderHeroCard(attachment);
                                    break;

                                // 브라우저에 이미지를 표시합니다.
                                case "image/png":
                                    Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                    Process.Start(attachment.ContentUrl);
                                    break;
                            }
                        }
                    }

                }

                // 1초 동안 기다린 후 봇을 다시 폴링합니다.
                await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
            }
        }


        /// <summary>
        /// 콘솔에 영웅 카드를 표시합니다.
        /// </summary>
        /// <param name="attachment">영웅 카드가 들어 있는 첨부 파일입니다.</param>
        private static void RenderHeroCard(Attachment attachment)
        {
            const int Width = 70;
            // 별표 사이에 문자열을 두는 함수입니다.
            Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

            // 영웅 카드 데이터를 추출합니다.
            var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

            // 영웅 카드를 표시합니다.
            if (heroCard != null)
            {
                Console.WriteLine("/{0}", new string('*', Width + 1));
                Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                Console.WriteLine("*{0}*", new string(' ', Width));
                Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                Console.WriteLine("{0}/", new string('*', Width + 1));
            }
        }
    }
}
```
> 참고: 이 코드는 다음 랩 섹션에서 사용할 몇 가지 항목을 포함하도록 [설명서](https://docs.microsoft.com/ko-kr/azure/bot-service/bot-builder-howto-direct-line?view=azure-bot-service-4.0&tabs=cscreatebot%2Ccsclientapp%2Ccsrunclient#create-the-console-client-app)에서 약간 수정되었습니다.

상단에 Direct Line 정보를 추가하는 것을 잊지 마십시오!

잠시 동안 이 샘플 코드를 검토하시기 바랍니다. PictureBot에 연결하고 응답을 받는 방법을 이해할 수 있는 좋은 연습입니다.

**4단계** - 앱 실행

PictureBotDL 프로젝트를 마우스 오른쪽 단추로 클릭하고 "시작 프로젝트로 설정"을 선택합니다. 그런 다음 앱을 실행하고 봇과 대화를 나눕니다.

![콘솔 앱](../images//consoleapp.png)

간단한 퀴즈 - 대화 ID를 어떻게 표시합니까? 다음 섹션에서 이것이 필요한 이유를 살펴보겠습니다.

## 랩 9.4: HTTP Get을 사용하여 메시지 검색

대화 ID가 있으므로 HTTP Get을 사용하여 사용자 및 봇 메시지를 검색할 수 있습니다. Rest 클라이언트에 익숙하고 경험이 있다면 자유롭게 원하는 도구를 사용할 수 있습니다.

이 랩에서는 Postman(웹 기반 클라이언트)을 사용하여 메시지를 검색합니다.

**1단계** - Postman 다운로드

[플랫폼의 기본 앱을 다운로드합니다](https://www.getpostman.com/apps). 간단한 계정을 만들어야 할 수도 있습니다.

**2단계**

Direct Line API를 사용하면 클라이언트가 HTTP Post 요청을 실행하여 봇에 메시지를 보낼 수 있습니다. 클라이언트는 WebSocket 스트림을 통해 또는 HTTP GET 요청을 실행하여 봇으로부터 메시지를 받을 수 있습니다. 이 랩에서는 메시지를 수신하기 위한 HTTP Get 옵션을 살펴봅니다.

GET 요청을 할 것입니다. 또한 헤더에 헤더 이름(**Authorization**) 및 헤더 값(**Bearer YourDirectLineSecret**)이 포함되어 있는지 확인해야 합니다. 아울러 요청에서 {conversationId}를 현재 대화 ID로 대체하여 콘솔 앱에서 기존 대화를 호출할 것입니다. `https://directline.botframework.com/api/conversations/{conversationId}/messages`.

Postman을 사용하면 이것을 매우 쉽게 구성할 수 있습니다.
- 드롭다운을 사용하여 요청을 "GET" 유형으로 변경합니다.
- 대화 ID로 요청 URL을 입력합니다.
- Authorizatio을 "전달자 토큰" 유형으로 변경하고 "토큰" 상자에 Direct Line 비밀 키를 입력합니다.

![전달자 토큰](../images//bearer.png)

마지막으로 보내기를 선택합니다. 결과를 검토합니다. 콘솔 앱으로 새 대화를 만들고 사진을 검색해야 합니다. Postman을 사용하여 새 요청을 만듭니다. 이미지 배열 내에 표시되는 이미지 URL을 찾을 수 있습니다.

![이미지 배열 예제](../images//imagesarray.png)

### 다음 단계

여분의 시간이 있습니까? 터미널에서 curl(다운로드 링크: https://curl.haxx.se/download.html)을 활용하여 대화를 검색할 수 있습니까(Postman과 마찬가지로)?

> 힌트: 명령을 `curl -H "Authorization:Bearer {SecretKey}" https://directline.botframework.com/api/conversations/{conversationId}/messages -XGET`과 유사할 수 있습니다.
