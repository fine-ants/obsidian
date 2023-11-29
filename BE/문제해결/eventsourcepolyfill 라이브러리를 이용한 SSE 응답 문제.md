## 상황
프론트 엔드 클라이언트에서 EventSourcePolyfill 라이브러리를 이용하여 Spring 서버로부터 SSE 요청을 했을 때 구글 개발 툴로 응답을 확인하여야 하는데 확인을 할 수 없습니다.

![[Pasted image 20231129135026.png]]

## 원인
EventSourcePolyfill 라이브러리를 사용하는 경우 구글 개발 툴에서는 일반적으로 EventStream 응답을 볼 수 없습니다.