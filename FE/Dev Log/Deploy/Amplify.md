# Amplify 배포

## Table of Contents
- [[#SPA Redirect and Rewrites 설정]]

## SPA Redirect and Rewrites 설정
- 대부분 SPA는 사이트내에서의 navigate(i.e. `/index.html`에서 출발을 할 때)을 할 때에는 서버로의 요청없이도 HTML5 `history.pushState()`으로 브라우저의 location을 바꿀 수 있다.
- 하지만, 사이트내의 다른 페이지로 *바로* navigate할 때에는 실패한다.
	- Ex: OAuth Redirect URI.
	- Ex: `site.amplifyapp.com/signin`을 직접 브라우저에 입력하면 해당 페이지를 찾는데 실패한다.
- 실패하는 이유는, SPA는 `index.html`이 먼저 로드가 되어야하지만, 바로 `site.amplifyapp.com/signin`을 한다는 것은 `index.html` 및 라우트 처리를 해줄 스크립트가 로드되지 않았기 때문이다.
### 설정 내용
- 정규식 표현을 사용해서 특정 파일 확장자들을 제외한 모든 파일들에 대한 요청을 `index.html`로 200 rewrite하도록 설정해야 한다.
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
