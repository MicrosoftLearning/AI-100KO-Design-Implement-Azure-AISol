---
lab:
    title: '랩 5: QnA Maker와 봇 통합'
    module: '모듈 3: QnA Maker로 봇 개선'
---

# 랩 5: 사용자 지정 QnA Maker 봇 만들기

##  소개

이 랩에서는 사전 학습된 기술 자료에 연결하는 봇을 만들기 위한 QnA Maker를 살펴보겠습니다.  QnAMaker를 사용하면 문서를 업로드하거나 웹 페이지를 가리키고 기술 자료가 미리 채워지도록 할 수 있으며, 기술 자료는 자주 묻는 질문과 같은 일반적인 용도로 간단한 봇에 공급될 수 있습니다.

## 랩 5.1: QnA Maker 설정

1.  [Azure Portal](https://portal.azure.com)을 엽니다.

1.  **새 리소스 추가** 를 클릭합니다.

1.  **QnA Maker** 를 입력하고 **QnA Maker** 를 선택합니다.

1.  **만들기** 를 클릭합니다.

1.  이름을 입력합니다.

1.  리소스 가격 책정 계층의 경우 **S0** 계층을 선택합니다.  1MB 보다 큰 파일을 나중에 업로드하기 때문에 무료 계층을 사용하지 않습니다.

1.  리소스 그룹을 선택합니다.

1.  검색 가격 책정 계층의 경우 **F** 계층을 선택합니다.

1.  앱 이름을 입력합니다. 앱 이름은 고유해야 합니다.

1.  **만들기** 를 클릭합니다.  이렇게 하면 리소스 그룹에 다음과 같은 리소스가 만들어집니다.

-   App Service 계획
-   App Service
-   Application Insights
-   Search Service
-   QnAMaker 유형의 Cognitive Service 인스턴스

## 랩 5.2: 기술 자료 만들기

1.  [QnA Maker 사이트](https://qnamaker.ai) 를 엽니다.

1.  오른쪽 상단에서 **로그인** 을 클릭합니다. 새 QnA Maker 리소스에 대한 Azure 자격 증명을 사용하여 로그인합니다.

1.  위쪽 탐색 영역에서 **기술 자료 만들기** 를 클릭합니다.

1.  이미 리소스를 만들었으므로 **1단계** 를 건너뜁니다.

1.  QnA Maker 리소스에 연결된 Azure AD 및 구독을 선택한 다음 새로 만든 QnA Maker 리소스를 선택합니다.

1.  이름에 **Microsoft FAQs*** 를 입력합니다.

1.  파일의 경우 **파일 추가** 를 클릭하고 **code/surface-pro-4-user-guide-EN.pdf** 파일을 찾아봅니다.

1.  파일의 경우 **파일 추가** 를 클릭하고 **code/Manage Azure Blob Storage** 파일을 찾아봅니다.

> **참고** 지원되는 파일 유형 및 데이터 원본에 대한 자세한 내용은 [여기](https://docs.microsoft.com/ko-kr/azure/cognitive-services/qnamaker/concepts/data-sources-supported) 에서 확인할 수 있습니다.

1.  **Chit-chat** 의 경우 **위트 있음** 을 선택합니다.

1.  **KB 만들기** 를 클릭합니다.

## 랩 5.3: 기술 자료 게시 및 테스트

1.  기술 자료 QnA 쌍을 검토합니다. 기술 자료를 공급한 두 문서에 따라 200개 정도의 다른 쌍을 볼 수 있습니다.

1.  상단 메뉴에서 **게시** 를 클릭합니다.  

1.  게시 페이지에서 **게시** 를 클릭합니다.  서비스가 게시되면 성공 페이지에서 **봇 만들기** 단추를 클릭합니다.

1.  메시지가 표시되면 랩 리소스 그룹에 연결된 계정으로 로그인합니다.

1.  봇 서비스 만들기 페이지에서 이름 오류를 수정한 다음 **만들기** 를 클릭합니다.

> **참고** Azure의 최근 변경 사항으로 인해 일부 리소스 이름에서 대시("-")를 제거해야 합니다.

1.  봇 리소스가 만들어지면 새 **웹앱 봇** 으로 이동한 다음 **웹 채팅에서 테스트** 를 클릭합니다.

1.  Surface Pro 4 또는 Azure Blob Storage 관리와 관련하여 다음과 같이 질문합니다.

+ 메모리를 추가하려면 어떻게 해야 합니까?
+ 배터리 충전에 걸리는 시간은 얼마입니까?
+ 하드 리셋은 어떻게 해야 합니까?
+ Blob이란 무엇입니까?

1.  봇이 알지 못하는 사항을 몇 가지 질문합니다.

+ 스트라이크를 치려면 어떻게 해야 합니까?

## 랩 5.4: 봇 소스 코드 다운로드

1.  **봇 관리** 에서 **빌드** 탭을 선택합니다.

1.  **봇 소스 코드 다운로드** 를 클릭하고, 메시지가 표시되면 **예** 를 클릭합니다.  

1.  Azure에서 소스 코드를 빌드합니다. 완료되면 **봇 소스 코드 다운로드** 를 클릭하고, 메시지가 표시되면 **예** 를 선택합니다.

1.  로컬 컴퓨터로 zip 파일을 추출합니다.

1.  솔루션 파일을 엽니다. 그러면 Visual Studio가 열립니다.

1.  **Startup.cs** 파일을 엽니다. 여기에 특별한 것이 추가되지 않은 것을 알 수 있습니다.

1.  **Bots/{BotName}.cs** 파일을 엽니다. QnA Maker가 초기화되는 위치는 다음과 같습니다.

```csharp
var qnaMaker = new QnAMaker(new QnAMakerEndpoint
{
    KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
    EndpointKey = _configuration["QnAAuthKey"],
    Host = GetHostname()
},
null,
httpClient);
```

그런 다음, 호출됨:

```csharp
var response = await qnaMaker.GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(MessageFactory.Text(response[0].Answer), cancellationToken);
}
else
{
    await turnContext.SendActivityAsync(MessageFactory.Text("No QnA Maker answers were found."), cancellationToken);
}
```

위에서 볼 수 있듯이,코드 몇 줄로 매우 간단하게 자신의 봇에 생성 된 QnA Maker를 추가할 수 있습니다.

## 다음 단계

QnAMaker 코드를 사진 봇에 통합하려면 어떻게 해야 합니까?

##  리소스

-   [QnA Maker 서비스는 무엇입니까?](https://docs.microsoft.com/ko-kr/azure/cognitive-services/qnamaker/overview/overview)

##  다음 단계

-   [랩 06-01: LUIS 구현](../Lab6-Implement_LUIS/01-Introduction.md)