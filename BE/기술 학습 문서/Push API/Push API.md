
## Abstract
Push API는 push service를 통해서 웹 애플리케이션에 push message의 전송을 활성화시킵니다. 웹 애플리케이션이나 User Agent(브라우저 등)가 비활성화되었을 때도 애플리케이션 서버는 어느 때건 push message를 전송할 수 있습니다. push service는 User Agent에게 신뢰적이고 효율적인 배송을 보장합니다. push message들은 웹 애플리케이션의 원격지에서 실행하는 Service Worker로 배송됩니다. 이 배송되는 정보들은 사용자에게 로컬 상태나 알림을 알려주기위한 메시지 정보입니다.

Push API 스펙은 애플리케이션 서버나 User Agent가 push service와 같이 상호작용하는 방법을 설명하는 **web push protocol**을 사용합니다.


## 1. Introduction
Push API는 비동기적으로 User Agent와 통신하는 것을 웹 애플리케이션에게 허용합니다. Push API를 통해 애플리케이션 서버는 사용자가 웹 애플리케이션을 열기를 기다리는 대신에 정보가 알려질 때마다 User Agent에게 실시간으로 민감한 정보를 제공할 수 있습니다.

위에서 정의한 대로 push service는 어느때건 push message의 전송을 지원합니다.

특히, push message는 웹 애플리케이션이 브라우저창이 닫혀있는데도 불구하고 전송됩니다. 이는 사용자가 웹 애플리케이션을 닫을 수 있지만, push message가 수신될때 웹 애플리케이션이 재시작될 수 있다는 이점으로부터 여전히 이익을 얻는 사례에 관한 것입니다. 예를 들어, push message는 수신되는 WebRTC(Web Real-Time Communication) 호출을 사용자에게 알리기 위해 사용될 수 있습니다.

push message는 또한 User Agent가 오프라인일 때도 전송가능합니다. 이를 지원하기 위해 push service는 User Agent가 이용이 가능할 때까지 User Agent에 대한 메시지를 저장합니다.




