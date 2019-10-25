## 1_AzureSearch:
예상 시간: 20-30분

## Azure Search 

[Azure Search](https://docs.microsoft.com/ko-kr/azure/search/search-what-is-azure-search)는 개발자가 인프라를 관리하거나 검색 전문가가 되지 않고도 응용 프로그램에 훌륭한 검색 환경을 통합할 수 있게 해주는 SaaS(Search-as-a-Service) 솔루션입니다.

개발자는 앱에서 더 나은 결과를 더 빠르게 얻을 수 있도록 Azure에서 PaaS 서비스를 이용하길 원합니다. 검색은 많은 유형의 응용 프로그램에서 핵심적인 요소입니다. 웹 검색 엔진의 검색 품질이 높아짐에 따라 사용자는 즉각적인 결과, 입력 시 자동 완성, 결과 내 검색어 강조 표시, 우수한 순위 지정, 맞춤법이 잘못되거나 불필요한 단어가 포함되어 있더라도 검색하려는 항목을 파악하는 기능 등을 기대합니다.

검색은 어렵고 핵심 전문 분야인 경우가 드뭅니다. 인프라 관점에서는 뛰어난 가용성, 내구성, 확장성 및 운영 능력이 필요하며, 기능적 관점에서는 순위 지정, 언어 지원 및 지리 공간적 기능이 있어야 합니다.

![검색 요구 사항의 예](./resources/assets/AzureSearch-Example.png) 

위의 예는 사용자가 검색 환경에서 기대하는 일부 구성 요소를 보여 줍니다. [Azure Search](https://docs.microsoft.com/ko-kr/azure/search/search-what-is-azure-search)는 [모니터링 및 보고](https://docs.microsoft.com/ko-kr/azure/search/search-traffic-analytics), [간단한 점수 매기기](https://docs.microsoft.com/ko-kr/rest/api/searchservice/add-scoring-profiles-to-a-search-index) 그리고 [프로토타이핑](https://docs.microsoft.com/ko-kr/azure/search/search-import-data-portal) 및 [검사](https://docs.microsoft.com/ko-kr/azure/search/search-explorer) 도구를 제공하며 이러한 사용자 환경 기능을 구현할 수 있습니다.

일반적인 워크플로우:
1. 서비스 프로비전
	- [포털](https://docs.microsoft.com/ko-kr/azure/search/search-create-service-portal) 또는 [PowerShell](https://docs.microsoft.com/ko-kr/azure/search/search-manage-powershell)을 사용하여 Azure Search 서비스를 만들거나 프로비전할 수 있습니다.
2. 인덱스 만들기
	- [인덱스](https://docs.microsoft.com/ko-kr/azure/search/search-what-is-an-index)는 데이터, 인지 "테이블"을 위한 컨테이너이며, 스키마, [CORS 옵션](https://docs.microsoft.com/ko-kr/aspnet/core/security/cors), 검색 옵션을 갖습니다. [포털](https://docs.microsoft.com/ko-kr/azure/search/search-create-index-portal)에서 또는 [앱 초기화](https://docs.microsoft.com/ko-kr/azure/search/search-create-index-dotnet) 중에 인덱스를 만들 수 있습니다. 
3. 인덱스 데이터
	- [데이터로 인덱스를 채우는](https://docs.microsoft.com/ko-kr/azure/search/search-what-is-data-import) 방법에는 두 가지가 있습니다. 첫 번째 방법은 Azure Search [REST API](https://docs.microsoft.com/ko-kr/azure/search/search-import-data-rest-api) 또는 [.NET SDK](https://docs.microsoft.com/ko-kr/azure/search/search-import-data-dotnet)를 사용하여 데이터를 인덱스에 수동으로 푸시하는 것입니다. 두 번째 방법은 [지원되는 데이터 원본](https://docs.microsoft.com/ko-kr/azure/search/search-import-data-portal)에서 인덱스를 가리키고 Azure Search가 일정에 따라 데이터를 자동으로 가져오도록 하는 것입니다.
4. 인덱스 검색
	- Azure Search에 검색 요청을 제출할 때 간단한 검색 옵션을 사용할 수 있으며 [필터링](https://docs.microsoft.com/ko-kr/azure/search/search-filters), [정렬](https://docs.microsoft.com/ko-kr/rest/api/searchservice/add-scoring-profiles-to-a-search-index), [예측](https://docs.microsoft.com/ko-kr/azure/search/search-faceted-navigation) 및 [여러 페이지에 결과 표시](https://docs.microsoft.com/ko-kr/azure/search/search-pagination-page-layout) 기능을 사용할 수 있습니다. 맞춤법 오류, 음성 및 Regex를 처리할 수 있으며 검색 및 [제안](https://docs.microsoft.com/ko-kr/rest/api/searchservice/suggesters) 작업을 위한 옵션이 있습니다. 이러한 쿼리 매개 변수를 사용하면 [전체 텍스트 검색 환경](https://docs.microsoft.com/ko-kr/azure/search/search-query-overview)을 보다 세부적으로 제어할 수 있습니다.


### 랩: Azure Search 서비스 만들기

Azure Portal 내에서 **리소스 만들기->웹 + 모바일->Azure Search** 를 클릭합니다.

이를 클릭한 후에는 필요한 필드를 몇 가지 작성해야 합니다. 이 랩에서는 "F0" 무료 계층으로 충분합니다. 구독당 하나의 무료 Azure Search 인스턴스만 가질 수 있으므로 본인 또는 구독에 포함된 다른 구성원이 이미 이 작업을 수행한 경우 "기본" 가격 책정 계층을 사용해야 합니다. 이 워크샵의 모든 랩에서 하나의 리소스 그룹을 사용합니다. `lab01.1-pcl_and_cognitive_services`를 완료한 경우 해당 리소스 그룹을 사용하면 됩니다.

![새 Azure Search 서비스 만들기](./resources/assets/AzureSearch-CreateSearchService.png)

만들기가 완료되면 새 Search Service의 패널을 엽니다.

### 랩: Azure Search 인덱스 만들기

인덱스는 Azure Search 서비스에서 사용하는 문서 및 기타 구문의 영구 저장소입니다. 인덱스는 데이터를 보유하고 검색 쿼리를 받을 수 있는 데이터베이스와 같습니다. 데이터베이스의 필드와 유사하게 검색하려는 문서의 구조를 반영하도록 매핑할 인덱스 스키마를 정의합니다. 이러한 필드에는 전체 텍스트 검색이 가능한지 또는 필터링이 가능한지 등을 알려주는 속성이 있을 수 있습니다.  프로그래밍 방식으로 [콘텐츠를 푸시](https://docs.microsoft.com/ko-kr/rest/api/searchservice/addupdate-or-delete-documents)하거나 [Azure Search Indexer](https://docs.microsoft.com/ko-kr/azure/search/search-indexer-overview)(공통 데이터 저장소 크롤링 가능)를 사용하여 Azure Search에 콘텐츠를 채울 수 있습니다.

이 랩에서는 [Cosmos DB용 Azure Search Indexer](https://docs.microsoft.com/ko-kr/azure/search/search-howto-index-documentdb)를 사용하여 Cosmos DB 컨테이너의 데이터를 크롤링합니다. 

![가져오기 마법사](./resources/assets/AzureSearch-ImportData.png) 

방금 만든 Azure Search 블레이드 내에서 **데이터 가져오기->데이터 원본->문서 DB** 를 클릭합니다.

![DocDB용 가져오기 마법사](./resources/assets/AzureSearch-DataSource.png) 

이것을 클릭한 후에 Cosmos DB 데이터 원본의 이름을 선택합니다. 이전 랩인 `lab01.1-pcl_and_cognitive_services`를 완료한 경우 데이터가 있는 Cosmos DB 계정과 해당 컨테이너 및 컬렉션을 선택합니다. 이전 랩을 완료하지 않은 경우 "또는 연결 문자열 입력"을 선택하고 `AccountEndpoint=https://timedcosmosdb.documents.azure.com:443/;AccountKey=0aRt6JVgbf9KafBxRVuDMNfAj9YoSBbmpICdJ41N5CwHcjuMcVk7jWDBcu4BxbTitLR1zteauQsnF1Tgqs1A3g==;`를 붙여 넣습니다. 둘 다 데이터베이스는 "images"여야 하고 컬렉션은 "metadata"여야 합니다.

**확인** 을 클릭합니다.

그러면 Azure Search가 Cosmos DB 컨테이너에 연결하고 몇 가지 문서를 분석하여 Azure Search 인덱스에 대한 기본 스키마를 식별합니다. 이 작업을 완료하면 응용 프로그램에 필요한 대로 필드에 속성을 설정할 수 있습니다.

>참고: "_ts" 필드가 유효한 필드 이름이 아니라는 경고가 표시될 수 있습니다. 랩에서는 무시해도 되지만 [여기](https://docs.microsoft.com/azure/search/search-indexer-field-mappings)에서 자세한 정보를 확인할 수 있습니다.

인덱스 이름을 **images** 로 업데이트

키를 **id**(각 문서를 고유하게 식별)로 업데이트

모든 필드를 **Retrievable** 로 설정(검색 시 클라이언트가 이러한 필드를 검색할 수 있음)

**Tags, NumFaces 및 Faces** 필드를 **Filterable** 로 설정(클라이언트가 이러한 값을 기반으로 결과를 필터링할 수 있음)

**NumFaces** 필드를 **Sortable** 로 설정(클라이언트가 이미지의 얼굴 수에 따라 결과를 정렬할 수 있음)

**Tags, NumFaces 및 Faces** 필드를 **Facetable** 로 설정(클라이언트가 결과를 개수별로 그룹화할 수 있음. 예를 들어 검색 결과에 "해변"이라는 태그가 있는 사진이 5개 있음)

**Caption, Tags 및 Faces** 를 **Searchable** 로 설정(클라이언트가 이러한 필드의 텍스트에 대해 전체 텍스트 검색을 수행할 수 있음)

![Azure Search 인덱스 구성](./resources/assets/AzureSearch-ConfigureIndex.png) 

이제 Azure Search 분석기를 구성합니다.  간단히 말해, 분석기는 사용자가 입력하는 검색어를 확인한 후 인덱스에서 가장 일치하는 용어를 찾는 기능으로 생각할 수 있습니다.  Azure Search에는 56개 언어를 심층적으로 이해하는 Bing 및 Office 같은 기술에 사용되는 분석기가 포함되어 있습니다.  

**분석기** 탭을 클릭하고 **영어-Microsoft** [분석기](https://docs.microsoft.com/ko-kr/azure/search/search-analyzers)를 사용하도록 **Caption, Tags 및 Faces** 필드를 설정합니다.

![언어 분석기](./resources/assets/AzureSearch-Analyzer.png) 

최종 인덱스 구성 단계에서는 [**제안기**](https://docs.microsoft.com/ko-kr/rest/api/searchservice/suggesters)를 만들어 자동 완성에 사용될 필드를 설정함으로써 사용자가 단어의 일부를 입력하면 Azure Search가 이러한 필드에서 가장 일치하는 항목을 찾을 수 있도록 합니다. 유사 일치(사용자가 단어를 잘못 입력하는 경우 비슷한 일치 항목을 기반으로 결과를 반환하는 기능)를 지원하도록 검색을 확장하는 방법과 제안기에 대한 자세한 내용은 [이 예제](https://docs.microsoft.com/ko-kr/azure/search/search-query-lucene-examples#fuzzy-search-example)에서 확인하십시오.


**제안기** 탭을 클릭하고 제안기 이름을 **sg** 로 입력한 후 추천 용어를 찾을 필드로 **태그 및 얼굴** 을 선택합니다.

![검색 제안](./resources/assets/AzureSearch-Suggester.png) 

**확인** 을 클릭하여 인덱서의 구성을 완료합니다.  인덱서가 변경 사항을 확인하는 빈도에 대한 일정을 설정할 수 있지만 이 랩에서는 한 번만 실행하겠습니다.  

**고급 옵션** 을 클릭하고 **Base-64 인코딩 키** 를 선택하여 Azure Search 키 필드에서 지원되는 문자만 ID 필드에 사용되도록 합니다.

**확인, 세 번** 을 클릭하여 Cosmos DB 데이터베이스에서 데이터를 가져오는 인덱서 작업을 시작합니다.

![인덱서 구성](./resources/assets/AzureSearch-ConfigureIndexer.png) 

***검색 인덱스 쿼리***

인덱싱이 시작되었음을 알리는 메시지가 나타납니다.  인덱스의 상태를 확인하려면 기본 Azure Search 블레이드에서 "인덱스" 옵션을 선택합니다.

이제 인덱스를 검색해 볼 수 있습니다.  

**검색 탐색기** 를 클릭하고 결과 블레이드에서 인덱스가 아직 선택되지 않은 경우 인덱스를 선택합니다.

**검색** 을 클릭하여 모든 문서를 검색합니다. 'water' 또는 다른 것을 검색해보고 **Ctrl 키를 누른 채 클릭** 하여 URL을 선택하고 확인해 보십시오. 결과가 예상한 대로 나타납니까?

![검색 탐색기](./resources/assets/AzureSearch-SearchExplorer.png)

결과 json에서 `@search.score` 다음에 숫자가 표시됩니다. 점수 매기기는 검색 결과에서 반환되는 모든 항목에 대한 검색 점수를 계산하는 것을 의미합니다. 점수는 현재 검색 작업의 컨텍스트에서 항목의 관련성을 나타내는 지표입니다. 점수가 높을수록 항목의 관련성이 높아집니다. 검색 결과에서 항목은 각 항목에 대해 계산된 검색 점수에 따라 높은 점수에서 낮은 점수 순으로 정렬됩니다.

Azure Search는 기본 점수 매기기를 사용하여 초기 점수를 계산하지만 [점수 매기기 프로필](https://docs.microsoft.com/ko-kr/rest/api/searchservice/add-scoring-profiles-to-a-search-index)을 통해 계산을 사용자 지정할 수 있습니다. 점수 매기기에 [용어 부스팅](https://docs.microsoft.com/ko-kr/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_termboost)을 사용하는 실습을 해볼 수 있도록 이 워크샵 끝부분에 추가 랩이 있습니다.

**일찍 끝났다면 이 추가 크레딧 랩을 사용해 보십시오.**

[Postman](https://www.getpostman.com/)은 Azure Search REST API 호출을 쉽게 실행할 수 있게 해주는 유용한 도구이자 훌륭한 디버깅 도구입니다.  Azure Search Explorer에서 모든 쿼리를 가져가서 Azure Search API 키와 함께 Postman 내에서 실행할 수 있습니다.

[Postman](https://www.getpostman.com/) 도구를 다운로드하여 설치합니다. 

설치한 후 Azure Search Explorer에서 쿼리를 받아 Postman에 붙여 넣은 다음 GET을 요청 유형으로 선택합니다.  

헤더를 클릭하고 다음 매개 변수를 입력합니다.

+ 콘텐츠 형식: application/json
+ api-key: [Azure Search Portal에서 "키" 섹션 아래에 API 키 입력]

보내기를 선택하면 JSON 형식의 데이터가 표시됩니다.

[이와 같은 예제](https://docs.microsoft.com/ko-kr/rest/api/searchservice/search-documents#a-namebkmkexamplesa-examples)를 사용하여 다른 검색을 수행해 보십시오.


### [2_LUIS](./2_LUIS.md)로 계속 진행

[README](./0_README.md)로 돌아가기