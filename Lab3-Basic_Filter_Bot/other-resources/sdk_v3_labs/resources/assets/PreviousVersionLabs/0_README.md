# LUIS 및 Azure Search를 사용하여 지능형 응용 프로그램 개발

이 실습 랩에서는 Microsoft Bot Framework, Azure Search 및 Microsoft LUIS(Language Understanding Intelligent Service)를 사용하여 종단 간 지능형 봇을 만드는 방법을 안내합니다.


## 목표
이 워크샵에서는 다음과 같은 내용을 다룹니다.
- 응용 프로그램 내에서 긍정적인 검색 경험을 제공하도록 Azure Search 기능을 구현하는 방법 이해
- 데이터를 확장하여 전체 텍스트, 언어 인식 검색을 사용하도록 Azure Search 서비스 구성
- 봇이 효과적으로 통신할 수 있도록 LUIS 모델 빌드, 학습 및 게시
- LUIS 및 Azure Search를 활용하는 Microsoft Bot Framework를 사용하여 지능형 봇 빌드


LUIS 및 Azure Search에 중점을 두고 있지만 다음 기술도 활용합니다.

- DSVM(Data Science Virtual Machine)
- Windows 10 SDK(UWP)
- CosmosDB
- Azure Storage
- Visual Studio


## 사전 요건

이 워크샵은 Azure의 AI 개발자를 위한 것입니다. 짧은 워크샵이므로 시작 전에 충족되어야 하는 사항들이 있습니다.

첫째, Visual Studio를 사용한 경험이 있어야 합니다. 워크샵에서 빌드하는 모든 것에 Visual Studio가 사용되므로 응용 프로그램을 만들기 위한 [사용 방법](https://docs.microsoft.com/ko-kr/visualstudio/ide/visual-studio-ide)을 숙지하고 있어야 합니다. 또한 이 워크샵은 응용 프로그램 코딩 또는 개발 방법을 가르치는 수업이 아니며, C#에서 코딩하는 방법([여기](https://mva.microsoft.com/ko-kr/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949)에서 학습할 수 있음)은 알고 있지만 고급 검색 및 NLP(자연어 처리) 솔루션을 구현하는 방법은 알지 못한다고 가정합니다.

둘째, Microsoft의 Bot Framework를 사용하여 봇을 개발한 경험이 있어야 합니다. 봇 디자인 방법이나 대화 상자 작동 방식에 대해 설명하는 데는 많은 시간을 할애하지 않습니다. Bot Framework에 익숙하지 않은 경우 워크샵에 참가하기 전에 [이 Microsoft Virtual Academy 과정](https://mva.microsoft.com/ko-kr/training-courses/creating-bots-in-the-microsoft-bot-framework-using-c-17590#!)을 수강해야 합니다.

셋째, 포털을 사용한 경험이 있으며 Azure에서 리소스를 만들고 비용을 지출할 수 있어야 합니다. 이 워크샵에서는 Azure Pass를 제공하지 않습니다.

>**참고:** 이 워크샵은 Visual Studio Community Version 15.4.0을 사용하여 DSVM(Data Science Virtual Machine)에서 개발 및 테스트되었습니다.

## 소개

이 워크샵에서는 사진을 가져오고, Cognitive Services를 사용하여 이미지에서 개체와 사람을 찾고, 그 사람들의 감정을 파악하고, 이러한 모든 데이터를 NoSQL Store(DocumentDB)에 저장하는 종단 간 시나리오를 빌드합니다. NoSQL Store를 사용하여 Azure Search 인덱스를 채운 다음, 쉬운 대상 쿼리가 가능하도록 LUIS를 사용해 Bot Framework 봇을 빌드하겠습니다.

> **참고:** 이 랩은 `lab01.1-pcl_and_cognitive_services`에서 계속되는 것이므로 사진을 가져오고, Cognitive Services를 사용하여 이미지에 대한 정보를 파악하고, DocumentDB에 데이터를 저장하는 작업은 생략됩니다. 이 랩에서는 DocumentDB를 사용하여 검색 인덱스를 채우는 작업만 수행합니다. `lab01.1-pcl_and_cognitive_services`를 완료한 경우, 제공된 문자열 대신 DocumentDB 연결 문자열을 사용해도 됩니다.

## 아키텍처

`lab01.1-pcl_and_cognitive_services`에서는 로컬 드라이브에서 사진을 가져온 다음 여러 가지 Cognitive Services를 호출하여 이미지에 대한 데이터를 수집하는 단순한 C# 응용 프로그램을 빌드했습니다.

- [Computer Vision](https://www.microsoft.com/cognitive-services/ko-kr/computer-vision-api): 이를 통해 태그와 설명을 얻습니다.
- [Face](https://www.microsoft.com/cognitive-services/ko-kr/face-api): 이를 통해 각 이미지에서 얼굴과 세부 사항을 인식합니다.
- [Emotion](https://www.microsoft.com/cognitive-services/ko-kr/emotion-api): 이를 통해 이미지의 각 얼굴에서 감정 점수를 산출합니다.

이 데이터를 수집한 후 처리하고 필요한 모든 정보를 Microsoft [NoSQL](https://en.wikipedia.org/wiki/NoSQL) [PaaS](https://azure.microsoft.com/ko-kr/overview/what-is-paas/) 제품인 [DocumentDB](https://azure.microsoft.com/ko-kr/services/documentdb/)에 저장했습니다.

DocumentDB에 데이터를 저장했으니 이를 기반으로 [Azure Search](https://azure.microsoft.com/ko-kr/services/search/) 인덱스를 빌드합니다. Azure Search는 내결함성 패싯 검색을 위한 PaaS 제품입니다. 관리 오버헤드가 없는 Elastic Search라고 생각할 수 있습니다. 데이터를 쿼리하는 방법을 설명한 후 데이터를 쿼리하는 [Bot Framework](https://dev.botframework.com/) 봇을 빌드하겠습니다. 마지막으로, 쿼리에서 의도를 자동으로 도출하고 이를 바탕으로 지능적으로 검색을 수행하도록 [LUIS](https://www.microsoft.com/cognitive-services/ko-kr/language-understanding-intelligent-service-luis)를 사용하여 이 봇을 확장하겠습니다.

![아키텍처 다이어그램](./resources/assets/AI_Immersion_Arch.png)

> 이 랩은 이 [Cognitive Services 자습서](https://github.com/noodlefrenzy/CognitiveServicesTutorial)의 내용을 바탕으로 하여 적절히 수정되었습니다.

## GitHub 탐색 ##

[resources](./resources) 폴더에는 다음과 같은 여러 디렉터리가 있습니다.

- **assets**: 여기에는 랩 매뉴얼의 모든 이미지가 포함됩니다. 이 폴더는 무시할 수 있습니다.
- **code**: 여기에는 우리가 사용할 몇 가지 디렉터리가 있습니다.
	- **LUIS**: 여기서는 PictureBot의 LUIS 모델을 찾을 수 있습니다. 모델을 직접 만들지만 뒤처지거나 다른 LUIS 모델을 테스트하려는 경우 .json 파일을 사용하여 이 LUIS 앱을 가져올 수 있습니다.
	- **Models**: 이러한 클래스는 PictureBot에 검색을 추가할 때 사용됩니다.
	- **Finished-PictureBot**: 워크샵의 마지막 섹션에 사용되는 완성된 PictureBot.sln이 여기에 있습니다. 여기서 LUIS 및 검색 인덱스를 Bot Framework에 통합합니다. 작업이 뒤처지거나 막히면 이를 사용할 수 있습니다.

> 이러한 랩을 실행하려면 Visual Studio가 필요하지만 워크샵 중 하나에서 Windows Data Science Virtual Machine을 이미 배포한 경우 이를 사용할 수 있습니다.

## 키 수집

이 랩에서는 다양한 키를 수집합니다. 워크샵 전반에서 쉽게 액세스할 수 있도록 모든 키를 텍스트 파일에 저장하는 것이 좋습니다.

> _키_
>- LUIS API:
>- Cosmos DB 연결 문자열:
>- Azure Search 이름:
>- Azure Search 키:
>- Bot Framework 앱 이름:
>- Bot Framework 앱 ID:
>- Bot Framework 앱 암호:


## 랩 탐색

이 워크샵은 다음 5개 섹션으로 나뉩니다.
- [1_AzureSearch](./1_AzureSearch.md): 여기서는 Azure Search에 대해 알아보고 인덱스를 만듭니다.
- [2_LUIS](./2_LUIS.md): 봇(다음 랩에서 빌드)의 Language Understanding을 개선하기 위해 LUIS 모델을 빌드합니다.
- [3_Bot](./3_Bot.md): Bot Framework를 사용하여 모든 항목을 통합하는 방법을 알아봅니다.
- [4_Bot_Enhancements](./4_Bot_Enhancements.md): 마지막으로, 정규식을 사용하여 봇을 개선하고 Bot Connector를 통해 게시합니다.
- [5_Challenge_and_Closing](./4_Challenge_and_Closing.md): 모든 랩을 마쳤으면 이 챌린지를 시도하십시오. 또한 수행한 작업에 대한 요약과 자세한 내용을 알아볼 수 있는 위치도 확인할 수 있습니다.



### [1_AzureSearch](./1_AzureSearch.md)로 계속 진행


