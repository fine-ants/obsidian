## Sample React app setup
fire_client라는 이름의 리액트 프로젝트를 생성합니다.
```
$ create-react-app fire_client
```

Firebase Cloud Messaing의 클라이언트 사이드 기능을 위한 의존성을 추가합니다.
```
$ npm install --save firebase react-bootstrap bootstrap
```


## Firebase setup
이번 단계에서는 Firebase 프로젝트를 생성합니다.

1. 프로젝트 추가를 클릭합니다.
![[Pasted image 20240131133011.png]]

2. 프로젝트 이름을 입력합니다.
![[Pasted image 20240131133131.png]]

3. Google 애널리스틱 구성을 선택합니다.
![[Pasted image 20240131133226.png]]

4. 새로 만든 프로젝트 개요 페이지에서 웹 앱을 클릭하여 앱을 추가합니다.
![[Pasted image 20240131133358.png]]

5. 앱 닉네임을 입력합니다.
![[Pasted image 20240131133431.png]]

6. 앱 추가를 확인합니다.
![[Pasted image 20240131133540.png]]

7. firebase.js 파일을 생성하고 다음과 같이 Firebase 설정을 입력합니다.
```js
import { getMessaging, getToken, onMessage } from "firebase/messaging";

import { initializeApp } from 'firebase/app';

  

const firebaseConfig = {

apiKey: "...",

authDomain: "...",

projectId: "...",

storageBucket: "...",

messagingSenderId: "...",

appId: "...",

measurementId: "..."

};

  

const firebaseApp = initializeApp(firebaseConfig);
```

위 firebaseConfig 객체의 값들은 Firebase 프로젝트의 FineAnts 웹앱에 프로젝트 설정 - 일반 페이지에서 확인할 수 있습니다.

## Integrate cloud messaing
다음으로 우리는 웹 푸시 인증 키를 생성하여야 합니다. FineAnts 웹앱 페이지에서 클라우드 메시징 -> 웹 구성으로 이동합니다.
![[Pasted image 20240131134038.png]]

위 화면 웹 구성에서 Generate key pair 버튼을 클릭하여 웹 푸시 인증 키를 생성합니다.

firebase.js 파일로 돌아와서 메시징을 활성화하기 위해서 다음과 같이 import합니다.
```
import { getMessaging, getToken, onMessage } from "firebase/messaging";
```

그리고 나서 우리는 firebase 객체로부터 messaging 객체에 다음과 같이 접근할 수 있습니다.
```
const firebaseApp = initializeApp(firebaseConfig);
const messaging = getMessaging(firebaseApp);
```

### Notification permissions and registering a client
브라우저로 push notification을 전송하기 위해서 우리는 사용자로부터 알림 허용 권한을 얻어야 합니다. 그러면 다른 웹 사이트에서 봤을 법한 "Enable notifications?" 팝업이 열립니다.

이 요청을 시작하는 방법은 Firebase가 제공하는 getToken 메소드를 호출하는 것입니다. 그전에 App.js 파일에 다음과 같이 state 변수를 추가합니다. 해당 변수는 우리가 notification 접근 권한을 가지고 있는지 추적합니다.

```
const [isTokenFound, setTokenFound] = useState(false);
getToken(setTokenFound);

// inside the jsx being returned:
{isTokenFound && 
 Notification permission enabled 👍🏻 
}
{!isTokenFound && 
 Need notification permission ❗️ 
}
```

firebase.js 파일에 다음과 같이 추가합니다.
```js
export const getToken_ = (setTokenFound) => {

return getToken(messaging, {vapidKey: 'Firebase에서 생성한 웹 푸시 인증키'}).then((currentToken) => {

if (currentToken) {

console.log('current token for client: ', currentToken);

setTokenFound(true);

// Track the token -> client mapping, by sending to backend server

// show on the UI that permission is secured

} else {

console.log('No registration token available. Request permission to generate one.');

setTokenFound(false);

// shows on the UI that permission is required

}

}).catch((err) => {

console.log('An error occurred while retrieving token. ', err);

// catch error while creating client token

});

}

```

리액트를 실행하여 권한 요청을 허용하게 되면 다음과 같은 화면으로 변경됩니다.
![[Pasted image 20240131134913.png]]

콘솔 로그를 확인하면 다음과 같이 token이 FCM으로부터 발급된 것을 확인할 수 있습니다.
![[Pasted image 20240131134950.png]]


## Configuring message listeners
### Background listener
지금 우리는 알림 권한과 브라우저 사이드에서 클라이언트 토큰을 가지고 있습니다. 다음 단계는 클라이언트 방향으로 직접적으로 오는 push notification 리스너를 추가하는 것입니다.

firebase-messaing-sw.js service worker 파일을 React app의 public 디렉토리에 추가합니다. 내용은 다음과 같습니다.
```
// Scripts for firebase and firebase messaging

importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js');

importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-messaging-compat.js');

  

// Initialize the Firebase app in the service worker by passing the generated config

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};

firebase.initializeApp(firebaseConfig);

  

// Retrieve firebase messaging

const messaging = firebase.messaging();

  

messaging.onBackgroundMessage(function(payload) {

console.log('Received background message ', payload);

  

const notificationTitle = payload.notification.title;

const notificationOptions = {

body: payload.notification.body,

};

  

self.registration.showNotification(notificationTitle,

notificationOptions);

});
```


### Foreground listener
클라이언트 앱이 포어그라운드 상태로 활성화되어 있는 경우를 고려하여 firebase.js 파일에 다음 메시지 리스너를 추가합니다.
```
export const onMessageListener = () =>

new Promise((resolve) => {

onMessage(messaging, (payload) => {

resolve(payload);

});

});
```

그리고 App.js 파일에 다음과 같이 파싱된 payload에서 알림을 생성하기 위한 로직을 추가합니다. App.js의 전체 코드는 다음과 ㄱ
```

```