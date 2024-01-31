
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

위와 같은 방식을 수행하기 위해서 Push API는 push service를 통해서 push message를 User Agent에게 전송합니다. Push API의 장점은 push message를 User Agent가 오프라인 인 경우에도 전송할 수 있다는 점입니다. 전송하기 위해서 push message는 push service에 의해서 메시지를 받을 때까지 저장하다가 User Agent가 온라인 경우(백그라운드 포함)에 push message를 전송받습니다.

Push API의 push message의 단점은 직접적으로 통신하는 것보다 지연율이 높고 push service가 전송할 수 있는 push message의 크기와 개수에 제한을 가집니다.

## 2. Dependencies
web push protocol(RTC8030)은 User Agent, 애플리케이션 서버와 push service간에 통신 방법을 설명합니다.  대체적인 프로토콜들이 web push protocol들을 대신하여 사용될 수 있지만, 해당 스펙에서는 web push protocol의 사용을 가정합니다. 대체적인 프토콜들은 호환되는 의미들을 제공할 것으로 예상됩니다.

RFC7231 3.1.2.2 섹션에 설명된 Content-Encoding HTTP Header는  push message의 payload에 적용되는 컨텐츠 코딩을 나타냅니다.

## 3. Concepts
### 3.1 Application Server
Application Server는 웹 애플리케이션의 서버 사이드 컴포넌트를 의미합니다.

### 3.2 Push Message
push message는 애플리케이션 서버에서 웹 애플리케이션에 전송하는 데이터입니다.

push message는 메시지가 제출된 push subscription과 관련된 active worker에게 전달됩니다. 만약 service worker가 현재 실행중이지 않다면, worker는 배송을 시작합니다.

### 3.3 Push subscription
push subscription은 웹 애플리케이션 대신에 **User Agent와 push service 사이에 설립된 메시지 전송 컨텍스트**입니다. 각각의 push subscription은 service worker 등록과 관련이 있고, service worker 등록은 최소 하나 이상의 push subscription을 가지고 있습니다.

push subscription에는 push endpoint가 있습니다. push endpoint는 애플리케이션 서버가 push message를 보낼 수 있는 push service에 의해서 노출되는 절대 경로 URL이어야 합니다. push endpoint는 push subscription을 고유하게 식별할 수 있어야 합니다.

push subscription는 subscription 만료 시간을 가질 수 있습니다. 만료시간이 설정될 때, 만료시간은 1970년 1월 1일 00:00:00 UTC 이후 구독이 만료되는 시간(밀리초 단위)이어야 합니다. User Agent는 subscription 만료 전에 push subscription 갱신을 위해 갱신을 시도하여야 합니다. 

push subscription은 [RFC8291]에 따른 P-256 ECDH 키페어 및 인증 비밀(authentication secret)을 위한 내부 슬롯을 갖고 있습니다. 이 슬롯들은 push subscription이 생성될 때 채워져야 합니다. 

If the [user agent](https://www.w3.org/TR/push-api/#dfn-user-agent) has to change the keys for any reason, it _MUST_ [fire the "`pushsubscriptionchange`" event](https://www.w3.org/TR/push-api/#dfn-fire-the-pushsubscriptionchange-event) with the [service worker registration](https://www.w3.org/TR/service-workers/#dfn-service-worker-registration) associated with the [push subscription](https://www.w3.org/TR/push-api/#dfn-push-subscription) as registration, a [`PushSubscription`](https://www.w3.org/TR/push-api/#dom-pushsubscription) instance representing the [push subscription](https://www.w3.org/TR/push-api/#dfn-push-subscription) having the old keys as oldSubscription and a [`PushSubscription`](https://www.w3.org/TR/push-api/#dom-pushsubscription) instance representing the [push subscription](https://www.w3.org/TR/push-api/#dfn-push-subscription) having the new keys as newSubscription.

push subscription을 생성하기 위해서 PushSubscriptionOptionsInit 가 주어질때
1. subscription을 새로운 PushSubscription으로 설정합니다.
2. 옵션을 새로 생성한 PushSubscriptionOptions 객체로 설정하고 해당 객체 속성을 해당 멤버 및 optionsDictionary의 값으로 초기화합니다.
3. subscription의 options 속성을 options로 설정합니다.
4. 새로운 P-256 ECDH(Elliptic Curve Diffie-Hellman) 키 페어를 생성합니다. subscription의 내부 슬롯에 있는 private key를 저장합니다. private key값은 애플리케이션들에서 사용가능하면 안됩니다. public key 또한 내부 슬롯에 저장되고 "p256dh"의 매개변수와 같이 PushSubscription의 메소드 getKey()를 호출함으로써 가져올 수 있습니다.
5. 새로운 [RFC8291]에 정의된 8진수 시퀀스로 구성된 인증 비밀 정보를 생성합니다. subscription의 내부 슬롯에 인증 비밀 정보를 저장합니다. 이 인증 비밀 정보 키는 "auth" 매개변수와 같이 PushSubscription의 메소드 getKey()를 호출함으로써 가져올 수 있습니다.
6. 새로운 push subscription을 요청합니다. 설정된 옵션의 applicationServerKey 속성을 포함합니다. 
7. push subscription이 성공적으로 완료될 때
	1. push subscription에 의해서 제공되는 URL을 subscription의 endpoint 속성에 설정합니다.
	2. 만약 push subscription에 의해서 제공되면, subscription의 만료 시간을 설정합니다.
8. subscription을 반환합니다.

### 3.3.1 Subscription Refreshes
User Agent나 push service는 어느때건 push subscription을 갱신할 수 있습니다. 에를 들어 push subscription이 특정한 나이에 도달하는 경우가 있습니다.

이러한 경우 User Agent는 현재 push subscription을 생성하기 위해 제공된 PushSubscriptionOptions이 지정된 경우 push subscription을 생성하는 단계를 실행해야 합니다. 새로운 push subscription은 기존의 subscription과는 다른 키페어를 가져야 합니다.

성공적으로 새로운 push subscription이 생성되면, User Agent는 push subscription과 관련된 service worker 등록을 등록으로하는 "pushsubscriptionchange" 이벤트를 반드시 발생시켜야 합니다. PushSubscription 인스턴스는 초기의 push subscription을 oldSubscription으로서 나타내고 새로운 push subscription을 newSubscription으로 나타냅니다.

애플리케이션 서버들에 변경을 전파하기 위한 시간을 허용하기 위해서 User Agent는 갱신후에 짧은 시간동안 old push subscription을 대상으로한 메시들을 접수할 수 있도록 지속되어야 합니다. 갱신된 push subscription에 대한 메시지가 수신되면 이전 push subscription은 비활성화해야 합니다.

만약 User Agent가 push subscription 갱신을 하지 못한다면, User Agent는 주기적으로 갱신을 재시도해야 합니다. push subscription이 예를 들어 만료와 같은 이유로 더이상 사용되지 않는다면, User Agent는 



