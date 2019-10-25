# 미들웨어 봇 샘플

메시지를 가로채서 로깅하는 방법을 보여주는 샘플 봇입니다.

[![Deploy to Azure][Deploy Button]][Deploy CSharp/MiddlewareLogging]

[Deploy Button]: https://azuredeploy.net/deploybutton.png
[Deploy CSharp/MiddlewareLogging]: https://azuredeploy.net

### 사전 요건

이 샘플을 실행하기 위한 최소 필수 구성 요소는 다음과 같습니다.
* Visual Studio 2015의 최신 업데이트. [여기](http://www.visualstudio.com)에서 커뮤니티 버전을 무료로 다운로드할 수 있습니다.
* Bot Framework Emulator. Bot Framework Emulator를 설치하려면 [여기](https://emulator.botframework.com/)에서 다운로드합니다. Bot Framework Emulator에 대한 자세한 내용은 [이 설명서 항목](https://github.com/microsoft/botframework-emulator/wiki/Getting-Started)을 참조하십시오.

### 코드 주요 내용

대화 기록을 사용할 때 가장 일반적인 작업 중 하나는 봇과 사용자 간의 메시지 활동을 가로채고 로깅하는 것입니다. [`Microsoft.Bot.Builder.History`](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Library/Microsoft.Bot.Builder.History) 네임스페이스는 이를 위한 인터페이스와 클래스를 제공합니다. 특히 [`IActivityLogger`](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Library/Microsoft.Bot.Builder/Dialogs/IActivityLogger.cs) 인터페이스에는 클래스가 메시지 활동을 로깅하기 위해 구현해야 하는 기능에 대한 정의가 포함되어 있습니다.

디버그 모드로 실행할 때만 추적 수신기에 메시지 활동을 기록하는 `IActivityLogger` 인터페이스의 [`DebugActivityLogger`](DebugActivityLogger.cs) 구현을 살펴보십시오.

````C#
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
    }
}
````

기본적으로 BotBuilder 라이브러리는 no-op 활동 로거인 대화 컨테이너에 [`NullActivityLogger`](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Library/Microsoft.Bot.Builder/Dialogs/IActivityLogger.cs#L81)를 등록합니다. [`Global.asax.cs`](Global.asax.cs#L11-L13)에서 샘플의 [`DebugActivityLogger`](DebugActivityLogger.cs)를 Autofac 컨테이너에 등록하는 것을 살펴보십시오.

````C#
var builder = new ContainerBuilder();
builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency();
builder.Update(Conversation.Container);
````

### 결과

샘플 솔루션을 열고 실행할 때 [Visual Studio 출력 창의 디버그 텍스트 창](https://blogs.msdn.microsoft.com/visualstudioalm/2015/02/09/the-output-window-while-debugging-with-visual-studio/)에 다음 결과가 표시됩니다.

봇이 받은 메시지:
````
From:default-user - To:ii845fc9l02209hh6 - Message:Hello bot!
````

사용자에게 보낸 메시지:
````
From:ii845fc9l02209hh6 - To:default-user - Message:You sent Hello bot! which was 10 characters
````

### 자세한 정보

Bot Builder for .NET 및 대화 기록을 시작하는 방법에 대한 자세한 내용은 다음 리소스를 검토하십시오.

* [Bot Builder for .NET](https://docs.microsoft.com/ko-kr/bot-framework/dotnet/)
* [메시지 가로채기](https://docs.microsoft.com/ko-kr/bot-framework/dotnet/bot-builder-dotnet-middleware)
* [Microsoft.Bot.Builder.History 네임스페이스](https://docs.botframework.com/ko-kr/csharp/builder/sdkreference/dc/dc6/namespace_microsoft_1_1_bot_1_1_builder_1_1_history.html)
* [TableLogger(Azure Table Storage를 사용하는 활동 로거)](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Library/Microsoft.Bot.Builder.Azure/TableLogger.cs#L60)