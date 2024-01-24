# Push Notification 기능

## Table of Contents
- [[#FineAnts Push Notification 기능 요구사항]]
- [[#Overview]]
	- [[#Web Push 구조]]
	- [[#Push Service Subscription 및 Push Message 흐름]]
		- [[#Push Service Subscribe 과정]]
		- [[#Push Message 과정]]
		- [[#Push Service Unsubscribe 과정]]
	- [[#Client-Server Push Notification 흐름]]
- [[#Reference]]

## FineAnts Push Notification 기능 요구사항
- 포트폴리오 목표 수익률 알림
- 포트폴리오 최대 손실율 알림
- 지정가 알림

## Overview
### Web Push 구조
![[webpush-architecture.png]]

### Push Service Subscription 및 Push Message 흐름
![[push-api-sequence-diagram.png]]

#### Push Service Subscribe 과정
1. FE는 사용자로부터 Push Notification 알림 승인을 받음.
	1. 즉, `https://fineants.co` 가 Chrome을 통해 Notification을 보낼 수 있도록 승인.
	2. 비고: 사용자는 OS 설정에서 Chrome이 데스크탑 Notification을 보여줄 수 있도록 설정을 해줘야함.
2. FE는 Browser의 Push API를 통해 Push Service로 Subscribe 요청을 보냄.
	1. 이때, **Public Key**를 포함하여 보냄.
3. Push Service는 `PushSubscription` 객체를 생성하여 FE로 응답함.
	1. 이 `PushSubscription` 객체는 받은 Publick Key와 연결된 Subscription URL Endpoint를 담고 있음.
		```json
		// PushSubscription Example
		{
			"endpoint": "https://push_service_endpoint...",
			"expirationTime": null,
			"keys": {
				"p256dh": "BBFkhjj6iuxKoFTwl_l25xlUO4RaUHl6iXCXoBtsHCXQ9V4VVaMrZCF",
				"auth": "pE6ZHrFS1bisuuW6hCowrA"
			}
		}
		```
4. FE는 받은 `PushSubscription` 객체를 BE로 보냄.
5. BE는 해당 정보를 DB에 저장함.
#### Push Message 과정
6. BE는 새로운 메시지를 **web push protocol request**에 맞게 Push Service(`PushSubscription` 객체에 들어있는 endpoint)로 보냄.
	1. **Web push protocol request**은 메시지 내용, 메시지를 받을 타겟 Client, 그리고 Push Service가 "어떻게" 메시지를 전달할지(Ex: TTL header)를 포함.
	2. **Private Key**로 JSON 내용을 sign해야 함.
	3. Web Service Request Java Library Ex: https://github.com/web-push-libs/webpush-java
7. Push Service는 BE로부터 받은 메시지를 들고 있는 **Public Key**로 검증한 후, 타겟 Client로 메시지를 전달함.
	1. Client가 현재 offline이면, Push Service는 메시지들을 queue에 저장해두고 Client가 online이 될 때 전달함. 또는, 설정해둔 메시지 시간이 만료되면 queue에서 제거함.
8. Browser는 받은 push message을 decrypt하고 Service Worker에 `push` event을 통해 메시지를 전달함.
9. Service Worker에 있는 `push` event handler는
	1. `ServiceWorkerRegistration: showNofication()`을 통해 데스크탑 push notification을 보냄.
	2. `Client: postMessage()`을 통해 FE로 메시지를 전달함.
10. FE는 받은 메시지를 UI에 반영.
	1. BE와 적절한 상태 관리(Ex: 읽음, 등) 필요.
#### Push Service Unsubscribe 과정
11. FE는 BE로 Subscription 해제 요청을 보냄.
12. BE는 사용자와 관련된 Subscription 정보를 제거함.
13. FE는 `PushSubscription: unsubscribe()`을 통해 Browser를 거치고 Push Service에서 unsubscribe함.

### Client-Server Push Notification 흐름
![[client-server-push-notification.png]]

## Reference
### HTTP Web Push 관련
https://www.rfc-editor.org/rfc/rfc8030
https://www.w3.org/TR/push-api/
https://web.dev/articles/push-notifications-overview
### Public/Private Key 관련
https://datatracker.ietf.org/doc/html/draft-thomson-webpush-vapid-02
https://vapidkeys.com/
