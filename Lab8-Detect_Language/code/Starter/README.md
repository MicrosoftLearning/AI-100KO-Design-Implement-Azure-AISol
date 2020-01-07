# PictureBot

Bot Framework v4 에코 봇 샘플.

이 봇은 [Bot Framework](https://dev.botframework.com)를 사용하여 만들어졌으며, 사용자의 입력을 받아들이고 다시 에코하는 간단한 봇을 만드는 방법을 보여 줍니다.

## 사전 요구 사항

- [.NET Core SDK](https://dotnet.microsoft.com/download) 버전 2.1

```bash
  # dotnet 버전 확인
  dotnet --version
```

## 이 샘플 시도해 보기

- 터미널에서 `PictureBot` 으로 이동

```bash
    # 프로젝트 폴더로 변경
    cd # PictureBot
```

- 터미널 또는 Visual Studio에서 봇을 실행하고 옵션 A 또는 B를 선택합니다.

  A) 터미널에서

```bash
  # 봇 실행
  dotnet run
```

  B) Visual Studio에서

  - Visual Studio 시작
  - 파일 -> 열기 -> 프로젝트/솔루션
  - `PictureBot` 폴더로 이동
  - `PictureBot.csproj` 파일 선택
  - `F5` 키를 눌러 프로젝트 실행

## Bot Framework Emulator를 사용하여 봇 테스트

[Bot Framework Emulator](https://github.com/microsoft/botframework-emulator) 는 봇 개발자가 로컬호스트에서 또는 터널을 통한 원격 실행으로 봇을 테스트하고 디버깅할 수 있도록 하는 데스크톱 애플리케이션입니다.

- [여기](https://github.com/Microsoft/BotFramework-Emulator/releases) 에서 Bot Framework Emulator 버전 4.5.0 이상 설치

### Bot Framework Emulator를 사용하여 봇에 연결

- Bot Framework Emulator 시작
- 파일 -> 봇 열기
- 봇 URL로 `http://localhost:3978/api/messages` 입력

## Azure에 봇 배포

Azure에 봇을 배포하는 방법에 대해 자세히 알아보려면 [Azure에 봇 배포](https://aka.ms/azuredeployment) 를 참조하여 배포 지침의 전체 목록을 확인하십시오.

## 추가적인 읽을거리

- [Bot Framework 설명서](https://docs.botframework.com)
- [봇 기본 사항](https://docs.microsoft.com/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0)
- [활동 처리](https://docs.microsoft.com/ko-kr/azure/bot-service/bot-builder-concept-activity-processing?view=azure-bot-service-4.0)
- [Azure Bot Service 소개](https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0)
- [Azure Bot Service 설명서](https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0)
- [.NET Core CLI 도구](https://docs.microsoft.com/ko-kr/dotnet/core/tools/?tabs=netcore2x)
- [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest)
- [Azure Portal](https://portal.azure.com)
- [LUIS를 사용한 Language Understanding](https://docs.microsoft.com/ko-kr/azure/cognitive-services/luis/)
- [채널 및 Bot Connector Service](https://docs.microsoft.com/ko-kr/azure/bot-service/bot-concepts?view=azure-bot-service-4.0)
