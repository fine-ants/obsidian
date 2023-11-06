

## Table of Contents
- [[#현재 구현 방식]]

## 현재 구현 방식
- Client(SPA)에서 시작하는 기본 OAuth 2.0 Authorization Code Flow
### 문제점
#### SPA와 OAuth 2.0 Authorization Code Flow의 문제
- Client ID, Client Secret, Authorization Code(인가코드)가 노출된다.
	- 즉, 해커가 해당 정보를 탈취할 수 있다.
##### Reference
[Authorization Code Flow with Proof Key for Code Exchange (PKCE)](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-proof-key-for-code-exchange-pkce)
#### OAuth 2.0 Authorization Code Flow with PKCE
- 이를 보완하기 위해 OAuth 2.0은 






### 대안

