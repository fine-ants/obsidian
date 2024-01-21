# Push Notifications

## Table of Contents
- [[#FineAnts Notification Feature]]
- [[#Push API]]
- [[#Notifications API]]
- [[#Flow]]

## FineAnts Notification Feature
- 포트폴리오 목표 수익률 알림
- 포트폴리오 최대 손실율 알림
- 종목 현재가 알림

## Prerequisites
- Service Worker
	- Act as proxies between browsers and servers.
	- A JS script that runs in a worker context (no DOM access), and doesn't run on the main thread (non-blocking, fully async).
	- Only works over HTTPS.
	- List of running service workers on Chrome (chrome://serviceworker-internals/).

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

## Flow

1. Get user permission to send push notifications.
2. Subscribe the Client to Push Service (Push API).

## Reference
[Push API - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)  
[Notifications  |  web.dev](https://web.dev/explore/notifications)  
[Add push notifications to a web app  |  Google Codelabs](https://codelabs.developers.google.com/codelabs/push-notifications#0)  
