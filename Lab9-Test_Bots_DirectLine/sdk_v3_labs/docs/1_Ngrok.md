# Ngrok를 통한 개발 및 테스트
 
## 1.	목표
 
Microsoft Bot Framework를 사용하여 특정 채널에서 봇을 사용할 수 있도록 구성하려면 공용 URL 끝점에 봇 서비스를 호스트해야 합니다. 채널이 NAT 또는 방화벽 뒤에 숨겨진 로컬 서버 포트에 있는 경우 봇 서비스에 액세스할 수 없습니다.
  
코드를 디자인/작성/테스트할 때 지속적으로 재배포할 필요가 없도록 해야 합니다. 추가 호스팅 비용이 발생하기 때문입니다. ngrok를 사용하면 봇의 개발/테스트 단계를 가속화하는 데 실질적인 도움이 될 수 있습니다. 이 랩의 목표는 ngrok를 사용하여 공용 인터넷에 봇을 공개하고 공용 끝점을 사용하여 에뮬레이터에서 봇을 테스트하는 것입니다.
  
## 2.	설정
  
 a.	  ngrok가 설치되어 있지 않은 경우 https://ngrok.com/download 에서 해당 OS에 맞는 ngrok를 다운로드하고 설치합니다. 다운로드한 ngrok 파일의 압축을 풀고 설치합니다.

 b.	  이전 랩에서 만든 봇 프로젝트를 열거나 "봇 응용 프로그램" 프로젝트 템플릿을 사용하여 새 EchoBot을 만듭니다. 봇을 실행합니다. EchoBot을 만드는 경우 브라우저에서 아래 메시지가 표시됩니다.

**EchoBot**

여기서 봇에 대한 설명과 봇의 사용 약관을 제공합니다.

Bot Framework를 방문하여 봇을 등록합니다. 등록할 때 봇의 끝점을 https://your_bots_hostname/api/messages 로 설정해야 합니다.

*** 상기 정보는 단지 정보 제공을 위한 것이며, 이 랩에서는 봇을 등록할 필요가 없습니다.

## 3.	전달

 a.	 봇이 localhost:3979에서 호스팅되고 있으므로 ngrok를 사용하여 이 포트를 공용 인터넷에 공개할 수 있습니다.

 b.	 터미널을 열고 ngrok가 설치된 폴더로 이동합니다.
 
 c.	 아래 명령을 실행하면 전달 URL이 표시됩니다.

 ````ngrok.exe http 3979 -host-header="localhost:3979"````

![전달 URL](images/ForwardingUrl.png)

 e.	 Bot Emulator에 전달 URL(http)을 입력합니다. 봇 URL에서는 /api/messages 가 전달 URL에 추가됩니다. 메시지를 보내 에뮬레이터에서 봇을 테스트합니다. 이제 로컬 호스트 대신 공용 끝점을 사용하여 봇을 테스트했습니다.


![봇 URL](images/BotUrl.png)

## 4.	일찍 끝났다면 추가 크레딧을 위해 이 연습을 시도해 보십시오.

 Microsoft Bot Framework에서 봇을 등록할 때 메시지 끝점에 전달 URL을 사용할 수 있습니까? 원하는 채널(예: Skype)에서 전달 URL을 사용하여 봇을 테스트합니다.

 ### [2_Unit_Testing_Bots](2_Unit_Testing_Bots.md)로 계속 진행

 [README](../0_README.md)로 돌아가기
