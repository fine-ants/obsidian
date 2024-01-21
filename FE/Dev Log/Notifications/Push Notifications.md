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
	- If successful, a URI for the push message subscription is created.
- UA distributes the subscription to the Application Server.
- The Application Server uses the subscription to send messages to the Push Service.
- The UA uses the subscription to monitor the Push Service for incoming messages.
### Browser Compatibility
- Fully supported in Chrome, Edge, FireFox.
- Supported in Safari on macOS 13 (Ventura) and later.
### Reference
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
