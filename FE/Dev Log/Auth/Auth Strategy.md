

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
### 대안
#### OAuth 2.0 Authorization Code Flow with PKCE
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
###### Reference
- [RFC 7636 - Proof Key for Code Exchange by OAuth Public Clients](https://datatracker.ietf.org/doc/html/rfc7636)
- [draft-ietf-oauth-security-topics-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-11)
### 대안 문제점
- PKCE 지원
	- Google

- FineAnts는 OAuth을 인가 목적이 아닌 인증 목적이다.




