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

> **참고** 이 랩을 완료하기 위해 다양한 환경을 사용할 수 있습니다. 강사가 환경을 설정하고 실행하는 데 필요한 단계를 안내합니다. 이는 교실에서 로그인한 컴퓨터를 사용하는 것처럼 간단하거나 가상화된 환경을 설정하는 것처럼 복잡할 수 있습니다.  랩은 Azure에서 Azure DSVM(Data Science Virtual Machine)을 사용하여 만들어지고 테스트되었으므로 Azure 계정이 있어야 사용할 수 있습니다.

#### Azure 계정 설정:

[https://azure.microsoft.com/ko-kr/free/](https://azure.microsoft.com/ko-kr/free/)에서 Azure 무료 평가판을 활성화할 수 있습니다.

이 랩을 완료하기 위해 Azure Pass가 제공된 경우 [http://www.microsoftazurepass.com/](http://www.microsoftazurepass.com/)으로 이동하여 활성화할 수 있습니다.  활성화 프로세스를 안내하는 [https://www.microsoftazurepass.com/howto](https://www.microsoftazurepass.com/howto)의 지침을 따르십시오.  Microsoft 계정에는 Azure **무료 평가판 하나** 와 이에 연결된 Azure Pass 하나가 있을 수 있으므로, Microsoft 계정에서 Azure Pass를 이미 활성화한 경우 무료 평가판을 사용하거나 다른 Microsoft 계정을 사용해야 합니다.

### 환경 설정

이러한 랩은 [Visual Studio 2019, Community Edition](https://www.visualstudio.com/downloads/)을 사용하여 .NET Framework와 함께 사용하도록 만들어졌습니다.  원래 워크샵은 Azure DSVM(Data Science Virtual Machine)을 사용하도록 디자인되고 테스트되었습니다.  프리미엄 Azure 구독만 실제로 Azure에서 DSVM 리소스를 만들 수 있지만 Visual Studio 2019 Community Edition을 실행하는 로컬 컴퓨터와 랩 단계 전반에 걸쳐 나열된 필수 다운로드 소프트웨어를 사용하여 랩을 완료할 수 있습니다.

### 필요한 URL 및 키

이 랩에서는 다양한 Cognitive Services 키와 저장소 키를 수집합니다. 이후 랩에서 쉽게 액세스할 수 있도록 모든 키를 텍스트 파일에 저장하는 것이 좋습니다.

> _키_
>
>- Cognitive Services API Url:
>- Cognitive Services API 키:
>- LUIS API 엔드포인트:
>- LUIS API 키:
>- LUIS API 앱 ID:
>- 봇 앱 이름:
>- 봇 앱 ID:
>- 봇 앱 암호:
>- Azure Storage 연결 문자열:
>- Cosmos DB Url:
>- Cosmos DB 키:
>- DirectLine 키:

> **참고:** 이러한 항목 모두가 이 랩에서 채워지는 것은 아닙니다.

### Azure 설정

다음 단계에서는 이후의 랩을 위해 Azure 환경을 구성합니다.

####    Cognitive Services

첫 번째 랩은 [Computer Vision](https://www.microsoft.com/cognitive-services/ko-kr/computer-vision-api) Cognitive Services에 중점을 두고 있지만, Microsoft Azure를 사용하면 모든 서비스에 걸쳐 있는 Cognitive Service 계정을 만들거나 개별 서비스에 대한 Cognitive Services 계정을 만들 수 있습니다.  다음 단계에서는 모든 Cognitive Services에 걸쳐 있는 계정을 만듭니다.

1.  [Azure Portal](https://portal.azure.com)을 엽니다.

1.  **+ 리소스 만들기** 를 클릭한 다음 검색 상자에 **cognitive services** 를 입력합니다. 

1.  사용 가능한 옵션에서 **Cognitive Services** 를 선택한 다음 **만들기** 를 클릭합니다.

> **참고** 특정 Cognitive Services 리소스를 만들거나 모든 엔드포인트를 포함하는 단일 리소스를 만들 수 있습니다.

1.  자신이 선택한 이름을 입력합니다.

1.  구독 및 리소스 그룹을 선택합니다.

1.  가격 책정 계층의 경우 **S0** 을 선택합니다.

1.  확인 확인란을 선택합니다.

1.  **만들기** 를 클릭합니다.

1.  새 리소스로 이동하여 **빠른 시작** 을 클릭합니다.

1.  **URL** 및 **엔드포인트** 를 메모장에 복사합니다.

####    Azure Storage 계정

1.  Azure Portal에서 **+ 리소스 만들기** 를 클릭한 다음 검색 상자에 **스토리지** 를 입력합니다. 

1.  사용 가능한 옵션에서 **Storage 계정** 을 선택한 다음 **만들기** 를 클릭합니다.

1.  구독 및 리소스 그룹을 선택합니다.

1.  계정에 고유한 이름을 입력합니다.

1.  위치의 경우, 리소스 그룹과 동일한 위치를 선택합니다.

1.  성능은 **표준** 이어야 합니다.

1.  계정 종류는 **StorageV2(범용 v2)** 여야 합니다.

1.  복제의 경우 **LRS(로컬 중복 스토리지)** 를 선택합니다.

1.  **검토 + 만들기** 를 클릭합니다.

1.  **만들기** 를 클릭합니다.

1.  새 리소스로 이동하여 **액세스 키** 를 클릭합니다.

1.  메모장에 **연결 문자열** 을 복사합니다.

1.  **개요** 를 선택한 다음 **컨테이너** 를 클릭합니다.

1.  **+컨테이너** 를 선택합니다.

1.  이름에 **images** 를 입력합니다.

1.  **확인** 을 선택합니다.

####    Cosmos DB

1.  [Azure Portal](https://portal.azure.com)을 엽니다.

1.  **"+ 리소스 만들기"를** 클릭한 다음 검색 상자에 **cosmos** 를 입력합니다. 

1.  사용 가능한 옵션에서 **Azure Cosmos DB** 를 선택합니다.  

1.  **만들기** 를 클릭합니다.

1.  구독 및 리소스 그룹을 선택합니다.

1.  고유한 계정 이름을 입력하고 리소스 그룹과 일치하는 위치를 선택합니다.

1.  **검토 + 만들기** 를 클릭합니다.

1.  **만들기** 를 클릭합니다.

1.  새 리소스로 이동하고 **설정** 에서 **키** 를 선택합니다.

1.  **URI** 및 **기본 키** 를 메모장에 복사합니다.

### Bot Builder SDK

C#에 대한 Bot Builder 템플릿을 사용하여 이 과정에서 봇을 만듭니다.

#### Bot Builder SDK 다운로드

1.  [여기에서 C#용 Bot Builder SDK v4 템플릿 다운로드](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)로 브라우저 창을 엽니다. 

1.  **다운로드** 를 클릭합니다.

1.  다운로드 폴더 위치로 이동한 후 설치를 두 번 클릭합니다.

1.  모든 버전의 Visual Studio를 선택하고 **설치** 를 클릭합니다.  메시지가 표시되면 **작업 종료** 를 클릭합니다.  

1.  **닫기** 를 클릭합니다. 이제 Visual Studio 템플릿에 봇 템플릿이 추가되어 있을 것입니다. 

### Bot Emulator

최신 .NET SDK(v4)를 사용하여 봇을 개발하려고 합니다.  로컬 테스트를 수행하려면 Bot Framework Emulator를 다운로드해야 합니다.

### Bot Framework Emulator 다운로드

로컬에서 봇을 테스트하도록 v4 Bot Framework Emulator를 다운로드할 수 있습니다. 나머지 랩에 대한 지침에서는 v4 에뮬레이터를 다운로드한 것으로 가정합니다. 

1.  [이 페이지](https://github.com/Microsoft/BotFramework-Emulator/releases)로 이동하여 태그 "4.5.1" 이상이 있는 에뮬레이터의 최신 버전을 다운로드하는 방법으로 에뮬레이터를 다운로드합니다(Windows를 사용하는 경우 "*-windows-setup.exe" 파일 선택).

> **참고** 에뮬레이터 설치 위치는 
`"C:\Users\_your-username\AppData\Local\Programs\@bfemulatormain\Bot Framework Emulator.exe"`이지만, 시작 메뉴를 통해 **bot framework** 를 검색하여 이 위치에 액세스할 수 있습니다.

## 크레딧

이 시리즈의 랩은 [Cognitive Services 자습서](https://github.com/noodlefrenzy/CognitiveServicesTutorial) 및 [AI 부트캠프 학습](https://github.com/Azure/LearnAI-Bootcamp)에서 채택되었습니다.

##  리소스

여기에 설명된 아키텍처에 대한 이해를 높이고 AI 솔루션 개발에 광범위한 팀을 참여시키기 위해 다음 리소스를 검토하는 것이 좋습니다.

- [Cognitive Services](https://www.microsoft.com/cognitive-services) - AI 엔지니어
- [Cosmos DB](https://docs.microsoft.com/ko-kr/azure/cosmos-db/) - 데이터 엔지니어
- [Azure Search](https://azure.microsoft.com/ko-kr/services/search/) - 검색 엔지니어
- [봇 개발자 포털](http://dev.botframework.com) - AI 엔지니어

## 다음 단계

-   [랩 02-01: Computer Vision 구현](../Lab2-Implement_Computer_Vision/01-Introduction.md)
