
## Abstract
Push API는 push service를 통해서 웹 애플리케이션에 push message의 전송을 활성화시킵니다. 웹 애플리케이션이나 User Agent(브라우저 등)가 비활성화되었을 때도 애플리케이션 서버는 어느 때건 push message를 전송할 수 있습니다. push service는 User Agent에게 신뢰적이고 효율적인 배송을 보장합니다. push message들은 웹 애플리케이션의 원격지에서 실행하는 Service Worker로 배송됩니다. 이 배송되는 정보들은 사용자에게 로컬 상태나 알림을 알려주기위한 메시지 정보입니다.

Push API 스펙은 애플리케이션 서버나 User Agent가 push service와 같이 상호작용하는 방법을 설명하는 **web push protocol**을 사용합니다.


## 1. Introduction
Push API는 비동기적으로 User Agent와 통신하는 것을 웹 애플리케이션에게 허용합니다. Push API는 


