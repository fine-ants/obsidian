
## Abstract
Push API는 push service를 통해서 웹 애플리케이션에 push message의 전송을 활성화시킵니다. 웹 애플리케이션이나 User Agent(브라우저 등)가 비활성화되었을 때도 애플리케이션 서버는 어느 때건 push message를 전송할 수 있습니다. push service는 User Agent에게 신뢰적이고 효율적인 배송을 보장합니다. push message들은 웹 애플리케이션의 원격지에서 실행하는 Service Worker로 배송됩니다. 이 배송되는 정보들은 사용자에게 로컬 상태나 알림을 알려주기위한 메시지 정보입니다.

Push API 스펙은 애플리케이션 서버나 User Agent가 push service와 같이 상호작용하는 방법을 설명하는 **web push protocol**을 사용합니다.


## 1. Introduction
Push API는 비동기적으로 User Agent와 통신하는 것을 웹 애플리케이션에게 허용합니다. Push API를 통해 애플리케이션 서버는 사용자가 웹 애플리케이션을 열기를 기다리는 대신에 정보가 알려질 때마다 User Agent에게 실시간으로 민감한 정보를 제공할 수 있습니다.

위에서 정의한 대로 push service는 어느때건 push message의 전송을 지원합니다.

특히, push message는 웹 애플리케이션이 브라우저창이 닫혀있는데도 불구하고 전송됩니다. 이는 사용자가 웹 애플리케이션을 닫을 수 있지만, push message가 수신될때 웹 애플리케이션이 재시작될 수 있다는 이점으로부터 여전히 이익을 얻는 사례에 관한 것입니다. 예를 들어, push message는 수신되는 WebRTC(Web Real-Time Communication) 호출을 사용자에게 알리기 위해 사용될 수 있습니다.

push message는 또한 User Agent가 오프라인일 때도 전송가능합니다. 이를 지원하기 위해 push service는 User Agent가 이용이 가능할 때까지 User Agent에 대한 메시지를 저장합니다. 이는 웹 애플리케이션이 사용자가 오프라인 상태에서 발생하는 변경 사항을 학습하고 User Agent가 적시에 관련된 정보를 제공받을 수 있도록 사용 사례를 지원합니다. push message는 push service에 의해서 User Agent가 메시지를 받을 때까지 저장됩니다.

또한 Push API는 User Agent가 웹 애플리케이션을 능동적으로 사용하는 동안, push message의 신뢰성 있는 전송을 보장합니다. 예를 들어 사용자가 웹 애플리케이션을 능동적으로 사용하거나 웹 애플리케이션이 active worker, frame, 백그라운드 창을 통해 애플리케이션 서버와 통신하는 동안 push message의 신뢰성 있는 전송을 보장합니다. 이것은 Push API에 대한 주요한 사용 사례는 아닙니다. 웹 애플리케이션은 애플리케이션 서버와 지속적인 통신을 유지할 필요가 없도록 Push API를 사용하여 빈번하지 않은 메시지를 선택할 수 있습니다.

push message는 User Agent와 웹 애플리케이션 사이에 설립된 활성화된 통신 채널이 준비되지 않을 때 적합합니다. Fetch API 또는 WebSocket과 같은 직접적인 통신 방법과 비교했을 때 push message를 전송하는 것은 훨씬 더 많은 리소스를 필요로 합니다. push message는 보통 직접 통신보다 더 높은 지연율을 가지고 있고 사용에 제한을 받을 수 있습니다. 대부분의 push service는 전송할 수 있는 push message의 크기와 개수를 제한합니다.

---
정리하면 Push API는 웹 애플리케이션과 User Agent간에 비동기적으로 통신하는 것을 지원하는 API입니다. Push API를 통해서 User Agent가 먼저 웹 애플리케이션에 요청하고 응답을 받는 Pull 방식 대신 웹 애플리케이션 서버에서 먼저 데이터를 전송하는 방식을 제공합니다.

위와 같은 방식을 수행하기 위해서 Push API는 push service를 통해서 push message를 User Agent에게 전송합니다. Push API의 장점은 push message를 User Agent가 오프라인 인 경우에도 전송할 수 있다는 점입니다. 전송하기 위해서 push message는 push service에 의해서 메시지를 받을 때까지 저장하다가 User Agent가 온라인 경우에 

## Dependencies
web push protocol(RTC8030)은 User Agent, 애플리케이션 서버와 push service간에 통신 방법을 설명합니다.  



