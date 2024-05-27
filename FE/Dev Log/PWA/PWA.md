
# PWA

## Background
- Mobile browser에서는 데스크탑에서 하는 것 처럼 기존 알림 기능을 사용하지 못한다.
- PWA는 해당 디바이스/플랫폼의 앱처럼 설치가 되고, 오프라인 및 백그라운드에서도 실행이 가능하다.

## What is PWA?
- It combines traditional websites and platform-specific apps.
### Characteristics
#### That of Traditional Websites
- PWAs are developed based on web platform technologies. Hence, they can run on different operating systems and devices from a *single codebase*.
- PWAs can be accessed directly from the web.
#### That of Platform-specific Apps
- PWAs can be installed directly on the device. Once installed, it can be launched as a standalone app.
	- When a web app is "added/installed to home":
		- If it is a PWA, it opens as a standalone app.
		- If it is not a PWA, it opens in the default browser of the device.
- PWAs can operate in the background and offline (through service workers).

## PWA and the Browser
- Although PWAs usually look like a standalone application, they are still websites. Therefore, they need a browser engine to manage and run them.
	- cf. in a platform-specific app, the OS runs them.
![Diagram comparing the runtime environment for traditional websites, PWAs, and platform-specific apps](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/What_is_a_progressive_web_app/pwa-environment.svg)

## PWA Technical Requirements
- Must have a web app manifest file for browsers to display things like the app name, icon for the PWA install prompt.
	- Chromium-based browsers require `name`, `icons`, `start_url`, `display` and/or `display_override` manifest members.
- Must have a service worker for basic offline experience.
	- The service worker implements a backend which can support offline and background operation, making PWAs behave more like an app than a website.
	- Must have a `fetch` event handler that provides basic offline experience.
- Must be served in a secure context (served over https).

## Browser Support
- Desktop
	- Firefox and Safari do not support installing PWAs in any OS.
	- Chromium-based browsers support installing PWAs in Linux, Windows, MacOS, Chromebook.
- Mobile
	- Android 
		- Firefox and Chromium-based browsers support installing PWAs.
	- iOS
		- For iOS 16.3 >=, PWAs can only be installed with Safari.
		- For iOS 16.4 <=, PWAs can be installed from the "share" menu in Safari, Firefox, and Chromium-based browsers.

## Install Prompt
- `beforeinstallprompt` event (fired in the global `Window` object) has a `prompt()` method that shows a customizable install prompt.
	- Is not supported in iOS.

## Vite PWA Plugin
### Vite Build CLI Order of Execution (Vite PWA)
- ...
- `vite-plugin-pwa:build` plugin
	- Inject web manifest (mandatory) to html.
	- Inject registerSw script (if needed) to html.
	- Create and emit registerSw script if required.
- `vite-plugin-pwa:info` plugin
	- Resolve and load virtual:pwa-info when required.
- `vite-plugin-pwa:main` plugin
	- Resolve and load any virtual module (virtual:pwa-register, virtual:pwa-register/\<fw>) when required.
- `closeBundle` in `vite-plugin-pwa:build` plugin
- `buildEnd` in `vite-plugin-pwa:build` hook
- Copy `public/` folder content to `dist/`.

## References
- https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/What_is_a_progressive_web_app
- https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/Making_PWAs_installable
- https://vite-pwa-org.netlify.app/guide/
- https://vite-pwa-org.netlify.app/guide/cookbook.html





- [ ] Mobile browser 알림
	- 알림 display를 못함. 하려면, 아마도 PWA 등 앱처럼 핸드폰에 설치가 되어 있어야 할 듯함.
		- https://developer.apple.com/documentation/usernotifications/sending-web-push-notifications-in-web-apps-and-browsers
		- https://firebase.blog/posts/2023/08/fcm-for-safari/#how-to-enable-firebase-cloud-messaging-on-safari-for-ios-and-ipados