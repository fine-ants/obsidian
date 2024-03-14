# Service Worker, Env, Dev & Prod ft. Vite

## 관련 포스팅
https://www.reddit.com/r/Firebase/comments/1beemv3/fcm_service_workers_and_env_variables_ft_vite/
https://stackoverflow.com/questions/78151771/serving-service-worker-file-in-src-in-vite-dev-mode
https://www.reddit.com/r/Firebase/comments/18fj4zp/how_do_you_handle_sensitive_variables_with_a/

## 배경 및 목표
- Firebase API Key는 몇개를 제외하고는 코드에 직접 기입을 해도 괜찮음 (그리고 어차피 client에서 다 노출 됨).
	- [https://firebase.google.com/docs/projects/api-keys#api-keys-for-firebase-are-different](https://firebase.google.com/docs/projects/api-keys#api-keys-for-firebase-are-different)
- 하지만, GitHub repo에는 안올라갔으면 함.
- 즉, 환경변수를 사용하려 함.

## `public/firebase-messaging-sw.js`
- Service worker 파일에서 환경 변수 접근(import.meta.env 또는 process.env)이 안됨.
- 개발 모드 (문제 없음)
	- Custom plugin에서 dotenv를 활용하여 `process.env`가 가능함.
	- “[process.env.xyz](http://process.env.xyz)” 임의의 문자열을 실제 `process.env`에 있는 값으로 대체.
- 빌드 (문제 있음)
	- 빌드할 때는 public에 있는 파일들은 조작없이 그대로 dist에 output 되기 때문에 custom plugin 방식 등으로 환경 변수 값을 주입할 수 없음.
	- 대신, 빌드 시 node script로 파일을 읽고 “[process.env.xyz](http://process.env.xyz)” 문자열들을 `process.env` 값으로 대체하여 파일을 새로 output할 수 있음.
		- `”build”: “node —env-file=.env scripts/sw.js && tsc && vite build“`
		- 하지만, 이 방법은 실제 파일 자체를 변경하기 때문에 이상적이지 않음.

## `src/firebase-messaging-sw.js`
- Service worker 파일에서 환경 변수 접근(`import.meta.env`) 가능.
- 개발 모드 (문제 있음)
	- Service worker 경로가 안맞음.
	- FCM SDK가 `GET /firebase-messaging-sw.js`을 하지만, 거기에 없고 `/src/firebase-messaging-sw.js`에 있기 때문에 404에러가 뜸.
- 해결 시도
	- server.proxy config으로 `GET /firebase-messaging-sw.js`을 `src/firebase-messaging-sw.js`으로 proxy.
		- `Uncaught SyntaxError: Cannot use 'import.meta' outside a module (at firebase-messaging-sw.js:1:8)` 에러가 뜸.
		- `vite preview`를 할 때는 500에러가 뜸.
- 빌드 (문제 없음)
	- Service worker 파일이 src 안에 있으므로, dist의 루트 레벨로 output되지 않고 메인 애플리케이션 코드와 같이 번들링이 되기 때문에 build config에서 메인 애플리케이션과 service worker 파일을 구분하여 두개의 entry point와 output을 설정하면 됨.

## 정말 최후의 수단
- public/firebase-messaging-sw.js에 두고 값을 하드 코딩.

