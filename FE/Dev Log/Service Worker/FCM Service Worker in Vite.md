# FCM Service Worker in Vite

## Issue
- FCM의 tree-shaking이 가능한 modular API를 사용해야 함.
- 사용하려면 `import`가 가능한 Vite의 bundling pipeline에 포함이 되어야 함.

## Assessment
```tsx
// src/main.tsx

navigator.serviceWorker
	.register("/firebase-messaging-sw.js")
	.then(() => console.log("Registered FCM SW"))
	.catch((err) => console.log("Failed to register FCM SW", err));
```

```js
// src/firebase-messaging-sw.js

import { initializeApp } from "firebase/app";
import { getMessaging, onBackgroundMessage } from "firebase/messaging/sw";

const firebaseApp = initializeApp({
  // ...
});

const messaging = getMessaging(firebaseApp);

onBackgroundMessage(messaging, (payload) => {
	// ...
});
```

다음과 같은 에러가 남:
```
A bad HTTP response code (404) was received when fetching the script.
```



- `firebase-messaging-sw.js`을 `src` folder안에 넣으면 아래 에러가 뜸.
	```
	TypeError: Failed to register a ServiceWorker for scope ('http://localhost:5173/src/') with script ('http://localhost:5173/src/firebase-messaging-sw.js'): ServiceWorker script evaluation failed
	```


- Vite의 `public` folder는 여기에 포함되지 않음. 곧 바로 serve 됨.
	- Path Ex: `http://localhost:5173/firebase-messaging-sw.js`


Namespaced API Example
```js
importScripts("https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js");
importScripts("https://www.gstatic.com/firebasejs/8.10.1/firebase-messaging.js");
```
Modular API Example
```js
import { initializeApp } from "firebase/app";
import { getMessaging, onBackgroundMessage } from "firebase/messaging/sw";
```

