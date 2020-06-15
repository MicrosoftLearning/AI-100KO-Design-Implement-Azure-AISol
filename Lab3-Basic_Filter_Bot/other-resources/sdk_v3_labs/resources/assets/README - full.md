# lab02.3-luis_and_search - LUIS 및 Azure Cognitive Search를 사용하여 지능형 애플리케이션 개발

이 실습 랩에서는 Microsoft Bot Framework, Azure Cognitive Search 및 여러 가지 Cognitive Services를 사용하여 엔드투엔드 지능형 봇을 만드는 방법을 안내합니다. 

이 워크샵에서는 다음과 같은 내용을 다룹니다.
- 응용 프로그램에 지능형 서비스를 통합하는 방법 이해
- 애플리케이션 내에서 긍정적인 검색 경험을 제공하도록 Azure Cognitive Search 기능을 구현하는 방법 이해
- 데이터를 확장하여 전체 텍스트, 언어 인식 검색을 사용하도록 Azure Cognitive Search 서비스 구성
- 봇이 효과적으로 통신할 수 있도록 LUIS 모델 빌드, 학습 및 게시
- LUIS 및 Azure Cognitive Search를 활용하는 Microsoft Bot Framework를 사용하여 지능형 봇 빌드
- .NET 응용 프로그램에서 다양한 Cognitive Services API(특히 Computer Vision, Face, Emotion 및 LUIS) 호출

LUIS 및 Azure Cognitive Search에 중점을 두고 있지만 다음과 같은 기술도 활용합니다.

- Computer Vision API
- Face API
- Emotion API
- DSVM(Data Science Virtual Machine)
- Windows 10 SDK(UWP)
- CosmosDB
- Azure Storage
- Visual Studio

## 사전 요건

이 워크샵은 Azure의 AI 개발자를 위한 것입니다. 반나절 워크샵이므로 시작 전에 충족되어야 하는 사항들이 있습니다.

첫째, Visual Studio를 사용한 경험이 있어야 합니다. 워크샵에서 빌드하는 모든 것에 Visual Studio가 사용되므로 응용 프로그램을 만들기 위한 [사용 방법](https://docs.microsoft.com/ko-kr/visualstudio/ide/visual-studio-ide)을 숙지하고 있어야 합니다. 또한 이 워크샵은 응용 프로그램 코딩 또는 개발 방법을 가르치는 수업이 아니며, C#에서 코딩하는 방법([여기](https://mva.microsoft.com/ko-kr/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949)에서 학습할 수 있음)은 알고 있지만 고급 검색 및 NLP(자연어 처리) 솔루션을 구현하는 방법은 알지 못한다고 가정합니다. 

둘째, Microsoft의 Bot Framework를 사용하여 봇을 개발한 경험이 있어야 합니다. 봇 디자인 방법이나 대화 상자 작동 방식에 대해 설명하는 데는 많은 시간을 할애하지 않습니다. Bot Framework에 익숙하지 않은 경우 워크샵에 참가하기 전에 [이 Microsoft Virtual Academy 과정](https://mva.microsoft.com/ko-kr/training-courses/creating-bots-in-the-microsoft-bot-framework-using-c-17590#!)을 수강해야 합니다.

셋째, 포털을 사용한 경험이 있으며 Azure에서 리소스를 만들고 비용을 지출할 수 있어야 합니다. 이 워크샵에서는 Azure Pass를 제공하지 않습니다.

## 소개

이 워크샵에서는 사진을 가져오고, Cognitive Services를 사용하여 이미지에서 개체와 사람을 찾고, 그 사람들의 감정을 파악하고, 이러한 모든 데이터를 NoSQL Store(DocumentDB)에 저장하는 종단 간 시나리오를 빌드합니다. NoSQL Store를 사용하여 Azure Cognitive Search 인덱스를 채운 다음, 쉬운 대상 쿼리가 가능하도록 LUIS를 사용해 Bot Framework 봇을 빌드합니다.

## 아키텍처

로컬 드라이브에서 사진을 가져온 다음 여러 가지 Cognitive Services를 호출하여 이미지에 대한 데이터를 수집하는 단순한 C# 응용 프로그램을 빌드합니다.

- [Computer Vision](https://www.microsoft.com/cognitive-services/ko-kr/computer-vision-api): 이를 통해 태그와 설명을 얻습니다.
- [Face](https://www.microsoft.com/cognitive-services/ko-kr/face-api): 이를 통해 각 이미지에서 얼굴과 세부 사항을 인식합니다.
- [Emotion](https://www.microsoft.com/cognitive-services/ko-kr/emotion-api): 이를 통해 이미지의 각 얼굴에서 감정 점수를 산출합니다.

이러한 데이터가 확보되었으면 필요한 세부 정보를 끌어내어 Microsoft [NoSQL](https://en.wikipedia.org/wiki/NoSQL) [PaaS](https://azure.microsoft.com/ko-kr/overview/what-is-paas/) 제품인 [DocumentDB](https://azure.microsoft.com/ko-kr/services/documentdb/)에 저장합니다.

DocumentDB에 데이터를 저장한 후 이를 기반으로 [Azure Cognitive Search](https://azure.microsoft.com/ko-kr/services/search/) 인덱스를 빌드합니다. Azure Cognitive Search는 내결함성 패싯 검색을 위한 PaaS 제품입니다. 관리 오버헤드가 없는 탄력적 검색으로 볼 수 있습니다. 데이터를 쿼리하는 방법을 설명한 후 데이터를 쿼리하는 [Bot Framework](https://dev.botframework.com/) 봇을 빌드하겠습니다. 마지막으로, 쿼리에서 의도를 자동으로 도출하고 이를 바탕으로 지능적으로 검색을 수행하도록 [LUIS](https://www.microsoft.com/cognitive-services/ko-kr/language-understanding-intelligent-service-luis)를 사용하여 이 봇을 확장하겠습니다. 

![아키텍처 다이어그램](./resources/assets/AI_Immersion_Arch.png)


## GitHub 탐색 ##

[resources](./resources) 폴더에는 다음과 같은 여러 디렉터리가 있습니다.

- **assets**: 여기에는 랩 매뉴얼의 모든 이미지가 포함됩니다. 이 폴더는 무시할 수 있습니다.
- **code**: 여기에는 우리가 사용할 몇 가지 디렉터리가 있습니다.
	- **ImageProcessing**: 이 솔루션(.sln)은 워크샵의 첫 번째 부분에 대한 여러 가지 프로젝트를 포함하고 있습니다. 개괄적으로 살펴보겠습니다.
		- **ImageProcessingLibrary**: Vision과 관련된 다양한 Cognitive Services에 액세스하기 위한 도우미 클래스와 결과를 캡슐화하기 위한 몇 가지 "인사이트" 클래스가 포함된 PCL(이식 가능한 클래스 라이브러리)입니다.
		- **ImageStorageLibrary**: Cosmos DB는 UWP를 지원하지 않기 때문에 Blob Storage와 Cosmos DB에 액세스하기 위한 이식 불가능한 라이브러리입니다.
		- **TestApp**: 이미지를 로드하고 이미지에 대한 다양한 Cognitive Services를 호출한 후 결과를 살펴볼 수 있도록 하는 UWP 응용 프로그램입니다. 이미지에 대한 실험 및 탐색에 유용합니다.
		- **TestCLI**: 다양한 Cognitive Services를 호출한 다음 이미지와 데이터를 Azure에 업로드할 수 있는 콘솔 응용 프로그램입니다. 이미지는 Blob Storage에 업로드되고 다양한 메타데이터(태그, 캡션, 얼굴)가 Cosmos DB에 업로드됩니다.

		_TestApp_ 및 _TestCLI_에는 Cognitive Services 및 Azure에 액세스하는 데 필요한 다양한 키와 끝점이 들어 있는 `settings.json` 파일이 포함됩니다. 이러한 디렉터리는 처음에 비어 있으므로 리소스를 프로비전한 후에 서비스 키를 가져오고 스토리지 계정 및 Cosmos DB 인스턴스를 설정해야 합니다.
		
	- **LUIS**: 여기서는 PictureBot의 LUIS 모델을 찾을 수 있습니다. 모델을 직접 만들지만 뒤처지거나 다른 LUIS 모델을 테스트하려는 경우 .json 파일을 사용하여 이 LUIS 앱을 가져올 수 있습니다.
	- **Models**: 이러한 클래스는 PictureBot에 검색을 추가할 때 사용됩니다.
	- **PictureBot**: 워크샵의 마지막 섹션에 사용되는 PictureBot.sln이 여기에 있습니다. 여기서 LUIS 및 검색 인덱스를 Bot Framework에 통합합니다.


### 랩: Azure 계정 설정

[https://azure.microsoft.com/ko-kr/free/](https://azure.microsoft.com/ko-kr/free/)에서 Azure 무료 평가판을 활성화할 수 있습니다.  

이 랩을 완료하기 위해 Azure Pass가 제공된 경우 [http://www.microsoftazurepass.com/](http://www.microsoftazurepass.com/)으로 이동하여 활성화할 수 있습니다.  활성화 프로세스를 안내하는 [https://www.microsoftazurepass.com/howto](https://www.microsoftazurepass.com/howto)의 지침을 따르십시오.  Microsoft 계정에는 Azure **무료 평가판 하나**와 이에 연결된 Azure Pass 하나가 있을 수 있으므로, Microsoft 계정에서 Azure Pass를 이미 활성화한 경우 무료 평가판을 사용하거나 다른 Microsoft 계정을 사용해야 합니다.

### 랩: Data Science Virtual Machine 설정

Azure 계정을 만든 후에는 [Azure Portal](https://portal.azure.com)에 액세스할 수 있습니다. 포털에서 이 랩을 위한 리소스 그룹을 만듭니다. Data Science Virtual Machine에 대한 자세한 정보는 [온라인에서 확인](https://docs.microsoft.com/ko-kr/azure/machine-learning/data-science-virtual-machine/overview)할 수 있으며, 여기서는 이 워크샵에 필요한 내용만 살펴보겠습니다. 리소스 그룹에서 DS3_V2 크기(다른 모든 기본값은 그대로 사용 가능)로 Data Science Virtual Machine for Windows(2016)를 배포하고 이에 연결합니다. 

연결된 후에는 다음과 같은 몇 가지 작업을 수행하여 워크샵을 위해 DSVM을 설정해야 합니다.

1. Firefox에서 이 리포지토리로 이동하여 zip 파일로 다운로드합니다. 모든 파일을 추출하고 이 랩의 폴더를 바탕 화면으로 이동합니다.
2. resources>code>ImageProcessing 아래에 있는 "ImageProcessing.sln"을 엽니다. Visual Studio가 처음 열리는 데 시간이 걸릴 수 있으며 로그인해야 합니다.
3. 열린 후에는 Windows 10 App Development(UWP)용 SDK를 설치하라는 메시지가 표시됩니다. 표시된 메시지에 따라 설치합니다. 메시지가 표시되지 않을 경우 TestApp을 마우스 오른쪽 단추로 클릭하고 "프로젝트 다시 로드"를 선택하면 메시지가 표시됩니다.
4. 설치하는 동안 몇 가지 작업을 수행할 수 있습니다. 
	- Cortana 검색 창에 "개발자용 설정"을 입력하고 설정을 "개발자 모드"로 변경합니다.
	- Cortana 검색 창에 "gpedit.msc"를 입력하고 Enter 키를 누릅니다. 다음 정책을 사용하도록 설정합니다. 컴퓨터 구성>Windows 설정>보안 설정>로컬 정책>보안 옵션>사용자 계정 컨트롤: 기본 제공 관리자 계정에 대한 관리자 승인 모드
	- "키 수집" 랩을 시작합니다. 
5. 설치가 완료되고 개발자 설정 및 사용자 계정 컨트롤 정책을 변경한 후 DSVM을 재부팅합니다. 
> **참고** 워크샵이 끝난 후에는 요금이 부과되지 않도록 DSVM을 끄십시오.


### 랩: 키 수집

이 랩에서는 Cognitive Services 키와 저장소 키를 수집합니다. 이후 랩에서 쉽게 액세스할 수 있도록 모든 키를 텍스트 파일에 저장하는 것이 좋습니다.

>_Cognitive Services 키_
>- Computer Vision API:
>- Face API:
>- Emotion API:
>- LUIS API:

>- 저장소 키_
>- Azure Blob Storage 연결 문자열:
>- Cosmos DB URI:
>- Cosmos DB 키:

**Cognitive Services API 키 얻기**

포털 내에서 먼저 사용할 Cognitive Services용 키를 만듭니다. 주로 [Computer Vision](https://www.microsoft.com/cognitive-services/ko-kr/computer-vision-api) Cognitive Services의 여러 가지 API를 사용하므로 먼저 이를 위한 API 키를 만들겠습니다.

포털에서 **새로 만들기**를 누른 후 검색 상자에 **cognitive**를 입력하고 **Cognitive Services**를 선택합니다.

![Cognitive Services 키 만들기](./resources/assets/new-cognitive-services.PNG)

다음으로, 만들 API 끝점에 대한 몇 가지 세부 정보를 작성하고 원하는 API와 끝점을 사용할 위치 그리고 요금제를 선택합니다. 자습서에 필요한 처리량을 위해 S1을 사용하여 새 _리소스 그룹_을 만듭니다. Blob Storage 및 Cosmos DB에도 이 리소스 그룹을 동일하게 사용하려고 합니다. 쉽게 찾을 수 있도록 _대시보드에 고정_합니다. Computer Vision API는 향후 Cognitive Services Vision 제품을 개선하기 위해 이미지를 Microsoft에 내부적으로 안전하게 저장하므로 계정 만들기를 _사용하도록 설정_해야 합니다. 구독 관리자만 이 기능을 사용하도록 설정할 수 있으므로 엔터프라이즈 환경의 사용자에게는 걸림돌이 될 수 있지만 Azure Pass 사용자에게는 문제가 되지 않습니다.

![Cognitive Services 세부 정보 선택](./resources/assets/cognitive-account-creation.PNG) 

새 API 구독을 만든 후에는 블레이드의 해당 섹션에서 키를 가져와 _TestApp_ 및 _TestCLI_의 `settings.json` 파일에 추가할 수 있습니다.

![Cognitive API 키](./resources/assets/cognitive-keys.PNG)

또한 Computer Vision 제품군 내 다른 API도 사용할 것이므로 이 기회에 _Emotion_ 및 _Face_ API를 위한 API 키를 만듭니다. 이러한 키는 위와 동일한 방식으로 만들어지며 아까 만든 것과 동일한 리소스 그룹을 재사용합니다. _대시보드에 고정_하고 `settings.json` 파일에 해당 키를 추가합니다.

자습서의 뒷부분에서 [LUIS](https://www.microsoft.com/cognitive-services/ko-kr/language-understanding-intelligent-service-luis)를 사용하므로 이 기회에 LUIS 구독도 만들겠습니다. LUIS 구독은 위와 동일한 방식으로 만들어지지만 API 드롭다운에서 Language Understanding Intelligent Service를 선택하고 위에서 만든 것과 동일한 리소스 그룹을 재사용합니다. 자습서의 해당 단계에서 쉽게 찾을 수 있도록 이번에도 _대시보드에 고정_합니다.  

**저장소 설정**

이 프로젝트에서는 Azure에 두 가지 저장소를 사용합니다. 원시 이미지를 저장하기 위한 저장소와 Cognitive Services 호출 결과를 저장하기 위한 저장소가 그것입니다. Azure Blob Storage는 파일 시스템과 유사한 형식으로 대량의 데이터를 저장하기 위해 만들어졌으며 이미지 같은 데이터를 저장하기에 매우 적합합니다. Azure Cosmos DB는 복원력 있는 NoSQL PaaS 솔루션으로, 이미지 메타데이터 결과와 같은 느슨하게 구조화된 데이터를 저장하는 데 매우 유용합니다. 다른 선택(Azure Table Storage, SQL Server)을 할 수도 있지만, Cosmos DB는 스키마를 자유롭게 변경하고(예: 새 서비스를 위한 데이터 추가), 손쉽게 쿼리를 수행하고, Azure Cognitive Search에 빠르게 통합할 수 있는 유연성을 제공합니다.

_Azure Blob Storage_

자세한 "시작" 지침을 [온라인에서 확인](https://docs.microsoft.com/ko-kr/azure/storage/storage-dotnet-how-to-use-blobs)할 수 있으며, 여기서는 이 랩에 필요한 사항만 살펴보겠습니다.

Azure Portal 내에서 **새로 만들기->저장소->스토리지 계정**을 클릭합니다.

![새 Azure Storage](./resources/assets/create-blob-storage.PNG)

클릭하면 위의 필드가 작성을 위해 표시됩니다. 스토리지 계정 이름(소문자 및 숫자)을 선택하고, _계정 종류_를 _Blob Storage_로 설정하고, _복제_를 _LRS(로컬 중복 저장소)_(비용을 절감하기 위한 것임)로 설정하며, 위와 동일한 리소스 그룹을 사용하고, _위치_를 _미국 서부_로 설정합니다.  각 지역에서 사용할 수 있는 Azure 서비스 목록은 https://azure.microsoft.com/ko-kr/regions/services/에 나와 있습니다. 쉽게 찾을 수 있도록 _대시보드에 고정_합니다.

이제 Azure Storage 계정이 있으므로 _연결 문자열_을 가져와서 _TestCLI_ 및 _TestApp_ `settings.json`에 추가하겠습니다.

![Azure Blob 키](./resources/assets/blob-storage-keys.PNG)

_Cosmos DB_

자세한 "시작" 지침을 [온라인에서 확인](https://docs.microsoft.com/ko-kr/azure/documentdb/documentdb-get-started)할 수 있으며, 여기서는 이 프로젝트에 필요한 사항만 살펴보겠습니다.

Azure Portal 내에서 **새로 만들기->데이터베이스->Azure Cosmos DB**를 클릭합니다.

![새 Cosmos DB](./resources/assets/create-cosmosdb-portal.png)

이를 클릭한 후에는 필요한 필드를 몇 가지 작성해야 합니다. 

![Cosmos DB 작성 양식](./resources/assets/create-cosmosdb-formfill.png)

소문자, 숫자, 대시만 사용해야 한다는 제약 조건에 따라 원하는 ID를 선택합니다. Mongo가 아닌 Document DB SDK를 사용할 것이므로 NoSQL API로 Document DB를 선택합니다. 이전 단계에 사용한 것과 동일한 리소스 그룹 및 동일한 위치를 사용하며, 지속적으로 추적하고 쉽게 돌아올 수 있도록 _대시보드에 고정_을 선택한 다음 만들기를 누릅니다.

작성이 완료되면 새 데이터베이스의 패널을 열고 _키_ 하위 패널을 선택합니다.

![Cosmos DB용 키 하위 패널](./resources/assets/docdb-keys.png)

_TestCLI_의 `settings.json` 파일에 **URI** 및 **기본 키**가 필요하므로 이를 파일에 복사합니다. 이제 클라우드에 이미지와 데이터를 저장할 수 있습니다.


## Cognitive Services

이 워크샵에서는 모든 Cognitive Services API가 아니라 Azure Cognitive Search와 LUIS에 초점을 둡니다. 그러나 Cognitive Services에 대한 실습을 더 해보고 싶은 경우 TODO: Add reference lab link-Lab 1을 통해 다양한 Cognitive Services를 사용하여 지능형 키오스크를 빌드하는 것이 좋습니다. 

이 워크샵에서는 Computer Vision API를 사용하여 태그와 설명을 통해 이미지를 파악하고, Face API를 사용하여 업로드하는 이미지의 모든 얼굴을 추적하며, Emotion API를 사용하여 이미지에 있는 사람들의 감정 점수를 산출합니다. 단지 이러한 정보를 얻기 위해 API를 호출하며, 수행해야 할 학습이나 테스트는 없습니다. 

결과 정보에는 태그, 설명, 성별/연령/감정(사람인 경우)이 포함됩니다. 다음 랩을 완료한 후 생성된 ImageInsights.json 파일을 통해 이 정보가 어떻게 구조화되어 있는지 확인합니다.

봇이 더 효과적으로 언어를 이해를 할 수 있도록 이후에 약간의 시간을 들여 LUIS 모델을 개발할 것입니다. 이 서비스의 경우 학습 및 게시 전에 모델에 의도, 엔터티, 발화 등을 추가합니다. 그런 다음에야 API 호출에서 액세스할 수 있습니다.

### 랩: Cognitive Services 및 Image Processing Library 탐색

`ImageProcessing.sln` 솔루션을 열면 이미지를 로드하고 다양한 Cognitive Services를 호출한 후 결과를 살펴볼 수 있도록 하는 UWP 응용 프로그램(`TestApp`)을 찾을 수 있습니다. 이는 이미지에 대한 실험 및 탐색에 유용합니다. 이 앱은 TestCLI 프로젝트에서 이미지를 분석하는 데도 사용되는 `ImageProcessingLibrary` 프로젝트를 기반으로 합니다. 

솔루션을 "빌드"하려면 `ImageProcessing.sln`을 마우스 오른쪽 단추로 클릭하고 "빌드"를 선택합니다. 또한 TestApp 프로젝트를 다시 로드해야 할 경우 마우스 오른쪽 단추로 프로젝트를 클릭하고 "프로젝트 다시 로드"를 선택하면 됩니다. 

앱을 실행하기 전에 `TestApp` 프로젝트 아래의 `settings.json` 파일에 Cognitive Services API 키를 입력해야 합니다. 그런 다음 앱을 실행하고 `폴더 선택` 단추를 통해 이미지를 배치할 폴더를 지정합니다(먼저 `sample_images`의 압축을 풀어야 함). 그러면 처리되는 모든 이미지와 이미지 컬렉션에 대한 필터로 사용되는 고유한 얼굴, 감정 및 태그가 표시됩니다.

![UWP 테스트 앱](./resources/assets/UWPTestApp.JPG)

앱이 지정된 디렉터리를 처리한 후에는 동일한 폴더의 `ImageInsights.json` 파일에 결과를 캐시하므로 다양한 API를 호출하지 않고도 해당 폴더 결과를 다시 볼 수 있습니다. 

## Cosmos DB 탐색

Cosmos DB는 이 워크샵의 초점이 아니지만, 관련 내용에 관심이 있다면 사용할 코드의 주요 사항은 다음과 같습니다.
- `ImageStorageLibrary`의 `DocumentDBHelper.cs` 클래스로 이동합니다. 사용하는 구현의 대부분을 [시작 가이드](https://docs.microsoft.com/ko-kr/azure/documentdb/documentdb-get-started)에서 찾을 수 있습니다.
- `TestCLI`의 `Util.cs`로 이동하여 `ImageMetadata` 클래스를 검토합니다. 여기서 Cognitive Services에서 검색한 `ImageInsights`를 Cosmos DB에 저장하기 위한 적절한 메타데이터로 전환합니다.
- 마지막으로 `Program.cs`와 `ProcessDirectoryAsync`에서 확인합니다. 우선, 이미지와 메타데이터가 이미 업로드되었는지 확인합니다. `DocumentDBHelper`를 사용하여 ID로 문서를 찾으며 문서가 존재하지 않는 경우 `null`이 반환됩니다. 다음으로 `forceUpdate`를 설정했거나 이전에 이미지가 처리되지 않은 경우 `ImageProcessingLibrary`에서 `ImageProcessor`를 사용하여 Cognitive Services를 호출하고 `ImageInsights`를 검색하여 현재 `ImageMetadata`에 추가합니다. 
- 이 모든 작업이 완료되면 먼저 `BlobStorageHelper` 인스턴스를 사용하여 실제 이미지를 Blob Storage에 저장한 다음 `DocumentDBHelper` 인스턴스를 사용하여 Cosmos DB에 `ImageMetadata`를 저장할 수 있습니다. 이전 확인에 따라 문서가 이미 있는 경우 기존 문서를 업데이트하고, 없는 경우 새 문서를 만들어야 합니다.

### 랩: TestCLI를 사용하여 이미지 로드

이벤트 루프, 양식 또는 기타 UX 관련 사항으로 주의가 산만해지지 않고 처리 코드에 집중할 수 있도록 기본 처리 및 저장소 코드를 명령줄/콘솔 응용 프로그램으로 구현합니다. 나중에 고유한 UX를 추가해 보십시오.

Cognitive Services API 키, Azure Blob Storage 연결 문자열 및 Cosmos DB 끝점 URI 및 키를 _TestCLI_의 `settings.json`에서 설정한 후에는 _TestCLI_를 실행할 수 있습니다.

_TestCLI_를 실행한 다음 명령 프롬프트를 열고 ImageProcessing\TestCLI 폴더로 이동합니다(힌트: "cd" 명령을 사용하여 디렉터리 변경). 그런 다음 `.\bin\Debug\TestCLI.exe`를 입력합니다. 다음과 같은 결과를 얻어야 합니다.

    > .\bin\Debug\TestCLI.exe

    사용법:  [옵션]

    옵션:
    -force            파일이 이미 추가된 경우에도 강제로 업데이트하려면 사용합니다.
    -settings         설정 파일(선택 사항, 설정되지 않은 경우 임베디드 리소스 settings.json이 사용됨)
    -process          처리할 디렉터리
    -query            실행할 쿼리
    -? | -h | --help  도움말 정보를 표시합니다.

기본적으로 `settings.json`(`.exe`에 기본으로 포함됨)에서 설정이 로드되지만 `-settings` 플래그를 사용하여 고유한 설정을 제공할 수 있습니다. 이미지(및 Cognitive Services의 메타데이터)를 클라우드 저장소에 로드하려면 다음과 같이 _TestCLI_가 이미지 디렉터리를 `-process`하도록 지시할 수 있습니다.

    > .\bin\Debug\TestCLI.exe -process c:\my\image\directory

처리가 완료되면 다음과 같이 _TestCLI_를 사용하여 Cosmos DB에 직접 쿼리할 수 있습니다.

    > .\bin\Debug\TestCLI.exe -query "select * from images"

## Azure Cognitive Search

[Azure Cognitive Search](https://docs.microsoft.com/ko-kr/azure/search/search-what-is-azure-search)는 개발자가 인프라를 관리하거나 검색 전문가가 되지 않고도 애플리케이션에 훌륭한 검색 환경을 통합할 수 있게 해주는 SaaS(Search-as-a-Service) 솔루션입니다.

개발자는 앱에서 더 나은 결과를 더 빠르게 얻을 수 있도록 Azure에서 PaaS 서비스를 이용하길 원합니다. 검색은 많은 범주의 응용 프로그램에서 핵심적인 요소입니다. 웹 검색 엔진의 검색 품질이 높아짐에 따라 사용자는 즉각적인 결과, 입력 시 자동 완성, 결과 내 검색어 강조 표시, 우수한 순위 지정, 맞춤법이 잘못되거나 불필요한 단어가 포함되어 있더라도 검색하려는 항목을 파악하는 기능 등을 기대합니다.

검색은 어렵고 핵심 전문 분야인 경우가 드뭅니다. 인프라 관점에서는 뛰어난 가용성, 내구성, 확장성 및 운영 능력이 필요하며, 기능적 관점에서는 순위 지정, 언어 지원 및 지리 공간적 기능이 있어야 합니다.

![검색 요구 사항의 예시](./resources/assets/AzureSearch-Example.png) 

위의 예는 사용자가 검색 환경에서 기대하는 일부 구성 요소를 보여 줍니다. [Azure Cognitive Search](https://docs.microsoft.com/ko-kr/azure/search/search-what-is-azure-search)는 [모니터링 및 보고](https://docs.microsoft.com/ko-kr/azure/search/search-traffic-analytics), [간단한 채점](https://docs.microsoft.com/ko-kr/rest/api/searchservice/add-scoring-profiles-to-a-search-index) 그리고 [프로토타입 생성](https://docs.microsoft.com/ko-kr/azure/search/search-import-data-portal) 및 [검사](https://docs.microsoft.com/ko-kr/azure/search/search-explorer) 도구를 제공하며 이러한 사용자 환경 기능을 구현할 수 있습니다.

일반적인 워크플로:
1. 서비스 프로비전
	- [포털](https://docs.microsoft.com/ko-kr/azure/search/search-create-service-portal) 또는 [PowerShell](https://docs.microsoft.com/ko-kr/azure/search/search-manage-powershell)을 사용하여 Azure Cognitive Search 서비스를 만들거나 프로비전할 수 있습니다.
2. 인덱스 만들기
	- [인덱스](https://docs.microsoft.com/ko-kr/azure/search/search-what-is-an-index)는 데이터, 인지 "테이블"을 위한 컨테이너이며, 스키마, [CORS 옵션](https://docs.microsoft.com/ko-kr/aspnet/core/security/cors), 검색 옵션을 갖습니다. [포털](https://docs.microsoft.com/ko-kr/azure/search/search-create-index-portal)에서 또는 [앱 초기화](https://docs.microsoft.com/ko-kr/azure/search/search-create-index-dotnet) 중에 인덱스를 만들 수 있습니다. 
3. 인덱스 데이터
	- [데이터로 인덱스를 채우는](https://docs.microsoft.com/ko-kr/azure/search/search-what-is-data-import) 방법에는 두 가지가 있습니다. 첫 번째 방법은 Azure Cognitive Search [REST API](https://docs.microsoft.com/ko-kr/azure/search/search-import-data-rest-api) 또는 [.NET SDK](https://docs.microsoft.com/ko-kr/azure/search/search-import-data-dotnet)를 사용하여 데이터를 인덱스에 수동으로 푸시하는 것입니다. 두 번째 방법은 [지원되는 데이터 원본](https://docs.microsoft.com/ko-kr/azure/search/search-indexer-overview)에서 인덱스를 가리키고 Azure Cognitive Search가 일정에 따라 데이터를 자동으로 끌어오도록 하는 것입니다.
4. 인덱스 검색
	- Azure Cognitive Search에 검색 요청을 제출할 때 간단한 검색 옵션을 사용할 수 있으며 [필터](https://docs.microsoft.com/ko-kr/azure/search/search-filters), [정렬](https://docs.microsoft.com/ko-kr/rest/api/searchservice/add-scoring-profiles-to-a-search-index), [예측](https://docs.microsoft.com/ko-kr/azure/search/search-faceted-navigation) 및 [여러 페이지에 결과 표시](https://docs.microsoft.com/ko-kr/azure/search/search-pagination-page-layout) 기능을 사용할 수 있습니다. 맞춤법 오류, 음성 및 Regex를 처리할 수 있으며 검색 및 [제안](https://docs.microsoft.com/ko-kr/rest/api/searchservice/suggesters) 작업을 위한 옵션이 있습니다. 이러한 쿼리 매개 변수를 사용하면 [전체 텍스트 검색 환경](https://docs.microsoft.com/ko-kr/azure/search/search-query-overview)을 보다 세부적으로 제어할 수 있습니다.


### 랩: Azure Cognitive Search 서비스 만들기

Azure Portal에서 **새로 만들기->웹 + 모바일->Azure Cognitive Search**를 클릭합니다.

이를 클릭한 후에는 필요한 필드를 몇 가지 작성해야 합니다. 이 랩에서는 "무료" 계층으로 충분합니다.

![새 Azure Cognitive Search 서비스 만들기](./resources/assets/AzureSearch-CreateSearchService.png)

만들기가 완료되면 새 Search Service의 패널을 엽니다.

### 랩: Azure Cognitive Search 인덱스 만들기

인덱스는 데이터의 컨테이너이며 SQL Server 테이블의 인덱스와 유사한 개념입니다.  테이블에 행이 있는 것처럼 인덱스에는 문서가 있습니다.  테이블에 필드가 있는 것처럼 인덱스에도 필드가 있습니다.  이러한 필드에는 전체 텍스트 검색이 가능한지 또는 필터링이 가능한지 등을 알려주는 속성이 있을 수 있습니다.  프로그래밍 방식으로 [콘텐츠를 푸시](https://docs.microsoft.com/ko-kr/rest/api/searchservice/addupdate-or-delete-documents)하거나 [Azure Cognitive Search Indexer](https://docs.microsoft.com/ko-kr/azure/search/search-indexer-overview)(공통 데이터 저장소 크롤링 가능)를 사용하여 Azure Cognitive Search에 콘텐츠를 채울 수 있습니다.

이 랩에서는 [Cosmos DB용 Azure Cognitive Search Indexer](https://docs.microsoft.com/ko-kr/azure/search/search-howto-index-documentdb)를 사용하여 Cosmos DB 컨테이너의 데이터를 크롤링합니다. 

![가져오기 마법사](./resources/assets/AzureSearch-ImportData.png) 

방금 만든 Azure Cognitive Search 블레이드에서 **데이터 가져오기->데이터 원본->문서 DB**를 클릭합니다.

![DocDB용 가져오기 마법사](./resources/assets/AzureSearch-DataSource.png) 

이것을 클릭한 후에 Cosmos DB 데이터 원본의 이름을 선택하고 데이터가 있는 Cosmos DB 계정과 해당 컨테이너 및 컬렉션을 선택합니다.  

**확인**을 클릭합니다.

그러면 Azure Cognitive Search는 Cosmos DB 컨테이너에 연결하고 몇 가지 문서를 분석하여 Azure Cognitive Search 인덱스의 기본 스키마를 식별합니다.  이 작업을 완료하면 응용 프로그램에 필요한 대로 필드에 속성을 설정할 수 있습니다.

인덱스 이름을 **images**로 업데이트

키를 **id**(각 문서를 고유하게 식별)로 업데이트

모든 필드를 **Retrievable**로 설정(검색 시 클라이언트가 이러한 필드를 검색할 수 있음)

**Tags, NumFaces 및 Faces** 필드를 **Filterable**로 설정(클라이언트가 이러한 값을 기반으로 결과를 필터링할 수 있음)

**NumFaces** 필드를 **Sortable**로 설정(클라이언트가 이미지의 얼굴 수에 따라 결과를 정렬할 수 있음)

**Tags, NumFaces 및 Faces** 필드를 **Facetable**로 설정(클라이언트가 결과를 개수별로 그룹화할 수 있음. 예를 들어 검색 결과에 "해변"이라는 태그가 있는 사진이 5개 있음)

**Caption, Tags 및 Faces**를 **Searchable**로 설정(클라이언트가 이러한 필드의 텍스트에 대해 전체 텍스트 검색을 수행할 수 있음)

![Azure Cognitive Search 인덱스 구성](./resources/assets/AzureSearch-ConfigureIndex.png) 

이제 Azure Cognitive Search 분석기를 구성합니다.  간단히 말해, 분석기는 사용자가 입력하는 검색어를 확인한 후 인덱스에서 가장 일치하는 용어를 찾는 기능으로 생각할 수 있습니다.  Azure Cognitive Search에는 56개 언어를 심층적으로 이해하는 Bing 및 Office와 같은 기술에 사용되는 분석기가 포함되어 있습니다.  

**분석기** 탭을 클릭하고 **영어-Microsoft** 분석기를 사용하도록 **Caption, Tags 및 Faces** 필드를 설정합니다.

![언어 분석기](./resources/assets/AzureSearch-Analyzer.png) 

최종 인덱스 구성 단계에서는 자동 완성에 사용될 필드를 설정하여 사용자가 단어의 일부를 입력하면 Azure Cognitive Search가 이러한 필드에서 가장 일치하는 항목을 찾을 수 있도록 합니다.

**제안기** 탭을 클릭하고 제안기 이름을 **sg**로 입력한 후 추천 용어를 찾을 필드로 **태그 및 얼굴**을 선택합니다.

![검색 제안](./resources/assets/AzureSearch-Suggester.png) 

**확인**을 클릭하여 인덱서의 구성을 완료합니다.  인덱서가 변경 사항을 확인하는 빈도에 대한 일정을 설정할 수 있지만 이 랩에서는 한 번만 실행하겠습니다.  

**고급 옵션**을 클릭하고 **Base-64 인코딩 키**를 선택하여 Azure Cognitive Search 키 필드에서 지원되는 문자만 ID 필드에 사용되도록 합니다.

**확인, 세 번**을 클릭하여 Cosmos DB 데이터베이스에서 데이터를 가져오는 인덱서 작업을 시작합니다.

![인덱서 구성](./resources/assets/AzureSearch-ConfigureIndexer.png) 

***검색 인덱스 쿼리***

인덱싱이 시작되었음을 알리는 메시지가 나타납니다.  인덱스의 상태를 확인하려면 기본 Azure Cognitive Search 블레이드에서 "인덱스" 옵션을 선택합니다.

이제 인덱스를 검색해 볼 수 있습니다.  

**검색 탐색기**를 클릭하고 결과 블레이드에서 인덱스가 아직 선택되지 않은 경우 인덱스를 선택합니다.

**검색**을 클릭하여 모든 문서를 검색합니다.

![검색 탐색기](./resources/assets/AzureSearch-SearchExplorer.png)

**일찍 끝났다면 이 추가 크레딧 랩을 사용해 보십시오.**

[Postman](https://www.getpostman.com/)은 Azure Cognitive Search REST API 호출을 쉽게 실행할 수 있도록 지원하는 유용한 도구이자 훌륭한 디버깅 도구입니다.  Azure Cognitive Search Explorer의 모든 쿼리를 Azure Cognitive Search API 키와 함께 Postman에서 실행할 수 있습니다.

[Postman](https://www.getpostman.com/) 도구를 다운로드하여 설치합니다. 

설치한 후 Azure Cognitive Search Explorer에서 쿼리를 가져와 Postman에 붙여 넣은 다음 GET을 요청 유형으로 선택합니다.  

헤더를 클릭하고 다음 매개 변수를 입력합니다.

+ 콘텐츠 형식: application/json
+ api-key: [Azure Cognitive Search 포털의 "키" 섹션에 API 키 입력]

보내기를 선택하면 JSON 형식의 데이터가 표시됩니다.

[이와 같은 예제](https://docs.microsoft.com/ko-kr/rest/api/searchservice/search-documents#a-namebkmkexamplesa-examples)를 사용하여 다른 검색을 수행해 보십시오.


## LUIS

먼저, [LUIS(Language Understand Intelligent Service)에 대해 알아보겠습니다](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/Home).

이제 LUIS가 무엇인지 알았으므로 LUIS 앱을 계획할 수 있습니다. 검색 결과를 기반으로 이미지를 반환하고 사용자가 공유하거나 주문할 수 있도록 하는 봇을 만들려고 합니다. 봇이 수행할 수 있는 다양한 작업을 트리거하는 의도를 만든 다음 해당 작업을 실행하는 데 필요한 일부 매개 변수를 모델링하는 엔터티를 만들어야 합니다. 예를 들어, PictureBot의 의도는 "SearchPics"일 수 있으며, 이는 검색 서비스를 트리거하여 사진을 검색하는 것이므로 검색할 내용을 파악하는 "패싯" 엔터티가 필요합니다. [여기](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/plan-your-app)에서 앱을 계획하기 위한 더 많은 예제를 볼 수 있습니다.

앱을 계획했으면 [앱을 빌드하고 학습](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/luis-get-started-create-app)시킬 수 있습니다. LUIS 애플리케이션을 만들 때 일반적으로 수행해야 하는 단계는 다음과 같습니다.
  1. [의도 추가](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/add-intents) 
  2. [발화 추가](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/add-example-utterances)
  3. [엔터티 추가](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/add-entities)
  4. [기능을 사용하여 성능 향상](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/add-features)
  5. [학습 및 테스트](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/train-test)
  6. [활성 학습 사용](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/label-suggested-utterances)
  7. [게시](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/publishapp)



### 랩: LUIS를 사용하여 응용 프로그램에 인텔리전스 추가

다음 랩에서는 PictureBot을 만듭니다. 먼저 LUIS를 사용하여 자연어 기능을 추가하는 방법을 살펴보겠습니다. LUIS를 사용하면 자연어 발화를 의도에 매핑할 수 있습니다.  우리의 응용 프로그램에는 몇 가지 의도가 있을 수 있습니다. 사진 찾기, 사진 공유, 사진 인쇄물 주문 등을 예로 들 수 있습니다.  이러한 각 항목을 요청하는 방안으로 몇 가지 예제 발화를 제공할 수 있으며, LUIS는 학습한 내용을 바탕으로 각 의도에 새 추가 발화를 매핑합니다.  

[https://www.luis.ai](https://www.luis.ai)로 이동하여 Microsoft 계정으로 로그인합니다.  이 랩의 시작 부분에서 Cognitive Services 키를 만드는 데 사용한 계정과 동일해야 합니다.  [https://www.luis.ai/applications](https://www.luis.ai/applications)의 LUIS 응용 프로그램 목록으로 리디렉션됩니다.  봇을 지원하는 새로운 LUIS 앱을 만들 것입니다.  

> 참고: [현재 페이지](https://www.luis.ai/applications)의 "새 앱" 단추 옆에 "앱 가져오기"도 있습니다.  LUIS 응용 프로그램을 만든 후 전체 앱을 JSON으로 내보내고 소스 제어에 체크 인할 수 있습니다.  이는 코드의 버전을 관리할 때 LUIS 모델의 버전을 관리할 수 있도록 권장되는 모범 사례입니다.  내보낸 LUIS 앱은 "앱 가져오기" 단추를 사용하여 다시 가져올 수 있습니다.  랩 중에 뒤처져서 쉬운 방법을 사용하려면 "앱 가져오기" 단추를 클릭하여 [LUIS 모델](./resources/code/LUIS/PictureBotLuisModel.json)을 가져올 수 있습니다.  

[https://www.luis.ai/applications](https://www.luis.ai/applications)에서 "새 앱" 단추를 클릭합니다.  이름을 지정하고(예로 "PictureBotLuisModel"을 선택함) 문화권을 "영어"로 설정합니다.  선택적으로 설명을 제공할 수 있습니다.  드롭다운을 클릭하여 사용할 끝점 키를 선택하고 워크샵 시작 시 Azure Portal에서 만든 LUIS 키가 있는 경우 해당 키를 선택합니다. 앱을 게시할 때까지 이 옵션이 나타나지 않을 수 있으므로 표시되지 않더라도 걱정하지 마십시오.  그런 다음 "만들기"를 클릭합니다.  

![LUIS 새 앱](./resources/assets/LuisNewApp.jpg) 

새 앱의 대시보드로 이동됩니다.  앱 ID가 표시됩니다. 나중에 **LUIS 앱 ID**로 사용하도록 기록해 둡니다.  그런 다음 "의도 만들기"를 클릭합니다.  

![LUIS 대시보드](./resources/assets/LuisDashboard.jpg) 

봇이 다음과 같은 작업을 수행할 수 있도록 하려고 합니다.
+ 사진 검색/찾기
+ 소셜 미디어에 사진 공유
+ 사진 인쇄물 주문
+ 사용자에게 인사(나중에 볼 수 있듯이 다른 방법으로도 가능)

이러한 각 작업을 요청하는 사용자의 의도를 만들어 보겠습니다.  "의도 추가" 단추를 클릭합니다.  

첫 번째 의도의 이름을 "Greeting"으로 지정하고 "저장"을 클릭합니다.  그런 다음 사용자가 봇과 인사할 때 말할 수 있는 몇 가지 예를 제시하고 매번 "Enter" 키를 누릅니다.  몇 개의 발화를 입력한 후에 "저장"을 클릭합니다.  

![LUIS Greeting 의도](./resources/assets/LuisGreetingIntent.jpg) 

이제 엔터티를 만드는 방법을 살펴보겠습니다.  사용자가 사진을 검색하도록 요청하는 경우 원하는 내용을 지정할 수 있습니다.  엔터티에 이를 캡처해 보겠습니다.  

왼쪽 열에서 "엔터티"를 클릭한 다음 "사용자 지정 엔터티 추가"를 클릭합니다.  엔터티 이름을 "패싯"으로 지정하고 엔터티 유형을 "단순"으로 지정합니다.  그런 다음 "저장"을 클릭합니다.  

![패싯 엔터티 추가](./resources/assets/AddFacetEntity.jpg) 

이제 왼쪽 사이드바에서 "의도"를 클릭한 다음 노란색 "의도 추가" 단추를 클릭합니다.  "SearchPics"의 의도 이름을 지정한 다음 "저장"을 클릭합니다.  

이제 일부 샘플 발화(사용자가 봇과 이야기할 때 말할 수 있는 단어/구/문장)를 추가해 보겠습니다.  사람들은 여러 가지 방식으로 사진을 검색할 수 있습니다.  아래의 발화들을 자유롭게 사용하고 봇에게 사진을 검색하도록 요청하는 문구를 자체적으로 더 추가해 보십시오. 

+ 야외 사진을 찾아 주세요.
+ 기차 사진이 있나요?
+ 음식 사진을 찾아 주세요.
+ 6개월 된 남자 아이의 사진을 검색해 주세요.
+ 20살 된 여자의 사진을 제공해 주세요.
+ 해변 사진을 보여 주세요.
+ 개 사진을 찾고 있어요.
+ 실내에 있는 여성의 사진을 검색해 주세요.
+ 행복해 보이는 소녀들의 사진을 보여 주세요.
+ 슬픈 소녀들의 사진을 보고 싶어요.
+ 행복한 아기 사진을 보여 주세요.

이제 LUIS에게 "패싯" 엔터티로 검색 항목을 선택하는 방법을 가르쳐야 합니다.  단어를 마우스로 가리키고 클릭한 다음(또는 단어 그룹을 끌어서 선택) "패싯" 엔터티를 선택합니다.  

![엔터티 레이블 지정](./resources/assets/LabellingEntity.jpg) 

다음과 같은 발화 목록이

![패싯 엔터티 추가](./resources/assets/SearchPicsIntentBefore.jpg) 

...패싯 레이블이 지정되면 이와 같이 될 수 있습니다.  

![패싯 엔터티 추가](./resources/assets/SearchPicsIntentAfter.jpg) 

완료되면 "저장"을 클릭하는 것을 잊지 마십시오!  

마지막으로 왼쪽 사이드바에서 "의도"를 클릭하고 다음 두 가지 의도를 추가합니다.
+ 한 의도의 이름을 **"SharePic"**으로 지정합니다.  이는 "이 사진을 공유해 주세요", "트윗해 줄래요?" 또는 "트위터에 게시해 주세요"와 같은 발화로 식별될 수 있습니다.  
+ **"OrderPic"**이라는 다른 의도를 만듭니다.  이는 "이 사진을 인쇄해 주세요", "인쇄물을 주문하고 싶습니다", "그 중 8x10을 얻을 수 있을까요?", "지갑을 주문해 주세요"와 같은 발화로 전달될 수 있습니다.  
발화를 선택할 때 질문, 지시, "~하고 싶어요..." 형식을 조합하여 사용하는 것이 유용할 수 있습니다.  

"None"이라는 의도도 있습니다.  의도에 매핑되지 않는 임의의 발화는 "None"에 매핑될 수 있습니다.  "땅콩 버터와 젤리를 좋아하나요?"와 같은 몇 가지 발화로 시작해도 좋습니다.

이제 모델을 학습시킬 수 있습니다.  왼쪽 사이드바에서 "학습 및 테스트"를 클릭합니다.  그런 다음 학습 단추를 클릭합니다.  제공한 학습 데이터와 함께 발화--> 의도 매핑을 수행하는 모델이 빌드됩니다.  

그런 다음 왼쪽 사이드바에서 "앱 게시"를 클릭합니다.  아직 설정하지 않은 경우 앞에서 설정한 끝점 키를 선택하거나 링크를 따라 Azure 계정에 새 키를 만듭니다.  끝점 슬롯을 "프로덕션"으로 유지할 수 있습니다.  그런 다음 "게시"를 클릭합니다.  

![LUIS 앱 게시](./resources/assets/PublishLuisApp.jpg) 

게시하면 LUIS 모델을 호출하는 끝점이 만들어지고  URL이 표시됩니다.  

왼쪽 사이드바에서 "학습 및 테스트"를 클릭합니다.  "게시된 모델 사용" 확인란을 선택하여 모델을 직접 호출하는 대신 게시된 끝점을 통해 호출이 이루어지도록 합니다.  몇 가지 발화를 입력하고 반환된 의도를 확인합니다.  

![LUIS 테스트](./resources/assets/TestLuis.jpg) 

**일찍 끝났다면 다음과 같은 추가 크레딧 작업을 시도해 보십시오.**


"SearchPics" 의도가 활용할 수 있는 추가 엔터티를 만듭니다. 예를 들어 이 앱은 연령을 파악하므로 연령에 미리 빌드된 엔터티를 만들어 봅니다. 

"목록" 엔터티 유형의 사용자 지정 엔터티를 사용하여 감정과 성별을 캡처합니다. 아래 감정의 사례를 참조하십시오. 

![목록을 사용한 사용자 지정 감정 엔터티](./resources/assets/CustomEmotionEntityWithList.jpg) 

> **참고** 엔터티 또는 기능을 더 추가하는 경우 `의도>발화`로 이동하여 추가한 엔터티와 함께 더 많은 발화를 확인/추가할 뿐 아니라 모델을 다시 학습시키고 게시해야 합니다.

## 봇 빌드

Bot Framework를 사용해본 경험이 있을 줄로 생각하는데, 그렇다면 잘 된 일이고 그렇지 않더라도 너무 염려할 필요 없습니다. 이 섹션에서 많은 것을 배울 수 있습니다. [이 Microsoft Virtual Academy 과정](https://mva.microsoft.com/ko-kr/training-courses/creating-bots-in-the-microsoft-bot-framework-using-c-17590#!)을 이수하고 [설명서](https://docs.microsoft.com/ko-kr/bot-framework/)를 살펴보시기 바랍니다.

### 랩: 봇 개발 준비

우리는 C# SDK를 사용하여 봇을 개발할 것입니다.  시작하려면 다음 두 가지가 필요합니다.
1. [여기](https://aka.ms/bf-bc-vstemplate)에서 Bot Framework 프로젝트 템플릿을 다운로드합니다.  이 파일은 "Bot Application.zip"이라고 하며 \Documents\Visual Studio 2019\Templates\ProjectTemplates\Visual C#\ 디렉터리에 저장해야 합니다.  압축된 전체 파일을 이 디렉터리에 놓으면 되며, 압축을 풀 필요가 없습니다.  
2. 로컬에서 봇을 테스트하도록 [여기](https://emulator.botframework.com/)에서 Bot Framework Emulator를 다운로드합니다.  이 에뮬레이터는 `c:\Users\`_your-username_`\AppData\Local\botframework\app-3.5.27\botframework-emulator.exe`에 설치됩니다. 

### 랩: 단순한 봇 만들기 및 실행

Visual Studio에서 파일 --> 새 프로젝트로 이동하여 "PictureBot"이라는 봇 응용 프로그램을 만듭니다.  

![새로운 봇 응용 프로그램](./resources/assets/NewBotApplication.jpg) 

메시지를 반복하고 문자 길이를 알려주는 에코 봇인 샘플 봇 코드를 살펴봅니다.  특히 다음 사항을 **참고**하십시오.
+ App_Start 아래의 **WebApiConfig.cs**에서 경로 템플릿은 api/{controller}/{id}(id는 선택 사항)입니다.  이 때문에 봇의 끝점을 호출할 때 항상 끝에 api/messages를 추가합니다.  
+ Controllers 아래의 **MessagesController.cs**는 봇으로의 진입점입니다. 봇은 다양한 활동 유형에 응답할 수 있으며 메시지를 보내면 RootDialog가 호출됩니다.  
+ Dialogs 아래의 **RootDialog.cs**에서 "StartAsync"는 사용자의 메시지를 기다리는 진입점이며, "MessageReceivedAsync"는 받은 메시지를 처리한 후 추가 메시지를 기다리는 메서드입니다.  "context.PostAsync"를 사용하여 봇에서 사용자에게 메시지를 다시 보낼 수 있습니다.  

솔루션을 마우스 오른쪽 단추로 클릭하고 **솔루션용 NuGet 패키지 관리**를 선택합니다. 설치됨 아래에서 Microsoft.Bot.Builder를 검색하고 최신 버전으로 업데이트합니다.

F5 키를 클릭하여 샘플 코드를 실행합니다.  적절한 종속 항목을 다운로드하는 것은 NuGet을 통해 처리됩니다.  

코드는 기본 웹 브라우저를 사용하여 http://localhost:3979/와 유사한 URL에서 시작됩니다.  

> 참고: 왜 이 포트 번호를 사용할까요?  이는 프로젝트 속성에서 설정됩니다.  솔루션 탐색기에서 "속성"을 두 번 클릭하고 "웹" 탭을 선택합니다.  프로젝트 URL은 "서버" 섹션에 설정되어 있습니다.  

![봇 프로젝트 URL](./resources/assets/BotProjectUrl.jpg) 

프로젝트가 여전히 실행 중인지 확인하고(프로젝트 속성을 보기 위해 중지한 경우 F5 키를 다시 누름) Bot Framework Emulator를 시작합니다.  방금 설치한 경우 아직 인덱싱되지 않아 로컬 컴퓨터에서 검색에 표시되지 않을 수 있습니다. c:\User\User\AppData\Local\botframework\app-3.5.27\botframework-emulator.exe에 설치된다는 점을 기억하십시오.  봇 Url이 위에서 코드가 시작된 포트 번호와 일치하고 끝에 api/messages가 추가되어 있어야 합니다.  이제 봇과 대화할 수 있습니다.  

![Bot Emulator](./resources/assets/BotEmulator.png) 

### 랩: LUIS를 사용하도록 봇 업데이트

이제 LUIS를 사용하도록 봇을 업데이트하려 합니다.  [LuisDialog 클래스](https://docs.botframework.com/ko-kr/csharp/builder/sdkreference/d8/df9/class_microsoft_1_1_bot_1_1_builder_1_1_dialogs_1_1_luis_dialog.html)를 사용하여 이 작업을 수행할 수 있습니다.  

**RootDialog.cs** 파일에서 다음 네임스페이스에 참조를 추가합니다.

```csharp

using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

```

다음으로, RootDialog 클래스가 IDialog<object>가 아닌 LuisDialog<object>에서 파생되도록 변경합니다.  그런 다음 LUIS 앱 ID 및 LUIS 키를 사용하여 이 클래스에 LuisModel 특성을 지정합니다.  (힌트: LUIS 앱 ID에는 하이픈이 있으며 LUIS 키에는 없습니다.  이러한 값을 찾을 수 없는 경우 http://luis.ai로 돌아가서  애플리케이션을 클릭하면 앱 ID가 대시보드 페이지와 URL에 바로 표시됩니다.  그런 다음 [상단 사이드바에서 "내 키"](https://www.luis.ai/home/keys)를 클릭하여 목록에서 끝점 키를 찾습니다.  

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

이제 코드를 실행해 보겠습니다.  F5 키를 눌러 Visual Studio에서 실행하고 Bot Framework Emulator에서 새로운 대화를 시작합니다.  봇과 채팅을 시도하고 예상 응답을 받을 수 있는지 확인합니다.  예기치 않은 결과가 발생하면 이를 기록하고 이후에 LUIS를 수정합니다.  

![봇 테스트 LUIS](./resources/assets/BotTestLuis.jpg) 

위의 스크린샷에서 봇에 "인쇄물 주문"이라고 말했을 때 다른 응답을 받을 것으로 예상했습니다. "OrderPic" 의도 대신 "SearchPics" 의도에 매핑된 것 같습니다.  http://luis.ai로 돌아가서 LUIS 모델을 업데이트할 수 있습니다.  해당 응용 프로그램을 클릭한 다음 왼쪽 사이드바에서 "의도"를 클릭합니다.  수동으로 새 발화로 이를 추가하거나 LUIS의 "발화 제안" 기능을 활용하여 모델을 개선할 수 있습니다.  "SearchPics" 의도(또는 발화 레이블이 잘못 지정된 의도)를 클릭한 다음 "발화 제안"을 클릭합니다.  레이블이 잘못 지정된 발화의 확인란을 클릭한 다음 "의도 재할당"을 클릭하고 올바른 의도를 선택합니다.  

![LUIS 의도 재할당](./resources/assets/LuisReassignIntent.jpg) 

봇에 이러한 변경 내용을 적용하려면 LUIS 모델을 다시 학습시키고 다시 게시해야 합니다.  왼쪽 사이드바에서 "앱 게시"를 클릭하고 "학습" 단추를 클릭한 다음 하단의 "게시" 단추를 클릭합니다.  그런 다음 에뮬레이터에서 봇으로 돌아가 다시 시도할 수 있습니다.  

> 참고: 발화 제안 기능은 매우 강력합니다.  LUIS는 표시할 발화에 대해 스마트한 결정을 내립니다.  개선 가능성이 가장 높아, 참여하는 사람이 수동으로 레이블을 지정할 수 있는 발화를 선택합니다.  예를 들어 주어진 발화가 47% 신뢰 수준으로 Intent1에 매핑되고 48% 신뢰 수준으로 Intent2에 매핑될 것으로 LUIS 모델이 예측한 경우, 후자가 사람에게 표시하여 수동으로 매핑할 강력한 후보가 됩니다. 이 모델에서는 두 의도 간 차이가 거의 없기 때문입니다.  

이제 LUIS 모델을 사용하여 사용자의 의도를 파악할 수 있으므로 Azure Cognitive Search를 통합하여 사진을 찾아보겠습니다.  

### 랩: Azure Cognitive Search를 사용하도록 봇 구성 

먼저 Azure Cognitive Search 인덱스에 연결하기 위해 관련 정보를 봇에 제공해야 합니다.  연결 정보를 저장하는 가장 좋은 위치는 구성 파일입니다.  

Web.config를 열고 appSettings 섹션에서 다음을 추가합니다.

```xml    
    <!-- Azure Cognitive Search Settings -->
    <add key="SearchDialogsServiceName" value="" />
    <add key="SearchDialogsServiceKey" value="" />
    <add key="SearchDialogsIndexName" value="images" />
```

SearchDialogsServiceName의 값을 앞에서 만든 Azure Cognitive Search 서비스의 이름으로 설정합니다.  필요한 경우 [Azure Portal](https://portal.azure.com)로 돌아가서 이를 확인합니다.  

SearchDialogsServiceKey의 값을 이 서비스의 키로 설정합니다.  이 값은 [Azure Portal](https://portal.azure.com)의 Azure Cognitive Search 키 섹션에서 찾을 수 있습니다.  아래 스크린샷에서 SearchDialogsServiceName은 "aiimmersionsearch"이고 SearchDialogsServiceKey는 "375..."입니다.  

![Azure Cognitive Search 설정](./resources/assets/AzureSearchSettings.jpg) 

### 랩: Azure Cognitive Search를 사용하도록 봇 업데이트

이제 Azure Cognitive Search를 호출하도록 봇을 업데이트하겠습니다.  먼저 도구-->NuGet Package Manager-->솔루션용 NuGet 패키지 관리를 선택합니다.  검색 상자에 "Microsoft.Azure.Search"를 입력합니다.  해당 라이브러리를 선택하고 프로젝트를 나타내는 확인란을 선택한 후 설치합니다.  다른 종속성 항목도 설치할 수 있습니다. 설치된 패키지에서 "Newtonsoft.Json" 패키지도 업데이트해야 할 수 있습니다.

![Azure Cognitive Search NuGet](./resources/assets/AzureSearchNuGet.jpg) 

Visual Studio의 솔루션 탐색기에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 추가-->새 폴더를 선택합니다.  "Models"라는 폴더를 만듭니다.  "Models" 폴더를 마우스 오른쪽 단추로 클릭하고 추가-->기존 항목을 선택합니다.  이 작업을 두 번 수행하여 Models 폴더 아래에 다음 두 파일을 추가합니다. 필요한 경우 네임스페이스를 조정하십시오.
1. [ImageMapper.cs](./resources/code/Models/ImageMapper.cs)
2. [SearchHit.cs](./resources/code/Models/SearchHit.cs)

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

### 랩: 정규식 및 Scorable Groups

봇을 개선하기 위해 할 수 있는 일이 여러 가지 있습니다. 우선, 봇이 사용자로부터 상당히 자주 수신하는 간단한 "안녕하세요" 인사말을 위해 LUIS를 호출하는 것은 좋지 않습니다.  단순한 정규식이 이와 일치할 수 있으며 네트워크 대기 시간으로 인한 시간과 LUIS 서비스를 호출하는 데 드는 비용을 줄여 줍니다.  

또한 봇의 복잡성이 증가하는 동시에, 사용자의 입력을 받고 다양한 서비스를 사용하여 해석하기 때문에 이 흐름을 관리하는 프로세스가 필요합니다.  예를 들어 정규식을 먼저 시도하고 일치하지 않을 경우 LUIS를 호출합니다. 그리고 이후에는 [QnA Maker](http://qnamaker.ai) 및 Azure Cognitive Search와 같은 다른 서비스를 사용해 볼 수 있습니다.  이를 관리하는 좋은 방법은 [ScorableGroups](https://blog.botframework.com/2017/07/06/Scorables/)를 사용하는 것입니다.  ScorableGroups는 이러한 서비스 호출에 순서를 설정하는 특성을 제공합니다.  코드에서 먼저 정규식과 일치시킨 다음, 발화 해석을 위해 LUIS를 호출하고, 마지막으로 일반적인 "무슨 말씀이신지 잘 모르겠습니다." 같은 응답을 제시하는 순서를 설정해 보겠습니다.    

ScorableGroups를 사용하려면 RootDialog가 LuisDialog 대신 DispatchDialog에서 상속해야 합니다(하지만 클래스에서 LuisModel 특성을 여전히 가질 수 있음).  또한 Microsoft.Bot.Builder.Scorables(및 기타)에 대한 참조가 필요합니다.  따라서 RootDialog.cs 파일에 다음을 추가합니다.

```csharp

using Microsoft.Bot.Builder.Scorables;
using System.Collections.Generic;

```

그리고 클래스 파생을 다음으로 변경합니다.

```csharp

    public class RootDialog : DispatchDialog

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

이제 정규식이 일치하지 않을 경우 두 번째 시도로 Scorable Group 1에서 LUIS를 호출합니다.  

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

Visual Studio의 솔루션 탐색기에서 봇 애플리케이션 프로젝트를 마우스 오른쪽 단추로 클릭하고 "게시"를 선택합니다.  이렇게 하면 Azure에 봇을 게시하는 과정을 안내하는 마법사가 시작됩니다.  

게시 대상을 "Microsoft Azure App Service"로 선택합니다.  

![Azure App Service에 봇 게시](./resources/assets/PublishBotAzureAppService.jpg) 

App Service 화면에서 적절한 구독을 선택하고 "새로 만들기"를 클릭합니다. 그런 다음 API 앱 이름, 구독, 지금까지 사용한 것과 동일한 리소스 그룹 및 App Service 계획을 입력합니다.  

![App Service 만들기](./resources/assets/CreateAppService.jpg) 

마지막으로 웹 배포 설정이 표시되고 "게시"를 클릭할 수 있습니다.  Visual Studio의 출력 창에 배포 프로세스가 표시되고  http://testpicturebot.azurewebsites.net/ 같은 URL에 봇이 호스팅됩니다. 여기서 "testpicturebot"은 App Service API 앱 이름입니다.  

### 랩: Bot Connector에 봇 등록

이제 웹 브라우저에서 [http://dev.botframework.com](http://dev.botframework.com)으로 이동합니다.  [봇 등록](https://dev.botframework.com/bots/new)을 클릭하고 봇의 이름, 핸들 및 설명을 입력합니다.  메시지 끝점은 끝에 "api/messages"가 추가된 Azure 웹 사이트 URL(예: https://testpicturebot.azurewebsites.net/api/messages)입니다.  

![봇 등록](./resources/assets/BotRegistration.jpg) 

그런 다음 Microsoft 앱 ID 와 암호를 만드는 단추를 클릭합니다.  Web.config에서 필요한 봇 앱 ID 및 암호입니다.  Bot 앱 이름, 앱 ID 및 앱 암호를 안전한 장소에 저장하십시오!  암호에 대해 "확인"을 클릭한 후에는 다시 돌아갈 방법이 없습니다.  그런 다음 "완료하고 Bot Framework로 돌아가기"를 클릭합니다.  

![봇 앱 이름, ID 및 암호 생성](./resources/assets/BotGenerateAppInfo.jpg) 

봇 등록 페이지에 앱 ID가 자동으로 채워져 있을 것입니다. 봇에서 로깅하기 위해 선택적으로 AppInsights 계측 키를 추가할 수 있습니다. 서비스 약관에 동의하는 경우 확인란을 선택하고 "등록"을 클릭합니다.  

그러면 https://dev.botframework.com/bots?id=TestPictureBot과 비슷하지만 실제 봇 이름을 포함한 URL의 봇 대시보드 페이지가 표시됩니다. 여기서는 다양한 채널을 사용하도록 설정할 수 있습니다.  Skype와 웹 채팅의 두 채널이 자동으로 사용하도록 설정되어 있습니다.  

마지막으로, 등록 정보를 사용하여 봇을 업데이트해야 합니다.  Visual Studio로 돌아가 Web.config를 엽니다.  BotId를 앱 이름으로, MicrosoftAppId를 앱 ID로, MicrosoftAppPassword를 봇 등록 사이트에서 얻은 앱 암호로 업데이트합니다.  

```xml

    <add key="BotId" value="TestPictureBot" />
    <add key="MicrosoftAppId" value="95b76ae6-8643-4d94-b8a1-916d9f753ab0" />
    <add key="MicrosoftAppPassword" value="kC200000000000000000000" />

```

프로젝트를 다시 빌드한 다음, 솔루션 탐색기에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 다시 "게시"를 선택합니다.  마지막 설정이 기억되므로 "게시"를 누르기만 하면 됩니다. 

> 오류가 발생하여 MicrosoftAppPassword를 입력하라는 메시지가 표시됩니까? XML이므로 키에 "&", "<", ">", "'" 또는 '"'가 포함된 경우 이러한 기호를 다음과 같은 [이스케이프 기능](https://en.wikipedia.org/wiki/XML#Characters_and_escaping)으로 바꿔야 합니다. "&amp;", "&lt;", "&gt;", "&apos;", "&quot;". 

이제 봇의 대시보드(예: https://dev.botframework.com/bots?id=TestPictureBot)로 돌아갈 수 있습니다.  채팅 창에서 봇과 대화해 보십시오.  웹 채팅에서는 캐러셀이 에뮬레이터에서와 다르게 보일 수 있습니다.  다양한 채널에서 다양한 컨트롤의 사용자 경험을 확인할 수 있는 Channel Inspector라는 훌륭한 도구가 https://docs.botframework.com/ko-kr/channel-inspector/channels/Skype/#navtitle에 있습니다.  
봇의 대시보드에서 다른 채널을 추가하고 Skype, Facebook Messenger 또는 Slack에서 봇을 사용해 볼 수 있습니다.  봇 대시보드의 채널 이름 오른쪽에 있는 "추가" 단추를 클릭하고 지침을 따르기만 하면 됩니다.

**일찍 끝났다면 이 추가 크레딧 랩을 사용해 보십시오.**

고급 Azure Cognitive Search 쿼리를 사용하여 실험해 보십시오. "행복한"을 "행복"(Cognitive Services에서 반환된 감정)에 매핑하여 _"행복한 사람 찾기"_ 같은 엔터티를 인식하도록 LUIS 모델을 확대하고 [용어 부스팅](https://docs.microsoft.com/ko-kr/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_termboost)을 사용하여 부스팅된 쿼리로 전환함으로써 용어 부스팅을 추가해 보십시오. 

## 랩 완료

이 랩에서는 Microsoft Bot Framework, Azure Cognitive Search 및 여러 가지 Cognitive Services를 사용하여 엔드투엔드 지능형 봇을 만드는 방법을 살펴봤습니다.

다음과 같은 내용을 다루었습니다.
- 응용 프로그램에 지능형 서비스를 통합하는 방법
- 애플리케이션 내에서 긍정적인 검색 경험을 제공하도록 Azure Cognitive Search 기능을 구현하는 방법
- 데이터를 확장하여 전체 텍스트, 언어 인식 검색을 사용하도록 Azure Cognitive Search 서비스를 구성하는 방법
- 봇이 효과적으로 통신할 수 있도록 LUIS 모델 빌드, 학습 및 게시하는 방법
- LUIS 및 Azure Cognitive Search를 활용하는 Microsoft Bot Framework를 사용하여 지능형 봇을 빌드하는 방법
- .NET 응용 프로그램에서 다양한 Cognitive Services API(특히 Computer Vision, Face, Emotion 및 LUIS)를 호출하는 방법

향후 프로젝트/학습을 위한 리소스:
- [Azure Bot Service 설명서](https://docs.microsoft.com/ko-kr/bot-framework/)
- [Azure Cognitive Search 설명서](https://docs.microsoft.com/ko-kr/azure/search/search-what-is-azure-search)
- [Azure Bot Builder 샘플](https://github.com/Microsoft/BotBuilder-Samples)
- [Azure Cognitive Search 샘플](https://github.com/Azure-Samples/search-dotnet-getting-started)
- [LUIS 설명서](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/Home)
- [LUIS 샘플](https://github.com/Microsoft/BotBuilder-Samples/blob/master/CSharp/intelligence-LUIS/README.md)