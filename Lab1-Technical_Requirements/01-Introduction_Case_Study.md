# 랩 1: 기술 요구 사항

## 소개

이 랩에서는 워크샵 사례 연구를 소개하고 Microsoft Cognitive Services 제품군 내에서 도구를 빌드할 수 있도록 로컬 워크스테이션과 Azure 인스턴스에 도구를 설정합니다.

## 워크샵 사례 연구

고객에게 자전거와 자전거 장비를 판매하는 새로운 고객인 Adventure Works LLC가 할당되었습니다.

Notebook 은 대규모 다국적 제조 회사입니다. 인터넷 채널과 리셀러 유통망을 통해 북미, 유럽 및 아시아 상업 시장에 자전거 및 자전거 부품을 제조 및 판매하고 있습니다. 기본 업무는 워싱턴 주 커클랜드에서 290명의 직원으로하고 있으며, 시장 기반 전체에 여러 지역 영업 팀이 있습니다.

성공적인 회계 연도를 맞이하여 Adventure Works는 기존 고객에게 추가 판매를 목표로 하여 수익을 높이려 합니다. 작년에 마케팅 부서는 다양한 무역 박람회 및 레이싱 이벤트에서 Adventure Works 제품의 제품 평가 등급 데이터를 수동으로 수집하는 이니셔티브를 시작했습니다. 이 데이터는 현재 Microsoft Excel 파일에 보관되어 있습니다.

수집된 정보를 통해 평가 등급 결과의 수동 분석으로 기존 고객 기반에 추가 제품 판매가 가능함을 입증할 수 있었습니다. 마케팅 팀은 Adventure Works 웹 사이트에서 온라인 버전의 설문조사를 만들어 이를 확장하고자 했지만 이러한 노력에 대해 알게된 영업 부서는 온라인 설문조사를 만들었습니다. 이 플랫폼의 설문 조사는 매우 열악했으며 Excel 파일에서 수집된 데이터와 필드가 일치하지 않았습니다.

마케팅 부서는 평가 등급 데이터를 사용하여 다른 제품을 권장함으로써 비즈니스의 상향 판매 기회로 활용할 수 있을 것으로 봅니다. 그러나 웹 사이트는 설문 조사를 배치하여 필요 데이터를 수집하기에 바람직하지 않은 채널임이 드러났으며, 이러한 상황의 개선 방법에 대한 조언을 구하고 있습니다. 또한 팀은 고객의 수가 늘어날 수록 수동 분석의 난이도가 올라간다는 것을 인식하고 있습니다.

 Adventrue Works는 다양한 언어를 구사하는 고객의 대량 문의를 처리하기 위한 원활한 확장을 목표로 합니다. 또한 확장 가능한 고객 서비스 플랫폼을 만들어 고객의 요구, 문제 및 제품 평가 등급에 대한 더 많은 인사이트를 얻고자 합니다.

또한 고객 서비스 부서는 고객 지원 기능 중 일부를 대화형 플랫폼으로 오프로드하려 합니다. 직원의 워크로드를 줄이고 일반적인 질문에 신속하게 답변하여 고객 만족도를 높이는 것이 목적입니다.

### 솔루션

대화형 플랫폼은 다음과 같은 기능으로 구성된 봇으로 구상합니다.

- 고객 언어 감지(현재 영어 응답만 지원)

- 사용자의 감정 모니터링

- 이미지 업로드 허용 및 개체가 자전거인지 확인

- FAQ를 챗봇에 통합

- 봇 채팅에서 입력한 텍스트를 기반으로 사용자의 의도 확인

- 추후 검토를 위해 채팅 봇 세션으로 로그

로컬 드라이브에서 사진을 수집한 다음 [Computer Vision API](https://www.microsoft.com/cognitive-services/ko-kr/computer-vision-api)를 호출하여 이미지를 분석하고 태그 및 설명을 얻을 수 있는 단순한 C# 애플리케이션을 빌드합니다.

랩 전반에 걸쳐 이 랩에서 계속하여 고객의 텍스트 문의와 상호 작용하는 [Bot Framework](https://dev.botframework.com/) 봇을 빌드하는 방법을 보여 드리겠습니다. 그런 다음 [QnA Maker](https://docs.microsoft.com/ko-kr/azure/cognitive-services/qnamaker/overview/overview)를 통해 기존 기술 자료 및 FAQ를 Bot Framework에 통합하기 위한 간단한 솔루션을 시연합니다. 마지막으로, 쿼리에서 의도를 자동으로 도출하고 이를 바탕으로 고객의 텍스트 요청에 지능적으로 응답하도록 [LUIS](https://www.microsoft.com/cognitive-services/ko-kr/language-understanding-intelligent-service-luis)를 사용하여 이 봇을 확장하겠습니다.

고객이 봇과 상호 작용하는 동안 다른 데이터에 액세스할 수 있도록 Bing Search를 사용하기 위한 컨텍스트만 제공하며 이러한 시나리오를 랩에서 구현하지는 않습니다. 참가자는 [Bing Web Search](https://azure.microsoft.com/ko-kr/services/cognitive-services/directory/search/) 서비스에 대한 자세한 내용을 확인할 수 있습니다.

이 랩의 범위를 벗어나지만 이 아키텍처는 Azure의 데이터 솔루션을 통합하고, [Blob Storage]((https://docs.microsoft.com/ko-kr/azure/storage/storage-dotnet-how-to-use-blobs)와 [Cosmos DB](https://azure.microsoft.com/ko-kr/services/cosmos-db/)를 통해 이 아키텍처의 이미지 및 메타데이터 저장을 관리합니다.

![아키텍처 다이어그램](../images/AI_Immersion_Arch.png)

### 아키텍처

팀에서는 최근 Adventure Works가 승인한 잠재적 아키텍처(아래)를 소개했습니다.

![아키텍처](../images/AI_Immersion_Arch.png)

- [Computer Vision](https://azure.microsoft.com/ko-kr/services/cognitive-services/computer-vision/)이미지 업로드 및 콘텐츠 감지
- [QnA Maker](https://azure.microsoft.com/ko-kr/services/cognitive-services/qna-maker/)정적 지식 기반의 봇 상호작용 지원
- [Text Analytics](https://azure.microsoft.com/ko-kr/services/cognitive-services/text-analytics/)를 통해 언어 감지
- [LUIS](https://docs.microsoft.com/ko-kr/azure/cognitive-services/LUIS/Home)(Language Understanding 지능형 서비스)
텍스트에서 의도 및 엔터티 추출
- [Azure Bot Service](https://azure.microsoft.com/ko-kr/services/bot-service/) 챗봇 인터페이스를 사용하도록 설정하여 앱 인텔리전스를 활용하는 커넥터 서비스

## 다음 단계

- [랩 01-02: 기술 요구 사항](02-Technical_Requirements.md)