# Push Notifications

## Requirements
- Title (FineAnts)
- Body (Stock price description based on set alert price)

## Push API
- Allows the Client to receive messages from the Server even when the web application is not in the foreground on the browser.
- The Server can send push messages via a push service (service worker) at any time.
### Overview
![[webpush-architecture.png]]
- UA creates a new message subscription by sending a POST request to the Push Service.
	- If successful, a URI for the push message subscription is created and responded in the Location header.
- UA distributes the subscription (the push URI) to the Application Server.
- The Application Server uses the subscription to send messages to the Push Service.
	- The Application Server sends an HTTP POST request to the Push Service with the message content in the `body` of the request.
	- If successful, the Push Service responds with 201 and a URI for the push message resource placed in the Location header. (Note: does not mean that the message has been delivered to UA).
		- If the Application Server wants to know when the push message is delivered to the UA (push message receipt), it can include the `Prefer header` field with the `"respond-async"` preference, which the Push Service will confirm the delivery. (Note: the particular Push Service must support delivery confirmations).
- The UA uses the subscription to monitor the Push Service for incoming messages.
### Browser Compatibility
- Fully supported in Chrome, Edge, FireFox.
- Supported in Safari on macOS 13 (Ventura) and later.
### Reference
https://www.rfc-editor.org/rfc/rfc8030
https://www.w3.org/TR/push-api/

## Notifications API
- Allows the web app (browser) to send notifications to the OS even when the application is idle or in the background.
### Browser Compatibility
- Supported in Chrome, Edge, FireFox, Safari.
	- 
### Reference
https://www.w3.org/TR/notifications/

## Reference
[Push API - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)  
[Notifications  |  web.dev](https://web.dev/explore/notifications)  
[Add push notifications to a web app  |  Google Codelabs](https://codelabs.developers.google.com/codelabs/push-notifications#0)  
