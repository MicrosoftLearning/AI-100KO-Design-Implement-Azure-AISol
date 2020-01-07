---
lab:
    title: '랩 1: 기술 요구 사항'
    module: '모듈 1: Azure Cognitive Services 소개'
---

# 랩 1: 기술 요구 사항

## 소개

이 랩에서는 워크샵 사례 연구를 소개하고 Microsoft Cognitive Services 제품군 내에서 도구를 빌드할 수 있도록 로컬 워크스테이션과 Azure 인스턴스에 도구를 설정합니다.

# 워크샵 사례 연구

## 시나리오

자전거와 자전거 장비를 판매하는 Adventure Works LLC가 여러분의 새 고객으로 배정되었습니다.

Adventure Works Cycles는 대규모 다국적 제조 회사로, 자전거 및 자전거 부품을 제조하고 인터넷 채널과 재판매인 유통망을 통해 북미, 유럽 및 아시아 상업 시장에 판매하고 있습니다. 워싱턴 주 커클랜드에 290명의 직원이 근무하고 있으며, 시장 기반에 걸쳐 여러 지역 영업 팀이 있습니다.

성공적인 회계 연도를 보낸 Adventure Works는 기존 고객을 대상으로 추가 판매를 유도하여 수익을 높이고자 합니다. 작년에 마케팅 부서는 다양한 무역 박람회 및 레이싱 이벤트에서 Adventure Works 제품에 대한 제품 평가 데이터를 수동으로 수집하는 이니셔티브를 시작했습니다. 이 데이터는 현재 Microsoft Excel 파일에 보관되어 있습니다.

이들은 수집된 정보에서 평가 결과의 수동 분석을 통해 기존 고객 기반에 추가 제품을 판매할 수 있다는 점을 확인할 수 있었습니다. 마케팅 팀은 Adventure Works 웹 사이트에서 온라인 버전의 설문조사를 만들어 이를 확장하고자 했으며 영업 부서가 이러한 활동에 대해 듣고 온라인 설문조사를 마련했습니다. 그런데 이 플랫폼의 설문조사는 매우 열악하게 구현되었고 필드가 Excel 파일에 수집된 데이터와 일치하지 않습니다.

마케팅 부서는 평가 데이터를 활용하여 상향 판매 기회로 다른 제품에 대한 추천을 제공할 수 있다는 확신을 가지고 있습니다. 하지만 웹 사이트에 설문조사를 구현하는 것은 필요한 데이터를 수집하기에 안 좋은 채널로 나타남에 따라 이 상황을 개선하는 방안에 대한 조언을 모색하고 있습니다. 또한 이 팀은 고객이 늘어남에 따라 수동으로 분석을 수행하는 것이 너무 어렵다는 점을 인지하고 있습니다.

 Adventure Works는 다양한 언어를 구사하는 고객의 대량 문의를 처리하도록 원활하게 확장하는 것을 목표로 합니다. 또한 확장 가능한 고객 서비스 플랫폼을 만들어 고객의 요구, 문제 및 제품 평가에 대한 더 많은 인사이트를 얻고자 합니다.

아울러 고객 서비스 부서는 고객 지원 기능 중 일부를 대화형 플랫폼으로 오프로드하려고 합니다. 직원의 업무량을 줄이고 일반적인 질문에 신속하게 답변하여 고객 만족도를 높이는 것이 목적입니다.

### 솔루션

대화형 플랫폼은 다음과 같은 기능으로 구성된 봇으로 구상됩니다.

- 고객 언어 감지(현재 영어 응답만 지원)

- 사용자의 감정 모니터링

- 이미지 업로드 허용 및 개체가 자전거인지 확인

- 챗봇에 FAQ 통합

- 봇 채팅에서 입력한 텍스트를 기반으로 사용자의 의도 확인

- 나중에 검토하기 위해 채팅 봇 세션 로깅

로컬 드라이브에서 사진을 수집한 다음 [Computer Vision API](https://www.microsoft.com/cognitive-services/ko-kr/computer-vision-api)를 호출하여 이미지를 분석하고 태그 및 설명을 얻을 수 있는 단순한 C# 애플리케이션을 빌드합니다.

랩 전반에 걸쳐 이 랩에서 계속하여 고객의 텍스트 문의와 상호 작용하는 [Bot Framework](https://dev.botframework.com/) 봇을 빌드하는 방법을 보여 드리겠습니다. 그런 다음 [QnA Maker](https://docs.microsoft.com/ko-kr/azure/cognitive-services/qnamaker/overview/overview)를 통해 기존 기술 자료 및 FAQ를 Bot Framework에 통합하기 위한 간단한 솔루션을 시연합니다. 마지막으로, 쿼리에서 의도를 자동으로 도출하고 이를 바탕으로 고객의 텍스트 요청에 지능적으로 응답하도록 [LUIS](https://www.microsoft.com/cognitive-services/ko-kr/language-understanding-intelligent-service-luis)를 사용하여 이 봇을 확장하겠습니다.

고객이 봇과 상호 작용하는 동안 다른 데이터에 액세스할 수 있도록 Bing Search를 사용하기 위한 컨텍스트만 제공하며 이러한 시나리오를 랩에서 구현하지는 않습니다. 참가자는 [Bing Web Search](https://azure.microsoft.com/ko-kr/services/cognitive-services/directory/search/) 서비스에 대한 자세한 내용을 확인할 수 있습니다.

이 랩의 범위를 벗어나지만 이 아키텍처는 Azure의 데이터 솔루션을 통합하고, [Blob Storage]((https://docs.microsoft.com/ko-kr/azure/storage/storage-dotnet-how-to-use-blobs)와 [Cosmos DB](https://azure.microsoft.com/ko-kr/services/cosmos-db/)를 통해 이 아키텍처의 이미지 및 메타데이터 저장을 관리합니다.

![아키텍처 다이어그램](../images/AI_Immersion_Arch.png)


## 아키텍처

여러분의 팀은 최근에 잠재적 아키텍처(아래)를 제시했고 Adventure Works에서 이를 승인했습니다.

![아키텍처](../images/AI_Immersion_Arch.png)

* [Computer Vision](https://azure.microsoft.com/ko-kr/services/cognitive-services/computer-vision/)을 통해 이미지 업로드 허용 및 콘텐츠 감지
* [QnA Maker](https://azure.microsoft.com/ko-kr/services/cognitive-services/qna-maker/)를 통해 정적 기술 자료에서 봇 상호 작용 촉진
* [Text Analytics](https://azure.microsoft.com/ko-kr/services/cognitive-services/text-analytics/)를 통해 언어 감지
* [LUIS](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/Home)(Language Understanding Intelligent Service)를 통해
텍스트에서 의도 및 엔터티 추출
* [Azure Bot Service](https://azure.microsoft.com/ko-kr/services/bot-service/) 커넥터 서비스를 통해 챗봇 인터페이스를 사용하여 앱 인텔리전스 활용

## 다음 단계

-   [랩 01-02: 기술 요구 사항](02-Technical_Requirements.md)