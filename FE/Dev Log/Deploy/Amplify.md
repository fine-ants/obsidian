# Amplify 배포

## Table of Contents
- [[#Redirect and Rewrites 설정]]

## Redirect and Rewrites 설정
- 대부분 SPA는 사이트내에서의 navigate(i.e. `/index.html`에서 출발을 할 때)을 할 때에는 서버로의 요청없이도 HTML5 `history.pushState()`으로 브라우저의 location을 바꿀 수 있다.
- 하지만, 사이트내의 다른 페이지를 바로 navigate할 때에는 실패한다.
	- Ex: OAuth Redirect URI.
- 정규식 표현을 사용해서 특정 파일 확장자들을 제외한 모든 파일들을 `index.html`로 200 rewrite하도록 설정해야한다.
### 설정 내용
- Source Address
	```
	</^[^.]+$|\.(?!(css|gif|ico|jpg|js|png|txt|svg|woff|woff2|ttf|map|json|webp)$)([^.]+$)/>
	```
- Target Address
	- `/index.html`
- Type
	- `200 (Rewrite)`
### Reference
- [Using redirects - AWS Amplify Hosting](https://docs.aws.amazon.com/amplify/latest/userguide/redirects.html#redirects-for-single-page-web-apps-spa)
