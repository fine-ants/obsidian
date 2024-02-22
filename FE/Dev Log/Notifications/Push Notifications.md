# Push Notifications

## Table of Contents
- [FineAnts Notification Feature](#fineants-notification-feature)
- [Prerequisites](#prerequisites)
- [APIs](#apis)
	- [Push API](#push-api)
	- [Notifications API](#notifications-api)
	- [Client API](#client-api)
	- [VAPID Keys](#vapid-keys)
- [Firebase Cloud Messaging(FCM)](#firebase-cloud-messagingfcm)
	- [Overview](#overview)
	- [구독 과정](#구독-과정)
	- [Message 과정](#message-과정)
	- [Message Type](#message-type)
	- [FineAnts Notification](#fineants-notification)

## FineAnts Notification Feature
- 포트폴리오 목표 수익률 알림
- 포트폴리오 최대 손실율 알림
- 지정가 알림
	- 한 종목 당 최대 5개 제한
- 알림 생명: 1달
- 알림 패널을 여는 순간 모두 읽음 처리
	- UI는 빨간점 유지. 새로고침 및 다른 페이지 가면 빨간점 사라짐.

## Prerequisites
- Web push notifications can be implementing using a combination of the Push API, Notifications API, and Service Worker API.
### Service Worker
- A type of web worker (browser API) that acts as a proxy between web applications and servers.
	- i.e. Service Workers can intercept network requests on behalf of the web application.
	- Service Workers can be used to control and intercept network requests, cache resources, manage background tasks.
- A JS script that runs in a worker context (no DOM access) in the background, and doesn't run on the main thread (non-blocking, fully async).
- Only works over HTTPS.
- A page needs to be within the registered Service Worker's scope.
	- Ex: a Service Worker loaded from `/subdir/sw.js` can only control pages located within `/subdir/`.
- List of running service workers on Chrome (chrome://serviceworker-internals/).
- Service Worker Life Cycle
	- Registration
		- Check if service worker is supported in the User Agent.
		- `navigator.serviceWorker.register("/sw.js")` when the page has fully loaded (using the main thread to register).
		- Service Worker state is "installing".
	- Installation
		- "install" event is fired.
		- Once installation finishes, Service Worker state becomes "activating"
	- Activation
		- "activate" event is fired.
- Push Service
	- A web service controlled by the user's browser vendor.
	- We as developers do not have or have to control the choice of the Push Service as it is standardized.
	- We as developers only need to ensure that we are sending the web push protocol request to the correct Push Service (necessary information is provided in the `PushSubscription` data that the browser received during the subscription process), and that our web push protocol request follows the spec (Ex: certain mandatory headers, and data must be sent as a stream of bytes).
		- `PushSubscription` object Example
			```json
			{
				// "endpoint" is the client identifier information that helps the Push Service determine the target Client to push the message to.
				"endpoint": "https://pushuri...",
				"expirationTime": null,
				// "keys" is used for the Server to encrypt the push message data when sending it to the Push Service
				"keys": {
					"p256dh": "blahblah",
					"auth": "blahblah"
				}
			}
			```
### Reference
https://developer.chrome.com/docs/workbox/service-worker-lifecycle

## APIs
### Push API
- Allows the Server to send a message to a Client even when the web application is not in the foreground on the browser.
- Done via a push service (service worker) at any time.
#### Web Push 구조
<div align="center">
	<img src="https://raw.githubusercontent.com/fine-ants/obsidian/main/FE/Dev%20Log/Notifications/refImg/webpush-architecture.png" alt="Web Push 구조"/>
</div>

#### Push Service Subscription 및 Push Message 흐름
<div align="center">
	<img src="https://raw.githubusercontent.com/fine-ants/obsidian/main/FE/Dev%20Log/Notifications/refImg/push-api-sequence-diagram.png" alt="Push Service Subscription 및 Push Message 흐름"/>
</div>

##### Push Service Subscribe 과정
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
##### Push Message 과정
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
##### Push Service Unsubscribe 과정
11. FE는 BE로 Subscription 해제 요청을 보냄.
12. BE는 사용자와 관련된 Subscription 정보를 제거함.
13. FE는 `PushSubscription: unsubscribe()`을 통해 Browser를 거치고 Push Service에서 unsubscribe함.
#### Browser Compatibility
- Fully supported in Chrome, Edge, FireFox.
- Partially Supported in Safari - requires macOS 13 (Ventura) and later.
#### Reference
https://www.rfc-editor.org/rfc/rfc8030
https://www.w3.org/TR/push-api/
https://developer.mozilla.org/en-US/docs/Web/API/Push_API
[Notifications  |  web.dev](https://web.dev/explore/notifications)  
[Push notifications overview  |  Articles  |  web.dev](https://web.dev/articles/push-notifications-overview)  
[Add push notifications to a web app  |  Google Codelabs](https://codelabs.developers.google.com/codelabs/push-notifications#0)  

### Notifications API
- Allows the web app (browser) to display notifications on the OS even when the application is idle or in the background.
#### FineAnts Requirements & Browser Compatibility
- `Notification.title` ("FineAnts")
- `Notification.options.body` (Alert content)
- `Notification.options.icon` (FineAnts logo)
- `Notification.options.timestamp`
- `Notification.options.requireInteraction`
	- Not supported in Chrome.
	- Partially supported in FireFox (only on Windows).
- `Notification.permission` (whether the user granted permission for FineAnts to display notifications)
- `Notification.requestPermission()` (request notification permission)
	- FireFox 72 requires `Notification.requestPermission()` to be called from a user invoked event (Ex: click).
- Other
	- Chrome 49 doesn't allow notifications in incognito mode.
#### Reference
https://www.w3.org/TR/notifications/

### Client API
- `postMessage()` - allows a Service Worker to send a message to a Client (a Window, Worker, or SharedWorker).
	- The message is received in the `"message"` event on `navigator.serviceWorker`.
#### Browser Compatibility
- Fully supported in all browsers.
#### Reference
https://developer.mozilla.org/en-US/docs/Web/API/Client/postMessage

### VAPID Keys
> An application server can voluntarily identify itself to a push service using the described technique. This identification information can be used by the push service to attribute requests that are made by the same application server to a single entity. This can used to reduce the secrecy for push subscription URLs by being able to restrict subscriptions to a specific application server.  An application server is further able to include additional information that the operator of a push service can use to contact the operator of the application server. | IETF

- Only allow your Server to be able to send notifications to a subscribing Client.
#### Reference
https://datatracker.ietf.org/doc/html/draft-thomson-webpush-vapid-02
https://vapidkeys.com/

## Firebase Cloud Messaging(FCM)
### Overview
<div align="center">
	<img src="https://raw.githubusercontent.com/fine-ants/obsidian/main/FE/Dev%20Log/Notifications/refImg/fcm-illustration.png" alt="FCM Overview"/>
</div>

### 구독 과정
1. FE는 사용자로부터 Push Notification 알림 승인을 받음.
	1. 즉, `https://fineants.co` 가 Chrome을 통해 Notification을 보낼 수 있도록 승인.
	2. 비고: 사용자는 OS 설정에서 Chrome이 데스크탑 Notification을 보여줄 수 있도록 설정을 해줘야함.
2. FE는 FCM으로 Subscribe 요청을 보내어 해당 기기를 등록.
	1. 이때, **Vapid Key**를 포함하여 보냄.
3. FCM은 Registration Token을 생성하여 FE로 응답함.
4. FE는 받은 Registration Token을 BE로 보냄.
5. BE는 해당 정보를 DB에 저장함.
### Message 과정
<div align="center">
	<img src="https://raw.githubusercontent.com/fine-ants/obsidian/main/FE/Dev%20Log/Notifications/refImg/client-fcm-server-message-flow.png" alt="FCM을 활용한 Message 과정"/>
</div>

6. BE는 FCM Admin SDK를 사용하여 Message을 생성하여 FCM Backend으로 보냄.
	- Message Example
		```json
		{
			"message": {
				"token": "target client's registration token",
				
				// handled by FCM Client SDK
				"notification": {
					"title": "Portfolio Achievement", // reserved key
					"body": "Portfolio1 has reached its target valuation" // reserved key
				},
				// handled by Client App
				"data": {
					"title": "Portfolio Achievement", // custom key
					"body": "Portfolio1 has reached its target valuation", // custom key
					"tu": "pac", // custom key
				},
				
				// Optional platform-specific options
				"android": {
					"TTL": "4500s",
					"notification": {
						"click_action": "OPEN_ACTIVITY_!"
					},
					// ...
				},
				"apns": {
					"payload": {
						"aps": {
							"category": "NEW_MESSAGE_CATEGORY"
						}
					},
					// ...
				},
				"webpush": {
					"headers": {
						"TTL": "86400"
					},
					"notification": {
						"requireInteraction": true,
						"icon": "/icons/notification.png"
					},
					"fcm_options": {
						"link": "https://fineants.co"
					}
				}
			}
		}
		```
7. FCM Backend은 Message ID와 metadata를 생성하여 BE로부터 받은 Message와 함께 특정 platform transport layer (Ex: Web Push)로 보냄.
8. 기기가 온라인이라면 해당 platform transport layer을 통해 기기로 보내짐.
9. Client App의 Background/Foreground 상태와 `data`/`notification` key 여부에 따라 desktop notification을 보내거나 애플리케이션으로 전달함.
### Message Type
- A message can be either of type notification or data or both.
- Messages are handled differently depending on the message type and the Client App's Background/Foreground state.
#### Notification Message
- Set the `notification` key in the payload to send a Notification Message type.
	- Can only contain reserved keys (Ex: `title`, `body`).
	- `data` payload도 optional로 포함할 수 있음.
- Client App이 Background에 있을시, FCM Client SDK가 자동으로 핸들링 함.
	- `data` payload가 있다면, User가 notification tray에 뜬 notification을 클릭 했을 시에만 Client App에서 callback을 통해 `data` payload을 핸들링할 수 있음.
- Client App이 Foreground에 있을시, Client App이 callback 함수를 통해 `notification` 및 `data` payload 둘다를 핸들링할 수 있음.
- Firebase Console 또는 Admin SDK 및 FCM Server Protocol을 활용하여 보낼 수 있음.
- Max. Payload = 4,000 bytes.
	- 1000 character limit when sent from Firebase Console (Notifications Composer).
- Example
	```js
	const payload = {
		notification: {
			title: "$FooCorp up 1.43% on the day",
			body: "$FooCorp gained 11.80 points to close at 835.67, up 1.43% on the day."
		}
	};
	```
#### Data Message
- Client App이 핸들링해야 함.
- Set the `data` key in the payload to send a Data Message type.
	- Can contain custom keys that are not reserved keys (Ex: `"notification"`, `"from"`, `"message_type"`).
	- The `data` payload is received in a callback function in the Client App.
- Admin SDK 및 FCM Server Protocol을 활용하여 보낼 수 있음.
- Max. Payload = 4,000 bytes.
- Example
	```js
	const payload = {
		data: {
			score: "850",
			time: "2:45"
		}
	};
	```
#### Notes
- On Android and Web/JS, `TTL` can be set between 0 to 2,419,200 seconds (28 days). 
	- `TTL = 0` means that messages that cannot be delivered immediately are discarded.
### FineAnts Notification

```json
{
	notification: {
		title: "",
		body: ""
	},
	data: {
		title: "",
		body: "",
	},
	"webpush": {
		"fcm_options": {
			"link": "" // link to open upon user clicking on notification
		}
	}
}
```

### Reference
[FCM Architectural Overview  |  Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/fcm-architecture)
[Set up a JavaScript Firebase Cloud Messaging client app](https://firebase.google.com/docs/cloud-messaging/js/client)
[About FCM messages  |  Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/concept-options)
