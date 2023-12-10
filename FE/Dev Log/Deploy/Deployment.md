# 배포

## Table of Contents
- [[#Amplify]]
	- [[#Amplify를 선택한 이유]]
	- [[#SPA Redirect and Rewrites 설정]]
- [[#Custom Domain (GoDaddy)]]

## Amplify
### Amplify를 선택한 이유
#### 정적 콘텐츠 호스팅 최적화
- **CDN (Content Delivery Network) 사용**
	- 세계에 분산된 네트워크를 활용하여 사용자가 물리적으로 서버에 가까울수록 더 빠른 로딩 시간을 경험할 수 있게 빠르게 콘텐츠를 제공한다.
	- 트래픽이 많은 시간에도 CDN은 여러 서버에 요청을 분산시켜 안정성을 유지한다. 이는 서버 과부하를 방지하고, 사용자에게 일관된 경험을 제공
	- CDN은 자주 요청되는 콘텐츠를 캐시에 저장하여, 동일한 요청에 대해 빠르게 응답함으로 서버의 부담을 줄이고 전체적인 응답속도를 향상시킴
- **HTTPS 사용**
	- HTTPS는 데이터를 암호화하여 전송해서 사용자의 데이터가 중간에 가로채지거나 변조되는 것을 방지하여 보안을 강화한다.
	- SSL/TLS 인증서를 사용해 웹사이트의 신뢰성을 보장
	- Google과 같은 검색 엔진은 HTTPS를 사용하는 웹사이트를 선호하는데, 이는 검색 엔진 최적화(SEO)에 긍정적인 영향을 주고, 웹사이트의 검색 순위를 향상시킬 수 있다.
- **CI/CD**
	- 코드 변경이 발생할 때마다 자동으로 빌드 및 배포가 이뤄짐으로써, 코드 업데이트에만 집중가능 
### SPA Redirect and Rewrites 설정
- 대부분 SPA는 사이트내에서의 navigate(i.e. `/index.html`에서 출발을 할 때)을 할 때에는 서버로의 요청없이도 HTML5 `history.pushState()`으로 브라우저의 location을 바꿀 수 있다.
- 하지만, 사이트내의 다른 페이지로 *바로* navigate할 때에는 실패한다.
	- Ex: OAuth Redirect URI.
	- Ex: `site.amplifyapp.com/signin`을 직접 브라우저에 입력하면 해당 페이지를 찾는데 실패한다.
- 실패하는 이유는, SPA는 `index.html`이 먼저 로드가 되어야하지만, 바로 `site.amplifyapp.com/signin`을 한다는 것은 `index.html` 및 라우트 처리를 해줄 스크립트가 로드되지 않았기 때문이다.
#### 설정 내용
- 정규식 표현을 사용해서 특정 파일 확장자들을 제외한 모든 파일들에 대한 요청을 `index.html`로 200 rewrite하도록 설정해야 한다.
- Source Address
```
</^[^.]+$|\.(?!(css|gif|ico|jpg|js|png|txt|svg|woff|woff2|ttf|map|json|webp)$)([^.]+$)/>
```
- Target Address
	- `/index.html`
- Type
	- `200 (Rewrite)`
#### Reference
- [Using redirects - AWS Amplify Hosting](https://docs.aws.amazon.com/amplify/latest/userguide/redirects.html#redirects-for-single-page-web-apps-spa)

## Custom Domain (GoDaddy)
- `fineants.co`
- GoDaddy는 ANAME/ALIAS record를 지원하지 않는다.
	- 즉, `fineants.co`로 입력을 하면 브라우저 주소창에 `release.blah.amplify.com` 주소가 뜬다.
	- GoDaddy 설정에서 Forwarding with masking 설정이 있지만 이는 `release.blah.amplify.com`을 frame 또는 iframe에 로딩을 하려고 시도한다.
		- 이렇게 설정하고 접속할 시 frame 관련하여 에러가 떠서 사이트 접속이 불가하다.
	- AWS는 Route53으로 migrate을 안할 시, GoDaddy에서 Temporary Forwarding 적용을 추천한다.
		- 
### Reference
- https://docs.aws.amazon.com/amplify/latest/userguide/understanding-dns-terminology-and-concepts.html
- https://docs.aws.amazon.com/amplify/latest/userguide/to-add-a-custom-domain-managed-by-godaddy.html
- https://au.godaddy.com/help/forward-my-godaddy-domain-12123
- https://webmasters.stackexchange.com/questions/136652/how-can-i-mask-an-aws-site-with-a-domain-registered-with-godaddy

