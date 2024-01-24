# Push Notifications

## Table of Contents
- [[#FineAnts Notification Feature]]
- [[#Push API]]
- [[#Notifications API]]
- [[#Client API]]
- [[#Flow]]

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

## Push API
- Allows the Server to send a message to a Client even when the web application is not in the foreground on the browser.
- Done via a push service (service worker) at any time.
### Overview
![[webpush-architecture.png]]
- UA creates a new message subscription by sending a POST request to the Push Service.
	- If successful, a URI for the push message subscription is created and responded in the Location header.
- UA distributes the subscription (the push URI) to the Application Server.
- The Application Server uses the subscription to send messages to the Push Service.
	- The Application Server sends an HTTP POST request to the Push Service with the message content in the `body` of the request.
		- It must also include the Time-To-Live(TTL) header (value in seconds indicating how long a push message is retained by the Push Service) when requesting for push message delivery.
			- Ex: if the user is offline, the Push Service can store the push messages for a period so that they can be displayed when they're back online.
		- If successful, the Push Service responds with 201 and a URI for the push message resource placed in the Location header. (Note: does not mean that the message has been delivered to UA).
			- If the Application Server wants to know when the push message is delivered to the UA (push message receipt), it can include the `Prefer header` field with the `"respond-async"` preference, which the Push Service will confirm the delivery. (Note: the particular Push Service must support delivery confirmations).
	- The Application Server can send an HTTP GET request to the receipt subscription resource to check the delivery of receipts from the Push Service. The Push Service does not respond to this request; instead, it uses HTTP/2 server push to send push receipts when messages are acknowledged by the UA.
		- The response to the synthesized GET request includes a status code indicating the result of the message delivery and carries no data.
- The UA uses the subscription to monitor the Push Service for incoming messages.
	- The UA sends a GET request to a push message subscription resource. The Push Service does not respond to this request; instead, it uses HTTP/2 server push to send the contents of the push messages send by the Application Server.
	- The UA must send an HTTP DELETE request to the Push Service on the push message resource to indicate that it received the push message.
		- If the DELETE request is not sent within some time, the Push Service considers the message not delivered and will retry deliverying the message until the message expires.
### Flow of Events for Subscription
![[push-api-sequence-diagram.png]]

### Browser Compatibility
- Fully supported in Chrome, Edge, FireFox.
- Partially Supported in Safari - requires macOS 13 (Ventura) and later.
### Reference
https://www.rfc-editor.org/rfc/rfc8030
https://www.w3.org/TR/push-api/
https://developer.mozilla.org/en-US/docs/Web/API/Push_API

## Notifications API
- Allows the web app (browser) to display notifications on the OS even when the application is idle or in the background.
### FineAnts Requirements & Browser Compatibility
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
### Reference
https://www.w3.org/TR/notifications/

## Client API
- `postMessage()` - allows a Service Worker to send a message to a Client (a Window, Worker, or SharedWorker).
	- The message is received in the `"message"` event on `navigator.serviceWorker`.
### Browser Compatibility
- Fully supported in all browsers.
### Reference
https://developer.mozilla.org/en-US/docs/Web/API/Client/postMessage

## Flow

1. Get user permission to send push notifications.
	1. Allow notifications for `https://fineants.co` in Chrome.
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

### VAPID Keys
> An application server can voluntarily identify itself to a push service using the described technique. This identification information can be used by the push service to attribute requests that are made by the same application server to a single entity. This can used to reduce the secrecy for push subscription URLs by being able to restrict subscriptions to a specific application server.  An application server is further able to include additional information that the operator of a push service can use to contact the operator of the application server. | IETF

- Only allow your Server to be able to send notifications to a subscribing Client.
#### Reference
https://datatracker.ietf.org/doc/html/draft-thomson-webpush-vapid-02
https://vapidkeys.com/

## Reference
[Notifications  |  web.dev](https://web.dev/explore/notifications)  
[Push notifications overview  |  Articles  |  web.dev](https://web.dev/articles/push-notifications-overview)  
[Add push notifications to a web app  |  Google Codelabs](https://codelabs.developers.google.com/codelabs/push-notifications#0)  
