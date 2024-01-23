# Push Notification 기능

## Table of Contents
- [[#FineAnts Push Notification 기능]]

## FineAnts Push Notification 기능
- 포트폴리오 목표 수익률 알림
- 포트폴리오 최대 손실율 알림
- 종목 현재가 알림


## Overview
### Web Push 구조
![[webpush-architecture.png]]

### Push Service Subscription 흐름
![[push-api-sequence-diagram.png]]

1. FE는 사용자로부터 Push Notification 알림 승인을 받음.
	1. 즉, `https://fineants.co` 가 Chrome을 통해 Notification을 보낼 수 있도록 승인.
	2. 비고: 사용자는 OS 설정에서 Chrome이 데스크탑 Notification을 보여줄 수 있도록 설정을 해줘야함.
2. FE는 Browser의 Push API를 통해 Push Service로 Subscribe 요청을 보냄.
	1. 이때, Public Key를 포함하여 보냄.
3. Push Service는 `PushSubscription` 객체를 생성하여 FE로 응답함.
	1. 이 `PushSubscription` 객체는 받은 Publick Key와 연결된 Subscription URL Endpoint를 담고 있음.
4. FE는 받은 `PushSubscription` 객체를 BE로 보냄.
5. BE는 해당 정보를 DB에 저장함.
---------------------------------------
6. The Server sends a message request (**web push protocol request**) to the Push Service (endpoint included in the `PushSubscription` object).
	1. The **web push protocol request** includes the message content, the target Client to send the message to, and instructions on how the Push Service should deliver the message (Ex: TTL header, delay time, etc).
	2. Use your **Private Key** to sign the JSON information.
	3. Web Service Request Java Library Ex: https://github.com/web-push-libs/webpush-java
7. The Push Service receives and authenticates the Server's request using the stored **Public Key**, and then routes the message to the target Client.
	1. If the Client is offline, the Push Service queues the push message until the Client comes online or until the message expires.
8. The Browser receives and decryptes the push message data and dispatches a `push` event to your Service Worker.
	1. The `push` event handler should call `ServiceWorkerRegistration.showNotification()` to display the information as a notification.