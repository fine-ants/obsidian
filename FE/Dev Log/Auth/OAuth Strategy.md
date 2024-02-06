
# OAuth Strategy

## Table of Contents
- [초기 구현 방식](#초기-구현-방식)
	- [SPA와 OAuth 2.0 Authorization Code Grant](#spa와-oauth-20-authorization-code-grant)
	- [문제점](#문제점)
- [대안 1 OAuth 2.0 Authorization Code Grant with PKCE](#대안-1-oauth-20-authorization-code-grant-with-pkce)
	- [문제 및 참고점](#문제-및-참고점)
- [대안 2 OpenID Connect Authorization Code Grant with PKCE](#대안-2-openid-connect-authorization-code-grant-with-pkce)
- [대안 3: Client ID 숨기기 및 `state`, `nonce` Parameter 추가](#대안-3-client-id-숨기기-및-state-nonce-parameter-추가)
- [FineAnts가 지원하는 OAuth Login](#fineants가-지원하는-oauth-login)
- [기타 보안 내용](#기타-보안-내용)
	- [Authorization Code Replay Attack ft. PKCE](#authorization-code-replay-attack-ft-pkce)
	- [Cross Site Request Forgery(CSRF) ft. `state`](#cross-site-request-forgerycsrf-ft-state)
	- [ID Token Replay Attack ft. `nonce`](#id-token-replay-attack-ft-nonce)

## 초기 구현 방식
### SPA와 OAuth 2.0 Authorization Code Grant
- Client(SPA)에서 시작하는 기본 OAuth 2.0 Authorization Code Grant를 적용하여 Access Token과 Refresh Token을 발급받아서 서버에서 Access Token을 이용하여 사용자 정보를 가져오고 있다.
### Illustration
- *아래 그림과 같은 흐름이지만 authorization으로서 8번 단계에서 ID Token이 아니라 Access Token과 Refresh Token을 받고 있다.*

<div align="center">
	<img src="https://images.ctfassets.net/cdy7uua7fh8z/2nbNztohyR7uMcZmnUt0VU/2c017d2a2a2cdd80f097554d33ff72dd/auth-sequence-auth-code.png" alt="OAuth Auth Code Grant" width="80%" />
	<p><em>https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow</em></p>
</div>

### 문제점
- Client ID, Client Secret, Authorization Code이 노출된다.
	- 즉, 해커가 해당 정보를 탈취하여 해당 OAuth Client을 가장할 수 있다.
	- Ex: 탈취한 Authorization Code을 활용하여 Access Token을 발급 받아 OAuth Provider의 Resource Server에 있는 사용자의 정보를 접근할 수 있다.
- Reference
	- [Authorization Code Flow with Proof Key for Code Exchange (PKCE)](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-proof-key-for-code-exchange-pkce)

## 대안 1: OAuth 2.0 Authorization Code Grant with PKCE
- Authorization Code 탈취에 대한 문제를 보완하기 위해 OAuth 2.0은 Authorization Code Grant에 PKCE를 적용한 흐름을 권장 및 강조한다.
- 기존 Authorization Code Flow와 동일하지만 아래와 같은 차이가 있다:
	- OAuth Client는 secret(Code Verifier)와 해당 secret의 변형된 값(Code Challenge)을 생성한다.
	- OAuth Client는 OAuth Provider로부터 Authorization Code을 받기 위한 요청에 Code Challenge을 같이 보낸다.
	- OAuth Client가 성공적으로 Authorization Code을 받으면, Authorization Code와 Code Verifier를 OAuth Provider로 보낸다.
	- OAuth Provider는 Code Verifier를 이전 단계에서 받은 Code Challenge을 이용하여 verify한다.
		- **해당 요청이 Authorization Code을 요청한 클라이언트와 동일한지 확인.**
	- Code Verifier가 성공적으로 verify가 되었다면 OAuth Provider는 Access Token과 Refresh Token을 반환한다.
- 개선 사항
	- 해커가 Authorization Code을 탈취했더라도 Code Verifier 없이는 Access Token을 발급 받을 수 없다.
	- 해커가 Code Challenge을 탈취하고 Code Challenge을 생성하기 위한 hashing algorithm을 알아내더라도, 1) Code Verifier를 추론할 수 없다 (Code Challenge를 생성하는데 단방향 알고리즘을 사용했기 때문), 2) 매 인가 요청마다 새로운 고유의 Code Verifier를 생성하기 때문에 추론하는데 의미가 없다.
- Reference
	- [RFC 7636 - Proof Key for Code Exchange by OAuth Public Clients](https://datatracker.ietf.org/doc/html/rfc7636)
	- [draft-ietf-oauth-security-topics-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-11#section-2.1.1)
### Illustration
- *아래 그림과 같은 흐름이지만 8번 단계에서 ID Token이 아니라 Access Token과 Refresh Token을 받는다.*

<div align="center">
	<img src="https://images.ctfassets.net/cdy7uua7fh8z/3pstjSYx3YNSiJQnwKZvm5/33c941faf2e0c434a9ab1f0f3a06e13a/auth-sequence-auth-code-pkce.png" alt="OIDC with PKCE" width="80%" />
	<p><em>https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-proof-key-for-code-exchange-pkce</em></p>
</div>

### 문제 및 참고점
- FineAnts는 OAuth Provider로 사용자를 대신하여 어떤 요청을 하지 않기 때문에, **OAuth을 Authorization(인가) 목적이 아닌 Authentication(인증) 목적으로 사용한다**.
- OAuth 2.0의 identity layer인 **OpenID Connect을 사용하는 것이 더 적절하다**.
- *"**Authorization Code Grant**" 및 "**PKCE**"는 authorization, authentication 두 상황 모두에 보안을 강화하기 위해 적용 가능한 절차이다.*

## 대안 2: OpenID Connect Authorization Code Grant with PKCE
- OpenID Connect은 OAuth 2.0의 identity layer로서 OAuth Client가 사용자를 인증하고 기본 정보를 받을 수 있는 프로토콜이다.
- 기본적인 흐름은 OAuth 2.0 Authorization Code Grant with PKCE와 비슷하지만 아래와 같은 차이가 있다.
	- OAuth Provider는 OAuth Client로부터 받은 Authorization Code가 유효하다면 ID Token과 Access Token을 반환한다.
		- 해당 ID Token은 OAuth 등록시 명시한 scope 및 field(claim)를 담고 있다.
			- 기본 사용자 정보(Ex: name, email, picture)를 명시할 수 있다.
		- *OIDC 맥락에서 Access Token이란 추가적인 사용자 정보를 요청할 수 있다는 것이다.*
			- *cf. 기존 OAuth Authorization에서 Access Token이란 사용자를 대신해서 액션을 실행할 수 있도록 OAuth Client에 인가를 하는 것이다.*
	- OAuth Client는 ID Token을 validate한 후 사용자의 로그인을 승인한다.
- OIDC server flow는 Backend가 브라우저(Frontend)를 활용하여 사용자를 인증하는 방식이다.
- Reference
	- [Final: OpenID Connect Core 1.0 incorporating errata set 1](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth)
### Illustration
<div align="center">
	<img src="https://images.ctfassets.net/cdy7uua7fh8z/3pstjSYx3YNSiJQnwKZvm5/33c941faf2e0c434a9ab1f0f3a06e13a/auth-sequence-auth-code-pkce.png" alt="OIDC with PKCE" width="80%" />
	<p><em>https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-proof-key-for-code-exchange-pkce</em></p>
</div>

## 대안 3: Client ID 숨기기 및 `state`, `nonce` Parameter 추가
- 대안 2와의 차이점
	- Frontend가 Authorization URL을 직접 생성하는 것이 아니라 Backend로부터 Authorization URL을 요청한다는 것이다. 즉, Frontend는 Client ID를 들고 있지 않는다.
	- `state` parameter를 이용하여 CSRF 공격을 방어한다.
	- `nonce` parameter를 이용하여 ID Token Replay 공격을 방어한다.
- Flow
	- 사용자는 소셜 로그인 버튼을 누른다.
	- Frontend는 Backend로부터 해당 OAuth Authorization URL 받기위한 요청을 보낸다.
		- Ex: `await fetch('http://localhost:300/auth/login/google', { method: 'POST' });`
	- Backend는 1) Code Verifier 및 Code Challenge를 생성한다, 2) Client ID, Frontend Redirect URI, Scope, State, Nonce, Code Challenge, Code Challenge Method 등을 활용하여 OAuth Authorization URL을 생성하여 Frontend로 반환한다.
		- Example
	```json
	{
		"authURL": "https://kauth.kakao.com/oauth/authorize?response_type=code&client_id=dfb1e25a2b97d03b0b225d4874a34823&redirect_uri=http://localhost:5173/signin?provider=kakao&scope=openid&state=276795850273511360818854556981559492420&nonce=e9e62073a211c8c9cf23d1e41a4182b7&code_challenge=6rUx4nIA1D1V51T8yOiQwlq0Y-h9SZIv2ZlrtcGK_0Y&code_challenge_method=S256",
	}
	```
	- Frontend는 해당 OAuth Authorization URL popup 화면(OAuth Consent Screen)을 띄운다.
	- 사용자는 OAuth 로그인을 진행한다.
	- 성공하면 OAuth Provider는 Frontend Redirect URI로 Authorization Code, State을 보낸다.
	- Frontend는 받은 Authorization Code, State을 Backend로 보낸다.
	- Backend는 받은 State이 처음에 보낸 State과 동일한지 확인한다.
	- Backend는 Authorization Code와 Code Verifier를 OAuth Provider로 보낸다.
	- OAuth Provider는 받은 Code Verifier를 Code Challenge Method로 해싱한 값이 이전에 받았던 Code Challenge와 동일한지 verify한 후 ID Token 및 Access Token을 반환한다.
	- Backend는 받은 ID Token과 Nonce 값을 verify 한 후 Frontend로 로그인 응답을 한다.
	- Frontend는 성공적으로 로그인된 화면을 보여준다.
### Illustration
<div align="center">
	<img src="https://raw.githubusercontent.com/fine-ants/obsidian/main/FE/Dev%20Log/Auth/refImg/strategy-3.png" alt="OAuth Auth Code Grant" width="80%" />
</div>

## FineAnts가 지원하는 OAuth Login
### Google
- Google Identity Services (Sign In With Google for Web)
	- Google의 OAuth 2.0을 기반하는 authentication 및 authorization을 한 패키지로 모아둔 SDK.
		- Authentication "순간"은 One Tap, automatic sign-in, Sign In With Google button을 제공한다.
			- 이 방식들은 ID Token만을 반환할 수 있고, OpenID Connect spec을 따른다.
			- 즉, Sign In With Google 방식들은 기본적으로 ID Token을 반환한다.
		- Authorization "순간"은 후에 Google의 Resource Server로부터 데이터 접근이 필요할 때 실행한다.
			- 이는 code 또는 Access Token만을 반환할 수 있다.
			- FineAnts에는 불필요한 부분이다.
	- Reference
		- [Overview  |  Authentication  |  Google for Developers](https://developers.google.com/identity/gsi/web/guides/overview#compare_to_oauth_and_openid_connect)
#### 참고
- Google은 직접적입 Google API 호출보다 해당 SDK 사용을 권장한다.
- Frontend 코드에 Client ID를 포함해야한다.
	- Client ID는 OAuth Client의 공개 식별자로서 OAuth Provider가 OAuth Client을 식별할 수 있도록 한다.
	- Client ID가 frontend code에 노출이 되지만 PKCE를 통해 보완한다.
- Backend 코드에 Client Secret이 숨겨져 있다.
	- Client Secret은 OAuth Client의 "인증서"로서 OAuth Provider가 OAuthClient를 인증할 수 있도록 한다.
	- 이 Client Secret을 Google Authorization Server로 authentication 요청과 함께 보내야지만 성공적으로 OAuth 인증이 이루어진다.
#### 전략
- **대안 3**
	- Sign in With Google SDK 미사용으로 Frontend에 Client ID를 포함하지 않는다.
### Kakao
- Kakao는 OpenID Connect와 PKCE를 지원한다.
- Reference
	- [[공지] 카카오 로그인 OpenID Connect 지원 / [Notice] Support of OpenID Connect - Notice / 공지 - 카카오 데브톡](https://devtalk.kakao.com/t/openid-connect-notice-support-of-openid-connect/121888)
#### 전략
- **대안 3**
### Naver
- Naver는 OpenID Connect 및 Authorization Code Grant with PKCE를 지원하지 않는다.
- 기본 Authorization Code Grant만 가능하다.
- `state` parameter를 이용할 수 있다.
#### 전략
- **기존 전략 + Authorization URL을 Backend로부터 받아는 방식.**
- Backend로부터 Auth URL을 먼저 받고 요청을 하는 방식이기 때문에 SDK 미사용.
	- SDK 사용은 Client에서 바로 OAuth Provider의 Auth URL로 요청이 가기 때문이다.

## 기타 보안 내용
### Authorization Code Replay Attack ft. PKCE
- a.k.a. Authorization Code Injection
#### Authorization Code Replay Attack
- Authorization Code Replay 공격이란, 유효한 Authorization Code의 무단 재사용을 의미한다.
- Authorization Code Replay 공격자의 목적은 탈취한 Authorization Code로 OAuth Provider로부터 토큰을 발급받는 것이다.
- Authorization Code Replay 공격 방어란, Authorization Code을 요청한 Client와 Authorization Code을 토큰으로 교환 요청을 하는 Client가 동일한지 확인하는 것이 목적이다.
#### Proof Key for Code Exchange(PKCE) ft. Code Verifier, Code Challenge
- PKCE란, OAuth 2.0 Authorization Code Grant 흐름에서 Authorization Code 탈취에 대한 문제를 보완하기 위해 나온 장치다.
- Code Verifier는 랜덤한 고유 값이다.
- Code Challenge는 Code Verifier를 단방향 해싱을 활용하여 변형한 값이다.
- 매 Authorization Code 요청마다 고유의 값을 갖는다.
- *OAuth Client가 Code Verifier 및 Code Challenge을 생성하고 OAuth Provider가 검증한다.*
##### 흐름
- OAuth Client는 secret(Code Verifier)와 해당 secret의 변형된 값(Code Challenge)을 생성한다.
- OAuth Client는 OAuth Provider로부터 Authorization Code을 받기 위한 요청에 Code Challenge을 같이 보낸다.
- OAuth Client가 성공적으로 Authorization Code을 받으면, Authorization Code와 Code Verifier를 OAuth Provider로 보낸다.
- OAuth Provider는 Code Verifier를 이전 단계에서 받은 Code Challenge을 이용하여 verify한다.
	- **해당 요청이 Authorization Code을 요청한 클라이언트와 동일한지 확인.**
- Code Verifier가 성공적으로 verify가 되었다면 OAuth Provider는 Access Token과 Refresh Token을 반환한다.
#### Reference
- [RFC 7636 - Proof Key for Code Exchange by OAuth Public Clients](https://datatracker.ietf.org/doc/html/rfc7636)
- [draft-ietf-oauth-security-topics-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-11#section-2.1.1)
### Cross Site Request Forgery(CSRF) ft. `state`
#### CSRF
- CSRF 공격이란, 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위(수정, 삭제, 등록 등)를 특정 웹사이트에 요청하게 하는 공격을 말한다.
- CSRF 공격자는 위조된 요청에 대한 응답을 확인 할 수 없다.
- CSRF 공격자의 목적은 사용자가 승인한 웹사이트로의 위조된 요청을 보내는 것이다.
- CSRF 공격 방어란, Auth Request와 Response을 binding하는 것이다.
	- i.e. OAuth Client가 보낸 Auth Request과 OAuth Provider가 보낸 Auth Response간의 세션을 유지한다.
#### `state` Parameter
- a.k.a. CSRF Token
- 매 요청마다 고유의 값을 갖는 `state` parameter를 활용하여 CSRF 공격을 방어할 수 있다.
- *OAuth Client가 `state`을 생성 및 검증한다.*
- 사용자가 보낸 Auth Request가 맞다면 Auth Response에서 받은 `state` 값이 일치해야 하는데, 내가 모르게 CSRF 공격자에 의해 Auth Request를 보냈다면 `state` 값이 일치하지 않을 것이다.
##### 흐름
- OAuth Client는 Authorization Code 요청을 할 때 `state` parameter를 포함하여 보낸다.
- OAuth Provider는 Redirect URI로 Authorization Code와 받은 `state` parameter를 그대로 보낸다.
- OAuth Client는 받은 `state` 값이 Authorization Code 요청을 할 때 보낸 `state` 값과 일치하는지 확인한다.
	- *i.e. Authorization Response가 조작되지 않고 다른 서버가 아닌 자신이 Authorization Code 요청을 보낸 서버로부터 온게 맞는지 확인한다.*
	- `state` 값이 다르다면, 자신이 보낸 요청에 대한 응답이 아니거나 OAuth Provider의 응답을 위조했다는 뜻이다.
#### Reference
- [RFC 6819 - OAuth 2.0 Threat Model and Security Considerations](https://datatracker.ietf.org/doc/html/rfc6819#section-4.4.1.8)
- [Prevent Attacks and Redirect Users with OAuth 2.0 State Parameters](https://auth0.com/docs/secure/attack-protection/state-parameters)
### ID Token Replay Attack ft. `nonce`

#### ID Token Replay Attack
- ID Token Replay 공격이란, 유효한 ID Token의 무단 재사용을 의미한다.
- ID Token Replay 공격자의 목적은 탈취한 ID Token을 활용하여 OAuth Client와 인증을 하는 것이다.
- ID Token Replay 공격 방어란, OAuth Client와 ID Token을 binding하여 세션을 유지하여 replay 공격을 방어한다.
	- 일회용 값을 사용하여 한번 인증에 사용한 ID Token을 무효화하여 ID Token을 재사용하여 인증을 하는 것을 방지한다.
#### `nonce` Parameter
- a.k.a. "number used once"
- *OAuth Client가 `nonce`을 생성 및 검증한다.*
- Implicit Grant에서는 `nonce` parameter가 필수다.
- Authorization Code Grant에서는 `nonce` parameter를선택적으로 적용할 수 있다.
- 한번 검증이 된 `nonce` 값은 더 이상 유효하지 않기 때문에 해커가 ID Token을 탈취하더라도 
##### 흐름
- OAuth Client는 Authorization Request에 `nonce` parameter를 추가하여 요청을 보낸다.
- OAuth Provider는 받은 `nonce`을 그대로 ID Token에 포함해서 응답한다.
- OAuth Client는 ID Token에 들어있는 `nonce` 값이 Authorization Request에 보냈던 `nonce` 값과 동일한지 확인한다.
	- ID Token에 들어있는 `nonce` 값이 처음 Auth Request에 보낸 `nonce` 값과 다를 경우, 해당 ID Token은 유효하지 않다.
	- OAuth Client는 `nonce` 확인을 하기 전까지 발급 받은 token을 사용하지 말아야 한다.
#### Reference
- [Final: OpenID Connect Core 1.0 incorporating errata set 1](https://openid.net/specs/openid-connect-core-1_0.html#IDToken)
- [draft-ietf-oauth-security-topics-24](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#nonce_as_injection_protection)
- [Difference between OAuth 2.0 "state" and OpenID "nonce" parameter? Why state could not be reused? - Stack Overflow](https://stackoverflow.com/questions/46844285/difference-between-oauth-2-0-state-and-openid-nonce-parameter-why-state-cou)
- [security - what is the case of replay attack for which OIDC nonce is a protection - Stack Overflow](https://stackoverflow.com/questions/77250027/what-is-the-case-of-replay-attack-for-which-oidc-nonce-is-a-protection)
- [OAuth Replay Attack Mitigation. When working with developers on… | by Ben Botto | Medium](https://medium.com/@benjamin.botto/oauth-replay-attack-mitigation-18655a62fe53)
