---
lab:
    title: '랩 1: 기술 요구 사항 충족'
    module: '모듈 1: Azure Cognitive Services 소개'
---

# 랩 1: 기술 요구 사항 충족

## 랩 1.1: 기술, 환경 및 키 설정

이 랩은 AI(인공 지능) 엔지니어 또는 Azure의 AI 개발자를 위한 것입니다. 연습을 수행할 시간을 가질 수 있도록 이 과정의 랩을 시작하기 전에 충족해야 할 특정 요구 사항이 있습니다.

Visual Studio를 사용해본 경험이 있는 것이 좋습니다. 랩에서 빌드하는 모든 것에 Visual Studio가 사용되므로 응용 프로그램을 만들기 위한 [사용 방법](https://docs.microsoft.com/ko-kr/visualstudio/ide/visual-studio-ide)을 숙지하고 있어야 합니다. 또한 이 랩은 코딩 또는 개발 방법을 가르치는 수업이 아니며, C#(중급 - [여기](https://mva.microsoft.com/ko-kr/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949) 및 [여기](https://docs.microsoft.com/ko-kr/dotnet/csharp/quick-starts/)에서 배울 수 있음) 사용 방법을 어느 정도 알고 있지만 Cognitive Services로 솔루션을 구현하는 방법은 알지 못한다고 가정합니다.

### 계정 설정

> 참고: 이 랩을 완료하기 위해 다양한 환경을 사용할 수 있습니다.  강사가 환경을 설정하고 실행하는 데 필요한 단계를 안내합니다.   이는 교실에서 로그인한 컴퓨터를 사용하는 것처럼 간단하거나 가상화된 환경을 설정하는 것처럼 복잡할 수 있습니다.  랩은 Azure에서 Azure DSVM(Data Science Virtual Machine)을 사용하여 만들어지고 테스트되었으므로 Azure 계정이 있어야 사용할 수 있습니다.

#### Azure 계정 설정:

[https://azure.microsoft.com/ko-kr/free/](https://azure.microsoft.com/ko-kr/free/)에서 Azure 무료 평가판을 활성화할 수 있습니다.

이 랩을 완료하기 위해 Azure Pass가 제공된 경우 [http://www.microsoftazurepass.com/](http://www.microsoftazurepass.com/)으로 이동하여 활성화할 수 있습니다.  활성화 프로세스를 안내하는 [https://www.microsoftazurepass.com/howto](https://www.microsoftazurepass.com/howto)의 지침을 따르십시오.  Microsoft 계정에는 Azure **무료 평가판 하나** 와 이에 연결된 Azure Pass 하나가 있을 수 있으므로, Microsoft 계정에서 Azure Pass를 이미 활성화한 경우 무료 평가판을 사용하거나 다른 Microsoft 계정을 사용해야 합니다.

### 환경 설정

이러한 랩은 [Visual Studio 2017, Community Edition](https://www.visualstudio.com/downloads/)을 사용하여 .NET Framework와 함께 사용하도록 만들어졌습니다.  원래 워크샵은 Azure DSVM(Data Science Virtual Machine)을 사용하도록 디자인되고 테스트되었습니다.  프리미엄 Azure 구독만 실제로 Azure에서 DSVM 리소스를 만들 수 있지만 Visual Studio 2017 Community Edition을 실행하는 로컬 컴퓨터와 랩 단계 전반에 걸쳐 나열된 필수 다운로드 소프트웨어를 사용하여 랩을 완료할 수 있습니다.

### 키 설정

이 랩에서는 다양한 Cognitive Services 키와 저장소 키를 수집합니다. 이후 랩에서 쉽게 액세스할 수 있도록 모든 키를 텍스트 파일에 저장하는 것이 좋습니다.

> 키_
>
>- Cognitive Services 교육 키:
>- Cognitive Services API 키:
>- LUIS API 끝점, 키, 앱 ID:
>- 봇 앱 이름:
>- 봇 앱 ID:
>- 봇 앱 암호:


#### 학습 키 받기:

교육 API 키를 사용하면 프로그래밍 방식으로 Custom Vision 프로젝트를 만들고 관리하며 학습시킬 수 있습니다. 학습 키는 이를 지원하는 AI 서비스에서 찾을 수 있습니다.  그렇지 않은 경우 생성한 각 서비스에 대해 생성된 키를 사용할 수 있습니다.   랩에서 해당 서비스의 주요 위치를 설명합니다.

> 참고: Internet Explorer는 지원되지 않습니다. Edge, Firefox 또는 Chrome을 사용하는 것이 좋습니다.

#### Cognitive Services API 키 얻기:

주로 [Computer Vision](https://www.microsoft.com/cognitive-services/ko-kr/computer-vision-api) Cognitive Services를 사용하므로 이 서비스를 위한 API 키를 만들겠습니다.

포털에서 **"+ 새로 만들기"** 단추(마우스로 가리키면 **리소스 만들기** 가 표시됨)를 클릭한 다음 검색 상자에 **computer vision** 을 입력하고 **Computer Vision API** 를 선택합니다.

![Cognitive Services 키 만들기](../../Linked_Image_Files/new-cognitive-services.PNG)

다음으로, 만들 API 끝점에 대한 몇 가지 세부 정보를 작성하고 원하는 API와 끝점을 사용할 위치(**미국 서부 지역을 선택해야만 작동!!**) 그리고 요금제를 선택합니다. 자습서에 필요한 처리량을 위해 **S1** 을 사용합니다. 쉽게 찾을 수 있도록 _대시보드에 고정_합니다. Computer Vision API는 향후 Cognitive Services Vision 제품을 개선하기 위해 이미지를 Microsoft에 내부적으로 안전하게 저장하므로 리소스를 만들기 전에 이에 대한 확인란을 선택해야 합니다.

**Computer Vision 서비스 위치를 미국 서부로 지정했는지 다시 한 번 확인합니다.**

> 다음 랩의 코드는 Computer Vision API를 호출하기 위해 미국 서부를 사용하도록 설정되어 있습니다. 이후 작업 시 다른 지역을 호출하는 방법을 [여기](https://docs.microsoft.com/ko-kr/azure/cognitive-services/Computer-vision/Vision-API-How-to-Topics/HowToSubscribe)에서 확인할 수 있습니다.


### Bot Builder SDK

C#에 대한 Bot Builder 템플릿을 사용하여 이 과정에서 봇을 만듭니다.

#### Bot Builder SDK 다운로드

[C# 언어 버전의 Bot Builder SDK v4 템플릿을 여기서](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) 다운로드하고 "다른 이름으로 저장"을 클릭하여 Visual C#용 Visual Studio ItemTemplates 폴더에 저장합니다. 이는 일반적으로 `C:\Users\`_your-username_`\Documents\Visual Studio 2017\Templates\ItemTemplates\Visual C#`에 있습니다. 폴더 위치로 이동한 다음 설치를 두 번 클릭하고 "설치"를 클릭하여 Visual Studio 템플릿에 템플릿을 추가합니다. 브라우저에 따라 템플릿을 다운로드할 때 템플릿을 두 번 클릭하여 Visual Studio Community 2017에 직접 설치해도 됩니다.

### Bot Emulator

최신 .NET SDK(v4)를 사용하여 봇을 개발하려고 합니다.  시작하려면 Bot Framework Emulator를 다운로드해야 하며 웹앱 봇을 만들고 소스 코드를 얻어야 합니다.

##### Bot Framework Emulator 다운로드

로컬에서 봇을 테스트하도록 v4 Preview Bot Framework Emulator를 다운로드할 수 있습니다. 나머지 랩에 대한 지침은 v4 에뮬레이터(v3 에뮬레이터가 아님)를 다운로드한 것으로 가정합니다. [이 페이지](https://github.com/Microsoft/BotFramework-Emulator/releases)로 이동하여 태그 "4.1.0"이 있는 에뮬레이터의 최신 버전을 다운로드하는 방법으로 에뮬레이터를 다운로드합니다(Windows를 사용하는 경우 ".exe"파일 선택).

이 에뮬레이터는 브라우저에 따라 `c:\Users\`_your-username_`\AppData\Local\botframework\app-`_version_`\botframework-emulator.exe` 또는 Downloads 폴더에 설치됩니다.

## 랩 1.2: 아키텍처 개요

로컬 드라이브에서 사진을 수집한 다음 [Computer Vision API](https://www.microsoft.com/cognitive-services/ko-kr/computer-vision-api)를 호출하여 이미지를 분석하고 태그 및 설명을 얻을 수 있는 단순한 C# 응용 프로그램을 빌드합니다.

랩 전반에 걸쳐 이 랩에서 계속하여 고객의 텍스트 문의와 상호 작용하는 [Bot Framework](https://dev.botframework.com/) 봇을 빌드하는 방법을 보여 드리겠습니다. 그런 다음 [QnA Maker](https://docs.microsoft.com/ko-kr/azure/cognitive-services/qnamaker/overview/overview)를 통해 기존 기술 자료 및 FAQ를 Bot Framework에 통합하기 위한 간단한 솔루션을 시연합니다. 마지막으로, 쿼리에서 의도를 자동으로 도출하고 이를 바탕으로 고객의 텍스트 요청에 지능적으로 응답하도록 [LUIS](https://www.microsoft.com/cognitive-services/ko-kr/language-understanding-intelligent-service-luis)를 사용하여 이 봇을 확장하겠습니다.

고객이 봇과 상호 작용하는 동안 다른 데이터에 액세스할 수 있도록 Bing Search를 사용하기 위한 컨텍스트만 제공하며 이러한 시나리오를 랩에서 구현하지는 않습니다. 참가자는 [Bing Web Search](https://azure.microsoft.com/ko-kr/services/cognitive-services/directory/search/) 서비스에 대한 자세한 내용을 확인할 수 있습니다.

이 랩의 범위를 벗어나지만 이 아키텍처는 Azure의 데이터 솔루션을 통합하고, [Blob Storage]((https://docs.microsoft.com/ko-kr/azure/storage/storage-dotnet-how-to-use-blobs)와 [Cosmos DB](https://azure.microsoft.com/ko-kr/services/cosmos-db/)를 통해 이 아키텍처의 이미지 및 메타데이터 저장을 관리합니다.

![아키텍처 다이어그램](../../Linked_Image_Files/AI_Immersion_Arch.png)

## 랩 1.3: 향후 프로젝트/학습을 위한 리소스

여기에 설명된 아키텍처에 대한 이해를 높이고 AI 솔루션 개발에 광범위한 팀을 참여시키기 위해 다음 리소스를 검토하는 것이 좋습니다.

- [Cognitive Services](https://www.microsoft.com/cognitive-services) - AI 엔지니어
- [Cosmos DB](https://docs.microsoft.com/ko-kr/azure/cosmos-db/) - 데이터 엔지니어
- [Azure Search](https://azure.microsoft.com/ko-kr/services/search/) - 검색 엔지니어
- [봇 개발자 포털](http://dev.botframework.com) - AI 엔지니어


## 크레딧

이 시리즈의 랩은 [Cognitive Services 자습서](https://github.com/noodlefrenzy/CognitiveServicesTutorial) 및 [AI 부트캠프 학습](https://github.com/Azure/LearnAI-Bootcamp)에서 채택되었습니다.
