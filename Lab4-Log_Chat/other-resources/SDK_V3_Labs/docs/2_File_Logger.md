# 파일 로거

## 1.	목표

이 랩의 목적은 미들웨어를 사용하여 파일에 채팅 대화를 로깅하는 것입니다. 과도한 I/O(입출력) 작업으로 인해 봇의 응답이 느려질 수 있습니다. 이 랩에서는 전역 이벤트를 활용하여 파일에 대한 채팅 대화를 효율적으로 로깅합니다. 이 활동은 이전 랩에서 다룬 개념의 연장선입니다.

Visual Studio에서 code\file-core-Middleware의 프로젝트를 가져옵니다.

## 2. 비효율적인 로깅 방법

채팅 대화를 파일에 로깅하는 매우 간단한 방법은 아래 코드 조각에 나온 대로 File.AppendAllLines를 사용하는 것입니다. File.AppendAllLines는 파일을 열고 지정된 문자열을 파일에 추가한 다음 파일을 닫습니다. 그러나 사용자/봇의 모든 메시지에 대해 파일을 열고 닫게 되므로 이 방법은 매우 비효율적일 수 있습니다.
tu
````C#
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        File.AppendAllLines("C:\\Users\\username\\log.txt", new[] { $"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity().Text}" });
    }
}
````

## 3.	효율적인 로깅 방법

파일에 로깅하는 보다 효율적인 방법은 Global.asax 파일을 사용하고 봇의 수명 주기와 관련된 이벤트를 사용하는 것입니다. 아래 코드 조각에 표시된 대로 global.asax를 확장합니다. 아래 줄을 해당 환경의 경로로 변경합니다.

````C# 
tw = new StreamWriter("C:\\Users\\username\\log.txt", true);
````

Application_Start는 특정 인코딩의 스트림에 문자를 쓰는 스트림 래퍼를 구현하는 StreamWriter를 엽니다. 이는 봇의 수명 동안 지속되므로 이제 각 메시지에 대한 파일을 열고 닫는 것이 아니라 메시지를 기록하기만 하면 됩니다. 또한 StreamWriter가 이제 DebugActivityLogger에 매개 변수로 전송된다는 점도 주목할 만합니다. 응용 프로그램이 언로드되기 전에 응용 프로그램 수명 중 한 번 호출되는 Application_End를 통해 로그 파일을 닫을 수 있습니다.

````C#
using System.Web.Http;
using Autofac;
using Microsoft.Bot.Builder.Dialogs;
using System.IO;
using System.Diagnostics;
using System;


namespace MiddlewareBot
{
    public class WebApiApplication : System.Web.HttpApplication
    {
        static TextWriter tw = null;
        protected void Application_Start()
        {
            tw = new StreamWriter("C:\\Users\\username\\log.txt", true);
            Conversation.UpdateContainer(builder =>
            {
                builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency().WithParameter("inputFile", tw);
            });

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }

        protected void Application_End()
        {
            tw.Close();
        }
    }
}
````

아래 줄을 추가하여 DebugActivityLogger 생성자에서 파일 매개 변수를 받고 로그 파일에 기록하도록 LogAsync를 업데이트합니다.

````C#
tw.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity().Text}", true);
tw.Flush();
````

전체 DebugActivityLogger 코드는 다음과 같습니다.

````C#
using System.Diagnostics;
using System.Threading.Tasks;
using Microsoft.Bot.Builder.History;
using Microsoft.Bot.Connector;
using System.IO;

namespace MiddlewareBot
{    
    public class DebugActivityLogger : IActivityLogger
    {
        TextWriter tw;
        public DebugActivityLogger(TextWriter inputFile)
        {
            this.tw = inputFile;
        }


        public async Task LogAsync(IActivity activity)
        {
            tw.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity().Text}", true);
            tw.Flush();
        }
    }

}
````

## 4. 로그 파일

봇 응용 프로그램을 실행하고 에뮬레이터에서 메시지를 사용하여 테스트합니다. 줄에 지정된 로그 파일을 검토합니다.

````C# 
tw = new StreamWriter("C:\\Users\\username\\log.txt", true);
````

사용자와 봇의 로그 메시지를 보려는 경우 이제 로그 파일에 다음과 같이 메시지가 표시됩니다.

````From:2c1c7fa3 - To:56800324 - Message:a log message````

````From:56800324 - To:2c1c7fa3 - Message:You sent a log message which was 13 characters````

## 5. 선택적 로깅

특정 시나리오에서는 선택적 메시지에 대한 모델링을 수행하거나 선택적으로 로깅하는 것이 바람직합니다. 이러한 경우는 많이 있으며 예를 들면 다음과 같습니다. a) 매우 활동적인 커뮤니티 봇은 무수한 채팅 메시지를 매우 빠르게 캡처할 수 있으며 많은 채팅 메시지는 별로 유용하지 않을 수 있습니다. b) 특정 사용자/봇 또는 특정 인기 제품 카테고리/토픽과 관련된 메시지에 초점을 맞추기 위해 선택적으로 로깅해야 할 수도 있습니다.

모든 채팅 메시지를 캡처하고 필터링을 통해 선택적 메시지를 마이닝할 수 있지만, 유연하게 선택적으로 로깅하는 기능이 유용할 수 있으며 Microsoft Bot Framework는 이를 지원합니다. 사용자의 메시지를 선택적으로 로깅하려면 활동 json을 검토하고 "name"을 기준으로 필터링할 수 있습니다. 예를 들어 아래 json에서는 "Bot1"에서 메시지를 보냈습니다.

```json
{
  "type": "message",
  "timestamp": "2017-10-27T11:19:15.2025869Z",
  "localTimestamp": "2017-10-27T07:19:15.2140891-04:00",
  "serviceUrl": "http://localhost:9000/",
  "channelId": "emulator",
  "from": {
    "id": "56800324",
    "name": "Bot1"
  },
  "conversation": {
    "isGroup": false,
    "id": "8a684db8",
    "name": "Conv1"
  },
  "recipient": {
    "id": "2c1c7fa3",
    "name": "User1"
  },
  "membersAdded": [],
  "membersRemoved": [],
  "text": "You sent hi which was 2 characters",
  "attachments": [],
  "entities": [],
  "replyToId": "09df56eecd28457b87bba3e67f173b84"
}
```
json을 검토한 후 ````activity.From.Name```` 속성을 확인하는 간단한 조건을 도입함으로써 사용자 메시지를 기준으로 필터링하도록 DebugActivityLogger의 LogAsync 메서드를 수정할 수 있습니다.

````C#
public class DebugActivityLogger : IActivityLogger
{
        public async Task LogAsync(IActivity activity)
        {
           if (!activity.From.Name.Contains("Bot"))
            {
               Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
            }
        }
}
````


### [3_SQL_Logger](3_SQL_Logger.md)로 계속 진행

[0_README](../0_README.md)로 돌아가기