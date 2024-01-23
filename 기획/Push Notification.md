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

1. FE는 사용자로부터 Push Notification 알림 승인을 받는다.
	1. 즉, `https://fineants.co` 가 Chrome.
	2. Allow notifications for Chrome in OS.
2. The Client (hence, the browser) sends a subscribe request to a Push Service (using the Push API) including your **Public Authentication Key** (to which the Push Service will associate the resulting endpoint with).
3. The Push Service generates and sends a `PushSubscription` object that contains the subscription's URL endpoint, to which your **Public Key** is associated to.
4. The Client sends the received `PushSubscription` object to the Server to be stored in a DB.
---------------------------------------
5. The Server sends a message request (**web push protocol request**) to the Push Service (endpoint included in the `PushSubscription` object).
	1. The **web push protocol request** includes the message content, the target Client to send the message to, and instructions on how the Push Service should deliver the message (Ex: TTL header, delay time, etc).
	2. Use your **Private Key** to sign the JSON information.
	3. Web Service Request Java Library Ex: https://github.com/web-push-libs/webpush-java
6. The Push Service receives and authenticates the Server's request using the stored **Public Key**, and then routes the message to the target Client.
	1. If the Client is offline, the Push Service queues the push message until the Client comes online or until the message expires.
7. The Browser receives and decryptes the push message data and dispatches a `push` event to your Service Worker.
	1. The `push` event handler should call `ServiceWorkerRegistration.showNotification()` to display the information as a notification.