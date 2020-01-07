## 2_Luis:
예상 시간: 20-30분

## LUIS

먼저, [Microsoft의 LUIS(Language Understand Intelligent Service)에 대해 알아보겠습니다](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/Home).

이제 LUIS가 무엇인지 알았으므로 LUIS 앱을 계획할 수 있습니다. 검색 결과를 기반으로 이미지를 반환하고 사용자가 공유하거나 주문할 수 있도록 하는 봇을 만들려고 합니다. 봇이 수행할 수 있는 다양한 작업을 트리거하는 의도를 만든 다음 해당 작업을 실행하는 데 필요한 일부 매개 변수를 모델링하는 엔터티를 만들어야 합니다. 예를 들어, PictureBot의 의도는 "SearchPics"일 수 있으며, 이는 검색 서비스를 트리거하여 사진을 검색하는 것이므로 검색할 내용을 파악하는 "패싯" 엔터티가 필요합니다. [여기](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/plan-your-app)에서 앱을 계획하기 위한 더 많은 예제를 볼 수 있습니다.

앱을 계획했으면 [앱을 빌드하고 학습](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/luis-get-started-create-app)시킬 수 있습니다. LUIS 응용 프로그램을 만들 때 일반적으로 수행해야 하는 단계는 다음과 같습니다.
  1. [의도 추가](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/add-intents) 
  2. [발화 추가](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/add-example-utterances)
  3. [엔터티 추가](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/add-entities)
  4. [기능을 사용하여 성능 향상](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/add-features)
  5. [학습 및 테스트](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/train-test)
  6. [활성 학습 사용](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/label-suggested-utterances)
  7. [게시](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/publishapp)


### 랩: 포털에서 LUIS 서비스 만들기

포털에서 **리소스 만들기** 를 누른 후 검색 상자에 **LUIS** 를 입력하고 **Language Understanding Intelligent Service** 를 선택합니다.

다음으로, 만들 API 끝점에 대한 몇 가지 세부 정보를 작성하고 원하는 API와 끝점을 사용할 위치 그리고 요금제를 선택합니다. 이 랩에는 무료 계층으로 충분합니다. LUIS는 향후 Cognitive Services 제품을 개선하기 위해 이미지를 Microsoft에 내부적으로 안전하게 저장하므로 이에 대한 확인란을 선택해야 합니다.

새 API 구독을 만든 후에는 블레이드의 해당 섹션에서 키를 가져와 키 목록에 추가할 수 있습니다.

![Cognitive API 키](./resources/assets/cognitive-keys.PNG)

### 랩: LUIS를 사용하여 응용 프로그램에 인텔리전스 추가

다음 랩에서는 PictureBot을 만듭니다. 먼저 LUIS를 사용하여 자연어 기능을 추가하는 방법을 살펴보겠습니다. LUIS를 사용하면 자연어 발화(사용자가 봇과 이야기할 때 말할 수 있는 단어/구/문장)를 의도(사용자가 수행하려는 작업)에 매핑할 수 있습니다.  우리의 응용 프로그램에는 몇 가지 의도가 있을 수 있습니다. 사진 찾기, 사진 공유, 사진 인쇄물 주문 등을 예로 들 수 있습니다.  이러한 각 항목을 요청하는 방안으로 몇 가지 예제 발화를 제공할 수 있으며, LUIS는 학습한 내용을 바탕으로 각 의도에 새 추가 발화를 매핑합니다.  

[https://www.luis.ai](https://www.luis.ai)로 이동하여 Microsoft 계정으로 로그인합니다.  이전 섹션에서 LUIS 키를 만드는 데 사용한 계정과 동일해야 합니다.  [https://www.luis.ai/applications](https://www.luis.ai/applications)의 LUIS 응용 프로그램 목록으로 리디렉션됩니다.  봇을 지원하는 새로운 LUIS 앱을 만들 것입니다.  

> 참고: [현재 페이지](https://www.luis.ai/applications)의 "새 앱" 단추 옆에 "앱 가져오기"도 있습니다.  LUIS 응용 프로그램을 만든 후 전체 앱을 JSON으로 내보내고 소스 제어에 체크 인할 수 있습니다.  이는 코드의 버전을 관리할 때 LUIS 모델의 버전을 관리할 수 있도록 권장되는 모범 사례입니다.  내보낸 LUIS 앱은 "앱 가져오기" 단추를 사용하여 다시 가져올 수 있습니다.  랩 중에 뒤처져서 쉬운 방법을 사용하려면 "앱 가져오기" 단추를 클릭하여 [LUIS 모델](./resources/code/LUIS/PictureBotLuisModel.json)을 가져올 수 있습니다.  

[https://www.luis.ai/applications](https://www.luis.ai/applications)에서 "새 앱" 단추를 클릭합니다.  이름을 지정하고(예로 "PictureBotLuisModel"을 선택함) 문화권을 "영어"로 설정합니다.  선택적으로 설명을 제공할 수 있습니다. 그런 다음 "만들기"를 클릭합니다.  

![LUIS 새 앱](./resources/assets/LuisNewApp.png) 

새 앱의 대시보드로 이동됩니다.  앱 ID가 표시됩니다. 나중에 **LUIS 앱 ID** 로 사용하도록 기록해 둡니다.  그런 다음 "의도 만들기"를 클릭합니다.  

![LUIS 대시보드](./resources/assets/LuisDashboard.jpg) 

봇이 다음과 같은 작업을 수행할 수 있도록 하려고 합니다.
+ 사진 검색/찾기
+ 소셜 미디어에 사진 공유
+ 사진 인쇄물 주문
+ 사용자에게 인사(나중에 볼 수 있듯이 다른 방법으로도 가능)

이러한 각 작업을 요청하는 사용자의 의도를 만들어 보겠습니다.  "의도 추가" 단추를 클릭합니다.  

첫 번째 의도의 이름을 "Greeting"으로 지정하고 "저장"을 클릭합니다.  그런 다음 사용자가 봇과 인사할 때 말할 수 있는 몇 가지 예를 제시하고 매번 "Enter" 키를 누릅니다.  몇 개의 발화를 입력한 후에 "저장"을 클릭합니다.  

![LUIS Greeting 의도](./resources/assets/LuisGreetingIntent.jpg) 

엔터티를 만드는 방법을 살펴보겠습니다.  사용자가 사진을 검색하도록 요청하는 경우 원하는 내용을 지정할 수 있습니다.  엔터티에 이를 캡처해 보겠습니다.  

왼쪽 열에서 "엔터티"를 클릭한 다음 "사용자 지정 엔터티 추가"를 클릭합니다.  엔터티 이름을 "패싯"으로 지정하고 엔터티 유형을 "단순"으로 지정합니다.  그런 다음 "저장"을 클릭합니다.  

![패싯 엔터티 추가](./resources/assets/AddFacetEntity.jpg) 

다음으로 왼쪽 사이드바에서 "의도"를 클릭한 다음 노란색 "의도 추가" 단추를 클릭합니다.  "SearchPics"의 의도 이름을 지정한 다음 "저장"을 클릭합니다.  

인사말과 마찬가지로 일부 샘플 발화(사용자가 봇과 이야기할 때 말할 수 있는 단어/구/문장)를 추가해 보겠습니다.  사람들은 여러 가지 방식으로 사진을 검색할 수 있습니다.  아래의 발화들을 자유롭게 사용하고 봇에게 사진을 검색하도록 요청하는 문구를 자체적으로 더 추가해 보십시오. 

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

몇 가지 발화를 확보했으면 LUIS에게 "패싯" 엔터티로 **검색 항목** 을 선택하는 방법을 가르쳐야 합니다. "패싯" 엔터티가 선택하는 것이 검색됩니다. 단어를 마우스로 가리키고 클릭한 다음(또는 단어 그룹을 끌어서 선택) "패싯" 엔터티를 선택합니다. 

![엔터티 레이블 지정](./resources/assets/LabellingEntity.jpg) 

다음과 같은 발화 목록이

![패싯 엔터티 추가](./resources/assets/SearchPicsIntentBefore.jpg) 

...패싯 레이블이 지정되면 이와 같이 될 수 있습니다.  

![패싯 엔터티 추가](./resources/assets/SearchPicsIntentAfter.jpg) 

완료되면 "저장"을 클릭하는 것을 잊지 마십시오!  

마지막으로 왼쪽 사이드바에서 "의도"를 클릭하고 다음 두 가지 의도를 추가합니다.
+ 한 의도의 이름을 **"SharePic"** 으로 지정합니다.  이는 "이 사진을 공유해 주세요", "트윗해 줄래요?" 또는 "트위터에 게시해 주세요"와 같은 발화로 식별될 수 있습니다.  
+ **"OrderPic"** 이라는 다른 의도를 만듭니다.  이는 "이 사진을 인쇄해 주세요", "인쇄물을 주문하고 싶습니다", "그 중 8x10을 얻을 수 있을까요?", "지갑을 주문해 주세요"와 같은 발화로 전달될 수 있습니다.  
발화를 선택할 때 질문, 지시, "~하고 싶어요..." 형식을 조합하여 사용하는 것이 유용할 수 있습니다.  

"None"이라는 의도도 있습니다.  의도에 매핑되지 않는 임의의 발화는 "None"에 매핑될 수 있습니다.  

이제 모델을 학습시킬 수 있습니다.  왼쪽 사이드바에서 "학습 및 테스트"를 클릭합니다.  그런 다음 학습 단추를 클릭합니다.  제공한 학습 데이터와 함께 발화--> 의도 매핑을 수행하는 모델이 빌드됩니다. 학습이 항상 즉각적으로 이루어지는 것은 아닙니다. 경우에 따라 큐에서 대기하고 몇 분 정도 걸릴 수 있습니다.

그런 다음 왼쪽 사이드바에서 "앱 게시"를 클릭합니다.  앱을 게시할 때 [자세한 끝점 응답 또는 Bing Spell Checker](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/PublishApp)를 사용하도록 설정하는 등 몇 가지 옵션이 있습니다. 아직 설정하지 않은 경우 앞에서 설정한 끝점 키를 선택하거나 링크를 따라 Azure 계정에서 키를 추가합니다.  끝점 슬롯을 "프로덕션"으로 유지할 수 있습니다.  그런 다음 "게시"를 클릭합니다.  



![LUIS 앱 게시](./resources/assets/PublishLuisApp.png) 

게시하면 LUIS 모델을 호출하는 끝점이 만들어지고  URL이 표시되며, 관련 내용은 이후 랩에서 설명합니다.

왼쪽 사이드바에서 "학습 및 테스트"를 클릭합니다.  "게시된 모델 사용" 확인란을 선택하여 모델을 직접 호출하는 대신 게시된 끝점을 통해 호출이 이루어지도록 합니다. 몇 가지 발화를 입력하고 반환된 의도를 확인합니다.  
>하지만 "게시된 모델 사용"에 열려있는 버그가 있으며, 이는 Chrome에서만 작동합니다. Chrome을 다운로드하여 사용해 보거나, 이를 건너뛰고 사용되도록 설정되지 않았다는 점을 기억하십시오.

![LUIS 테스트](./resources/assets/TestLuis.png) 

[브라우저에서 게시된 끝점을 테스트](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/PublishApp#test-your-published-endpoint-in-a-browser)할 수도 있습니다. URL을 복사한 다음 `{YOUR-KEY-HERE}`를 사용하려는 리소스의 키 문자열 열에 나열된 키 중 하나로 바꿉니다. 브라우저에서 이 URL을 열려면 URL 매개 변수 `&q`를 테스트 쿼리로 설정합니다. 예를 들어 URL에 `&q=Find pictures of dogs`를 추가한 다음 Enter 키를 누릅니다. HTTP 끝점의 JSON 응답이 브라우저에 표시됩니다.

**일찍 끝났다면 다음과 같은 추가 크레딧 작업을 시도해 보십시오.**


"SearchPics" 의도가 활용할 수 있는 추가 엔터티를 만듭니다. 예를 들어 이 앱은 연령을 파악하므로 연령에 미리 빌드된 엔터티를 만들어 봅니다. 

"목록" 엔터티 유형의 사용자 지정 엔터티를 사용하여 감정과 성별을 캡처합니다. 아래 감정의 예를 참조하십시오. 

![목록을 사용한 사용자 지정 감정 엔터티](./resources/assets/CustomEmotionEntityWithList.jpg) 

> **참고:** 엔터티 또는 기능을 더 추가하는 경우 **의도>발화** 로 이동하여 추가한 엔터티와 함께 더 많은 발화를 확인/추가할 뿐 아니라 모델을 다시 학습시키고 게시해야 합니다.


### [3_Bot](./3_Bot.md)로 계속 진행

[README](./0_README.md)로 돌아가기