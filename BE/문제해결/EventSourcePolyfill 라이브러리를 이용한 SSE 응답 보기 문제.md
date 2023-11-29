## 상황
프론트 엔드 클라이언트에서 EventSourcePolyfill 라이브러리를 이용하여 Spring 서버로부터 SSE 요청을 했을 때 구글 개발 툴로 응답을 확인하여야 하는데 확인을 할 수 없습니다.

![[Pasted image 20231129135026.png]]

## 원인
EventSourcePolyfill 라이브러리를 사용하는 경우 구글 개발 툴에서는 일반적으로 text/event-stream 응답을 볼 수 없습니다. 깃허브 이슈 참고 자료에 따르면 Native EventSource를 사용한다면 text/event-stream을 볼 수 있지만 eventsource-polyfill을 사용한다면 정보를 볼 수 없다고 합니다.

![[Pasted image 20231129135137.png]]

## 해결방법
위 문제를 해결하기 위해서는 SSE Viewer 구글 확장 프로그램을 설치하여 볼 수 있습니다.

1. SSE Viewer 구글 확장 프로그램을 설치합니다.
https://chromewebstore.google.com/detail/sse-viewer/pkofiecpdokojdgoccnbfplkphbmppaf

3. SSE Viewer 확장 프로그램을 활성화합니다.
![[Pasted image 20231129135701.png]]

3. 구글 개발툴을 열어서 sse 응답이 오는지 확인합니다.
![[Pasted image 20231129135744.png]]

