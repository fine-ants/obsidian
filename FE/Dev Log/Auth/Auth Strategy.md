

## Table of Contents
- [[#초기 구현 방식]]
	- [[#문제점]]
		- [[#SPA와 OAuth 2.0 Authorization Code Flow의 문제]]
- [[#대안 1 OAuth 2.0 Authorization Code Flow with PKCE]]
	- [[#문제점]]
- [[#대안 2 OpenID Connect]]
- [[#지원하는 OAuth Provider]]

## 초기 구현 방식
- Client(SPA)에서 시작하는 기본 OAuth 2.0 Authorization Code Flow
### 문제점
#### SPA와 OAuth 2.0 Authorization Code Flow의 문제
- Client ID, Client Secret, Authorization Code(인가코드)가 노출된다.
	- 즉, 해커가 해당 정보를 탈취할 수 있다.
- Reference
	- [Authorization Code Flow with Proof Key for Code Exchange (PKCE)](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-proof-key-for-code-exchange-pkce)

## 대안 1: OAuth 2.0 Authorization Code Flow with PKCE
- 이를 보완하기 위해 OAuth 2.0은 Authorization Code Flow에 PKCE를 적용한 흐름을 추천한다.
- 기존 Authorization Code Flow와 동일하지만 아래와 같은 차이가 있다:
	- OAuth Client는 secret (Code Verifier)와 해당 secret의 변형된 값 (Code Challenge)를 생성한다.
	- OAuth Client는 OAuth Provider로부터 인가코드를 받기 위한 요청에 Code Challenge을 같이 보낸다.
	- OAuth Client가 성공적으로 인가코드를 받으면, 인가코드와 Code Verifier를 OAuth Provider로 보낸다.
	- OAuth Provider는 Code Verifier를 이전 단계에서 받은 Code Challenge을 이용하여 verify한다.
		- **해당 요청이 인가코드 요청을 한 클라이언트와 동일한지 확인.**
	- Code Verifier가 성공적으로 verify가 되었다면 OAuth Provider는 ID token과 Access Token을 반환한다.
- 개선 사항
	- 해커가 인가코드를 탈취했더라도 Code Verifier 없이는 Access Token을 발급 받을 수 없다.
	- 해커가 Code Challenge을 탈취하고 Code Challenge을 생성하기 위한 hashing algorithm을 알아내더라도, 1) Code Verifier를 추론할 수 없다 (단방향 알고리즘을 사용했기 때문), 2) 매 인가 요청마다 새로운 고유의 Code Verifier를 생성하기 때문에 추론하는데 의미가 없다.
- Reference
	- [RFC 7636 - Proof Key for Code Exchange by OAuth Public Clients](https://datatracker.ietf.org/doc/html/rfc7636)
	- [draft-ietf-oauth-security-topics-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-11)
### 문제점
- FineAnts는 OAuth Provider로 사용자를 대신하여 어떤 요청을 하지 않기 때문에, **OAuth을 인가 목적이 아닌 인증 목적으로 사용한다**.
- OAuth 2.0의 authentication layer인 **OpenID Connect를 사용하는 것이 더 적절하다**.

## 대안 2: OpenID Connect
- OpenID Connect는 OAuth 2.0의 identity layer로서 OAuth Client가 사용자를 인증하고 기본 정보를 받을 수 있는 프로토콜이다.
- 기본적인 흐름은 OAuth 2.0 Authorization Code Flow와 비슷하다.
	- 



## 지원하는 OAuth Provider
### Google
- Google Identity Services (Sign In With Google for Web)
	- Google의 OAuth 2.0을 기반하는 authentication 및 authorization을 한 패키지로 모아둔 SDK.
		- Authentication "순간"은 One Tap, automatic sign-in, Sign In With Google button을 제공한다.
			- 이 방식들은 ID Token만을 반환할 수 있고, OpenID Connect spec을 따른다.
			- 즉, Sign In With Google 방식들은 기본적으로 ID Token을 반환한다.
				- 해당 ID Token을 
		- Authorization "순간"은 후에 Google의 Resource Server로부터 데이터 접근이 필요할 때 실행한다.
			- 이는 code 또는 Access Token만을 반환할 수 있다.
	- Reference
		- [Overview  |  Authentication  |  Google for Developers](https://developers.google.com/identity/gsi/web/guides/overview#compare_to_oauth_and_openid_connect)
##### 고민
- Google은 보통의 경우에 직접적입 Google API 호출보다 해당 SDK 사용을 권장한다.
- 하지만 이는 frontend 코드에 Client ID를 포함한다.
- Client ID를 frontend 코드에 포함해야 하는데 이를 어떻게 보완할 수 있는가?
### Kakao
- Kakao는 OpenID Connect을 지원한다.
- Reference
	- [[공지] 카카오 로그인 OpenID Connect 지원 / [Notice] Support of OpenID Connect - Notice / 공지 - 카카오 데브톡](https://devtalk.kakao.com/t/openid-connect-notice-support-of-openid-connect/121888)
### Naver
- Naver는 OpenID Connect와 Authorization Code Flow with PKCE를 지원하지 않는다.
- 기본 Authorization Code Flow만 가능하다.



