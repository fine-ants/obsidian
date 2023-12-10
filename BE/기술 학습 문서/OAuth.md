## Authorization Code Flow
인가 코드 플로우([OAuth 2.0 RFC 6749, section 4.1](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1)에 정의)는 토큰을 위해서 인가 코드를 교환하는 것과 관련이 있습니다.

인가 코드 플로우는 오직 신뢰있는 일반 적인 웹 애플리케이션과 같은 애플리케이션들에만 사용될 수 있습니다. 왜냐하면 애플리케이션의 인증 방법들이 토큰 교환에 포함되어 있고 보안을 유지해야 하기 때문입니다.

### 인가 코드 플로우(Authorization Code Flow)는 어떻게 작동하는가?
![[Pasted image 20231208122157.png]]

1. 사용자가 애플리케이션에 있는 로그인 링크를 클릭합니다.
2. Auth0의 SDK은 Auth0 인가 서버로 사용자를 리다이렉트 시킵니다. (/authorize 요청)
3. Auth0 인가 서버는 로그인과 인가 프롬프트를 위해서 사용자를 리다이렉트 시킵니다. 
4. 사용자는 설정된 로그인 옵션중 하나를 선택하여 인증합니다. 그리고 Auth0가 애플리케이션에 부여할 권한을 나열하는 동의 프롬프트를 볼 수 있습니다.
5. Auth0 인가 서버는 사용자를 다시 애플리케이션에 인가코드와 같이 리다이렉트 시킵니다.
6. Auth0의 SDK는 인가 코드(Authorization Code), 애플리케이션의 클라이언트 아이디(client-id), 클라이언트 시크릿(client-secret)이나 Private Key JWT와 같은 인증 정보들을 Auth0 인가 서버에 전송합니다. (/oauth/token endpoint
7. Auth0 인가 서버는 인가 코드, 애플리케이션의 클라이언트 아이디, 애플리케이션 인증 정보들을 검증합니다.
8. Auth0 인가 서버는 ID 토큰과 Access Token(선택적으로 Refresh Token)을 응답합니다.
9. 애플리케이션은 사용자에 대한 정보에 접근하기 위해 API를 호출하는데 Access Token을 사용할 수 있습니다.
10. API는 요청된 데이터를 응답합니다.

### 어떻게 인가 코드 플로우를 구현하는가?
인가 코드 플로우를 구현하는 가장 쉬운 방법은 [Regular Web App QuickStarts](https://auth0.com/docs/quickstart/webapp)를 따라가는 것입니다.

대안으로 인가 코드 플로우를 구현하기 위해서 Authentication API를 사용할 수 있습니다. 더 많은 정보를 얻기 위해서는 [Add Login Using the Authorization Code Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/add-login-auth-code-flow) 또는 [Call Your API Using the Authorization Code Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/call-your-api-using-the-authorization-code-flow)를 읽어보세요.

## Learn more

- [Auth0 Rules](https://auth0.com/docs/customize/rules)
- [Auth0 Hooks](https://auth0.com/docs/customize/hooks)
- [Tokens](https://auth0.com/docs/secure/tokens)
- [Token Best Practices](https://auth0.com/docs/secure/tokens/token-best-practices)
- [Which OAuth 2.0 Flow Should I Use?](https://auth0.com/docs/get-started/authentication-and-authorization-flow/which-oauth-2-0-flow-should-i-use)

### References
- [Authorization Code Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow#learn-more)

### 관련 링크
- [[BE/기술 학습 문서/OAuth#Tokens]]

---

## Tokens
식별과 관련된 토큰은 두가지 타입이 존재합니다. : ID tokens과 Access Tokens

### ID Tokens
ID tokens은 오직 애플리케이션에서만 사용할 수 있는 JSON Web Token(JWT)입니다. 만약에 달력을 동기화 하고, 구글을 이용하여 사용자들을 로그인시키는 애플리케이션이 있다면, 구글은 사용자의 정보를 저장한 ID Token을 애플리케이션에 전송할 것입니다. 애플리케이션은 그런 다음 토큰의 내용을 추출하고 사용자에게 개인적인 서비스를 제공하기 위해서 정보(이름과 프로파일 사진과 같은)들을 사용합니다. 

단, 포함된 정보를 사용하기 전에 [ID Token의 유효성을 확인](https://auth0.com/docs/secure/tokens/id-tokens/validate-id-tokens)해야 합니다. [라이브러리](https://jwt.io/libraries)를 사용하여 이 작업을 수행할 수 있습니다.

API에 대한 접근 권한을 얻기 위해서 ID Token을 사용하지 마세요. 각 토큰은 의도된 audience(보통, 수신자)에 대한 정보를 포함하고 있습니다. OpenID Connection 스펙에 따르면 ID Token의 audience(aud claim을 가리키는)는 인증 요청을 만드는 애플리케이션의 클라이언트 아이디가 되어야 합니다. 만약 이 경우에 해당되지 않으면, 여러분들은 토큰을 신뢰하면 안됩니다.

ID Token의 디코딩된 내용은 다음과 같을 수 있습니다.
```
{
	"iss": "http://{yourDomain}/", 
	"sub": "auth0|123456", 
	"aud": "{yourClientId}", 
	"exp": 1311281970, 
	"iat": 1311280970, 
	"name": "Jane Doe", 
	"given_name": "Jane", 
	"family_name": "Doe", 
	"gender": "female", 
	"birthdate": "0000-10-31", 
	"email": "janedoe@example.com", 
	"picture": "http://example.com/janedoe/me.jpg" 
}
```

이 토큰은 애플리케이션에 사용자를 인증합니다. 토큰의 audience(aud claim)은 오직 이 명세한 애플리케이션만이 ID Token을 소비해야 된다는 것을 의미하는 애플리케이션의 식별자로 설정됩니다. 

반대로 API는 aud 값을 가진 토큰이 API의 유니크한 식별자와 동일할 것으로 예상합니다. 그러므로 여러분들이 애플리케이션과 API 둘다 제어를 유지하지 않는한 일반적으로 API에 ID Token을 보내는 것은 작동하지 않을 것입니다. 왜냐하면 ID Token은 API에 의해서 서명되지 않았기 때문에 만약 API가 ID Token을 수락한다면 애플리케이션이 토큰을 수정(ex, 더 많은 범위 추가)했는지 알 수 있는 방법이 없을 것입니다. 자세한 정보는 [JWT Handbook](https://auth0.com/resources/ebooks/jwt-handbook)을 참고해주세요.


### Access Tokens
Access Token(JWT가 항상 있는 것은 아님)은 토큰의 베어러(Bearer)가 API에 접근할 수 있는 권한을 부여받았음을 API에 알리는데 사용됩니다. (부여된 범위에 의해 지정됨)

위의 구글 예제에서 구글은 사용자가 로그인 한 후에 앱에 Access Token을 전송하고 애플리케이션이 구글 캘린더에 읽고 쓰는것에 대한 동의를 제공합니다. 애플리케이션이 구글 캘린더에 글을 쓰고 싶을 때마다 HTTP Authorization 헤더에 있는 Access Token을 포함한 요청을 구글 캘린더 API에 전송합니다.

**Access Token은 [인증(authentication)](https://auth0.com/docs/authenticate)에 절대 사용되면 안됩니다.** Access Token은 사용자가 인증되었는지 여부를 알 수 없습니다. Access Token이 가지고 있는 유일한 사용자 정보는 하위 클레임에 있는 사용자 ID입니다. 애플리케이션에서 Access Token은 API 용이므로 불투명 문자열로 취급합니다. 여러분들의 애플리케이션은 특정 형식으로 토큰을 디코딩하려고 시도하거나 토큰을 수신하기를 기대해서는 안됩니다.

여기에 Access Token에 대한 예제가 있습니다.
```
{ 
	"iss": "https://{yourDomain}/", 
	"sub": "auth0|123456", 
	"aud": [ "my-api-identifier", "https://{yourDomain}/userinfo" ], 
	"azp": "{yourClientId}", 
	"exp": 1489179954, 
	"iat": 1489143954, 
	"scope": "openid profile email address phone read:appointments" 
}

```

토큰에는 사용자의 ID(서브 클레임) 외에 사용자에 대한 정보가 포함되어 있지 않습니다. 애플리케이션이 API에서 수행할 수 있는 작업(scope claim)에 대한 권한 정보만 포함되어 있습니다. 이것이 API 보안에는 유용하지만, 사용자 인증에는 유용하지 않습니다.

일부 상황에서는 API가 사용자에 대한 자세한 정보를 가져오기 위해 추가 작업을 수행해야 하는 것을 방지하기 위해 Access Token에 사용자 또는 하위 클레임 외에 다른 사용자 지정 클레임에 대한 추가 정보를 넣는 것이 바람직할 수 있습니다. 이렇게 선택하는 경우, 이러한 추가 클레임이 Access Token에서 읽을 수 있음을 명심하세요. 자세한 내용은 [Create Custom Claims](https://auth0.com/docs/secure/tokens/json-web-tokens/create-custom-claims)을 읽어주세요.

### Specialized tokens
Auth0의 토큰 기반 인증 시나리오에서 사용되는 3개의 특별한 토큰이 있습니다.
- Refresh Tokens : 사용자를 재인증 할 필요없이 갱신된 Access Token을 얻는 데 사용되는 토큰입니다.
- IDP(Identity Provider) access tokens : 사용자 인증 후 identity provider가 발행한 토큰에 접근하여 서드 파티 API를 호출하는데 사용할 수 있습니다.
- Auth0 Management API access tokens : Management API 엔드 포인트를 호출하기 위해서 여러분들에게 허용하는 특정 클레임(scope)을 포함되는 수명이 짧은 토큰입니다.

## Learn more

- [JSON Web Tokens](https://auth0.com/docs/secure/tokens/json-web-tokens)
- [ID Tokens](https://auth0.com/docs/secure/tokens/id-tokens)
- [Access Tokens](https://auth0.com/docs/secure/tokens/access-tokens)
- [Refresh Tokens](https://auth0.com/docs/secure/tokens/refresh-tokens)
- [Token Storage](https://auth0.com/docs/secure/security-guidance/data-security/token-storage)
- [Token Best Practices](https://auth0.com/docs/secure/tokens/token-best-practices)

### References
- [Tokens](https://auth0.com/docs/secure/tokens#learn-more)

---

## Which OAuth 2.0 Flow Should I Use? (어떤 OAuth 2.0 Flow를 사용해야 하는가?)
[OAuth 2.0 Authorization Framework](https://auth0.com/docs/authenticate/protocols/oauth)는 여러개의 다른 흐름들 또는 승인(grant)을 지원합니다. Access Token을 발급하는 여러개의 Flow가 있습니다. 사용 사례에 적합한 것을 결정하는 것은 여러분들의 애플리케이션의 타입에 따라 다르지만, 클라이언트에 대한 신뢰 수준이나 사용자가 원하는 경험과 같은 다른 매개변수도 고려됩니다.

### OAuth 2.0 terminology
- Resource Owner : 보호된 리소스에 접근하는 것을 승인해주는 사용자, 일반적으로 이 리소스는 최종 사용자의 정보입니다.
- Client : 리소스 소유자를 대신하여 보호된 리소스에 접근을 요청하는 애플리케이션
- Resource Server : 보호된 리소스들을 호스팅하는 서버. 이 서버는 여러분들이 접근하기 위해 원하는 API입니다.
- Authorization Server : Resource Owner를 인증하고 적절한 권한을 부여받은 후 Access Token을 발급하는 서버, 이경우에는 OAuth0가 해당됩니다.
- User Agent : Client와 같이 상호작용하기 위해 Resource Owner에 의해 사용되는 에이전트, 예를 들어 브라우저나 네이티브 애플리케이션이 해당됩니다.

### Is the Client the Resource Owner? (클라이언트가 리소스 소유자인가?)
첫번째 결정 포인트는 리소스들에 접근을 요구하는 서드파티가 머신인지 아닌지에 대한 것입니다. 머신 대 머신 인가의 사용의 경우에는 Client 또한 Resource Owner이기도 하므로 최종 사용자 권한이 필요하지 않습니다. API를 사용하여 데이터베이스로 정보를 가져오는 cron 작업이 그 예시입니다. 이 예에서 cron 작업은 client-id와 client-secret을 갖고 있고 이 정보들을 사용하여 인증 서버에서 액세스 토큰을 가져오기 때문에 Client 및 Resource Owner입니다.

> [!info]
> database cron working
> 반복작인 테스크를 자동화하기 위해서 여러분들의 데이터베이스에 procedure 또는 command를 스케줄링하는 프로세스


### Is the Client a web app executing on the server? (클라이언트가 서버에서 실행해하는 웹 애플리케이션인가?)
클라이언트가 서버에서 실행하는 일반적인 웹 애플리케이션 인경우 **Authorization Code Flow**를 사용해야 합니다. 이를 사용하여 클라이언트는 Access Tokenr과 Refersh Token을 발급받을 수 있습니다. 이 플로우는 안전한 선택으로 고려됩니다. 왜냐하면 Access Token이 노출되는 리스크와 사용자의 웹 브라우저를 통해 전달되는 것 없이 클라이언트를 호스팅하는 웹 서버로 직접적으로 전달되기 때문입니다. 

만약 이 케이스가 여러분들의 필요에 맞는다면 이 플로우가 어떻게 작동하고 어떻게 구현하는지 [Authorization Code Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow) 참고해주세요.

### Is the Client absolutely trusted with user credentials? (클라이언트가 사용자 인증정보로 절대적으로 신뢰받는가?)
이 결정 포인트로 인해 Resource Owner Password Credentials Grant가 발생할 수 있습니다. 이 플로우에서는 최종 사용자는 로그인 화면을 사용하여 인증정보(사용자이름/패스워드)를 입력하는 것을 요청받습니다. 이 정보는 백엔드로 전송되고 백엔듯에서 OAuth0 서버로 전송됩니다. 따라서 클라이언트가 이 정보를 절대적으로 신뢰해야 합니다.

이 승인 부여 플로우는 리다이렉트 기반 플로우들이 가능하지 않을때 사용되어야 합니다. 만약 이 케이스가 여러분들의 케이스라면, 이 플로우가 어떻게 작동하고 어떻게 구현되는지 [Resource Owner Password Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/resource-owner-password-flow)를 참고해주세요.

### Is the Client a Single-Page App? (클라이언트 싱글 페이지 애플리케이션인가?)
만약 클라이언트가 자바스크립트와 같은 스크립트 언어를 사용하여 브라우저에 실행하는 애플리케이션인 싱글 페이지 애플리케이션(Single-Page Application, SPA)라면, 두가지 승인 옵션이 있습니다. Proof Key for Code Exchange를 가진 Authorization Code Flow와 Form Post를 가진 묵시적인 옵션이 있습니다. 대부분의 경우에는 우리는 PKCE를 이용한 Authorization Code Flow를 사용하는 것을 권장합니다. 왜냐하면 액세스 토큰은 클라이언트 사이드에 노출되지 않기 때문입니다. 그리고 이 플로우는 [Refresh Tokens](https://auth0.com/docs/secure/tokens/refresh-tokens)을 반환할 수 있습니다.

이 플로우가 어떻게 작동하고 구현하는지 더 보기 위해서는 [Authorization Code Flow with Proof Key for Code Exchange (PKCE)](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-proof-key-for-code-exchange-pkce)을 참고해주세요. [Auth0 Single-Page App SDK](https://auth0.com/docs/libraries/auth0-single-page-app-sdk)는 SPA에서 PKCE를 가진 Authorization Code Flow를 구현하기 위한 높은 레벨의 API를 제공합니다.

만약 여러분들의 SPA가 액세스 토큰이 필요하지 않다면, 여러분들은 Form post를 가진 묵시적 플로우를 사용할 수 있습니다. Form post를 가진 묵시적 플로루가 어떻게 작동하고 구현하는지 알기 위해서 [Implict Flow with Form Post](https://auth0.com/docs/get-started/authentication-and-authorization-flow/implicit-flow-with-form-post)_를 참고해주세요.

### Is the Client a Native/Mobile App? (클라이언트가 네이티브/모바일 앱인가?)
만약 애플리케이션이 네이티브 앱이라면, **Authorization Code Flow with Proof Key for Code Exchange (PKCE)**를 사용하세요.

어떻게 작동하고 어떻게 구현하는지 보기 위해서는 [Authorization Code Flow with Proof Key for Code Exchange (PKCE)](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-proof-key-for-code-exchange-pkce).를 참고해주세요.

### I have an application that needs to talk to different resource servers (다른 리소스 서버와 통신해야 하는 애플리케이션이 있습니다.)
만약 싱글 애플리케이션이 다른 리소스 서버들에 대하여 액세스 토큰이 필요하다면, 여러개의 `/authorize`(같거나 서로 다른 Authorization Flow 실행) 콜이 수행될 필요가 있습니다. 각 인증은 audience에 대해서 다른 값을 사용하므로 플로우의 끝에서 다른 액세스 토큰이 생성됩니다. 더 많은 정보를 보기 위해서는 [OAuth 2.0: Audience Information Specification](https://tools.ietf.org/html/draft-tschofenig-oauth-audience-00#section-3).를 참고해주세요.

## Can I try the endpoints before I implement my application? (내가 나의 애플리케이션을 구현하기 전에 엔드포인트들에 시도할 수 있는가?)
물론이다! 여러분들은 우리의 [Authentication API Debugger Extension](https://auth0.com/docs/customize/extensions/authentication-api-debugger-extension)을 사용할 수 있다. 우리의 [Authentication API Reference](https://auth0.com/docs/api/authentication).에서 매 `/grant` 엔드포인트에 대한 자세한 정보를 찾아볼 수 있습니다. 
- Authorize 엔드포인트를 위하여 [Authorize Application](https://auth0.com/docs/api/authentication#authorize-application)를 가고 여러분들이 테스트하기를 원하는 승인 플로우를 위해서 "Test this endpoint" 문단을 읽어보세요.
- Token 엔드포인트를 위하여 [Get Token](https://auth0.com/docs/api/authentication#get-token)을 가고 여러분들이 테스트 하기를 원하는 승인 플로우에 대한 "Test this endpoint"를 읽어보세요.


### References
- [Which OAuth 2.0 Flow Should I Use?](https://auth0.com/docs/get-started/authentication-and-authorization-flow/which-oauth-2-0-flow-should-i-use#can-i-try-the-endpoints-before-i-implement-my-application-)

---

## Authorization Code Flow with Proof Key for Code Exchange (PKCE)
### 개요
주요 개념
- OAuth 2.0 grant type, Authorization Code Flow with Proof Key for Code Exchange (PKCE)에 대해서 알아보기.
- 네이티브 또는 싱글 페이지 앱과 같은 client secret을 저장할 수 없는 애플리케이션을 위하여 이 grant type 사용하기
- Auth0 SDK를 가지고 서로 다른 구현 방법들을 리뷰하기

공개 클라이언트(public client)가(네이티브와 싱글 페이지 애플리케이션) 액세스 토큰을 요청할 때, Authorization Code Flow만으로는 완화되지 않은 몇몇 추가적인 보안 문제가 제기됩니다. 이것은 다음과 같은 이유 때문입니다.

**Native apps**
- Client Secret을 보안적으로 저장할 수 없습니다. 앱을 디컴파일하면 앱에 바인딩 되어 있으며 모든 사용자와 장치에서 동일한 Client Secret이 표시됩니다.
- 커스텀 URL 스키마를 사용하여 리다이렉션(ex, MyApp://)을 캡처하여  악의적인 애플리케이션이 인증 서버로부터 Authorization Code를 받을 수 있습니다.

**Single-page apps**
- 전체 소스 코드가 브라우저에서 이용가능하면 Client Secret을 안전하게 저장할 수 없기 때문입니다.

이러한 상황이 주어졌을 때, OAuth 2.0은 Proof Key for Code Exchange (PKCE)을 사용하는 Authorization Code Flow 버전을 제공합니다. ([OAuth 2.0 RFC 7636](https://tools.ietf.org/html/rfc7636) 정의)

PKCE가 강화한 Authorization Code Flow는 Authorization Server에서 확인할 수 있는 sercret을 소개합니다. 이 secret 정보는 **Code Verifier**라고 불립니다. 추가적으로 호출하는 애플리케이션은 Code Challenge라고 불리는 Code Verifier의 변환된 값을 생성하고 Authorization Code를 발급하기 위해서 HTTPS를 통해서 Code Challenge 값이 전송됩니다. **이 방법은 악의적인 공격자가 오직 Authorization Code를 가로챌수는 있지만, Code Verifier없이 토큰을 교환할 수 없습니다.**

### How it works(PKCE가 적용된 Authorization Code Flow는 어떻게 작동하는가?)
PKCE가 강화한 Authorization Code Flow는 [표준 Authorization Code Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow) 기반으로 지어졌기 때문에 단계들이 매우 비슷합니다.

![[Pasted image 20231209140847.png]]

1. 사용자는 애플리케이션에 있는 로그이 버튼을 클릭합니다.
2. Auth0의 SDK는 암호학적으로 무작위한 code verifier를 만들고 이로부터 code challenge를 생성합니다.
3. Auth0의 SDK는 code challenge와 함께 Auth0 Authorization Server로 사용자를 리다이렉션 시킵니다. (/authorize 엔드포인트)
4. 여러분들의 Auth0 Authorization Server는 로그인 및 인증 프롬프트 화면으로 사용자를 리다이렉션 시킵니다.
5. 사용자는 설정된 로그인 옵션들 중 하나를 선택하여 인증하고 Auth0가 애플리케이션에게 권한을 주는 리스팅을 제공하는 페이지를 볼 수 있습니다.
6. 여러분들의 Auth0 Authorization Server는 code challenge를 저장하고 사용자를 authorization code를 가지고 애플리케이션에 다시 리다이렉션 시킵니다. 
7. Auth0의 SDK는 authorization code와 code verifier(2단계에서 생성한 데이터)를 Auth0 Authorization Server에 전송합니다.(`/oauth/token` 엔드포인트)
8. 여러분들의 Auth0 Authorization Server는 code challenge와 code verifier를 확인합니다.
9. 여러분들의 Auth0 Authorization Server는 ID Tokens과 액세스 토큰을 같이 응답합니다. (선택적으로 refresh token도 포함될 수 있음)
10. 여러분들의 애플리케이션은 액세스 토큰을 사용자에 대한 정보를 가져오기 위해서 API 콜하는데 사용할 수 있습니다.
11. API는 요청된 정보를 응답합니다.

만약 여러분들이 Refresh Token Rotation을 활성화했다면, 새로운 Refresh Token은 매 요청마다 생성되고 Access Token을 가지고 발급될 것입니다. Refresh Token이 교환될 때, 이전 Refresh Token은 유효하지 않게 됩니다. 그러나 관계에 대한 정보는 authorization server에 의해서 유지될 것입니다.

### How to implement it(PKCE가 강화한 Authorization Code Flow는 어떻게 구현하는가?)
PKCE가 강화한 Authorization Code Flow를 구현하는 가정 쉬운 방법은 [follow our Native Quickstarts](https://auth0.com/docs/quickstart/native) 또는[follow our Single-Page Quickstarts](https://auth0.com/docs/quickstart/spa).를 하는 것입니다.

**Mobile**
- [Auth0 Swift SDK](https://auth0.com/docs/libraries/auth0-swift)
- [Auth0 Android SDK](https://auth0.com/docs/libraries/auth0-android)

**Single-page**
- [Auth0 Single-Page App SDK](https://auth0.com/docs/libraries/auth0-single-page-app-sdk)
- [Auth0 React SDK](https://auth0.com/docs/libraries/auth0-react)

여러분들은 [Add Login Using the Authorization Code Flow with PKCE](https://auth0.com/docs/get-started/authentication-and-authorization-flow/add-login-using-the-authorization-code-flow-with-pkce)또는 [Call Your API Using the Authorization Code Flow with PKCE](https://auth0.com/docs/get-started/authentication-and-authorization-flow/call-your-api-using-the-authorization-code-flow-with-pkce).을 참고하여 튜토리얼을 해볼 수 있습니다.

## Learn more
- [Auth0 Rules](https://auth0.com/docs/customize/rules)
- [Auth0 Hooks](https://auth0.com/docs/customize/hooks)
- [Tokens](https://auth0.com/docs/secure/tokens)
- [Token Best Practices](https://auth0.com/docs/secure/tokens/token-best-practices)
- [Which OAuth 2.0 Flow Should I Use?](https://auth0.com/docs/get-started/authentication-and-authorization-flow/which-oauth-2-0-flow-should-i-use)

---

## Add Login Using the Authorization Code Flow with PKCE
여러분들은 PKCE가 강화한 Authorization Code Flow를 사용하여 네이티브, 모바일 또는 싱글 페이지 애플리케이션에 로그인을 추가할 수 있습니다. 이 인증 코드 플로우가 어떻게 작동하고 왜 사용해야 하는지 배우기 위해서는 [Authorization Code Flow with Proof Key for Code Exchange (PKCE)](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-proof-key-for-code-exchange-pkce).을 참고해주세요. 네이티브, 모바일 또는 싱글 페이지 애플리케이션에서 API 호출 방법을 알아보려면  [Call Your API Using Authorization Code Flow with PKCE](https://auth0.com/docs/get-started/authentication-and-authorization-flow/call-your-api-using-the-authorization-code-flow-with-pkce).을 읽어주세요.

PKCE Authorization Code Flow를 구현하기 위해서는 다음과 같은 리소스들을 사용할 수 있습니다.
- [Auth0 Mobile SDKs](https://auth0.com/docs/libraries) 및 [Auth0 Single-Page App SDK](https://auth0.com/docs/libraries/auth0-single-page-app-sdk): 플로우를 구현하는 가장 쉬운 방법이며, 대부분 무거운 리프팅을 수행할 것입니다. 우리의 [Mobile Quickstarts](https://auth0.com/docs/quickstart/native)와  [Single-Page App Quickstarts](https://auth0.com/docs/quickstart/spa)가 프로세스를 안내해 드립니다.
- [Authentication API](https://auth0.com/docs/api/authentication) : 만약 여러분들이 여러분들의 소유의 솔루션을 빌드하는 것을 선호한다면, 우리의 API를 직접적으로 호출하는 방법을 읽어보세요.

성공적인 로그인 다음에 여러분들의 애플리케이션은 사용자의 ID Token과 Access Token에 접근할 것입니다. ID Token은 기본적인 사용자 프로필 정보를 포함할 것입니다. 그리고 Access Token은 Auth0 `/userinfo` 엔드포인트를 호출 또는 자신의 보호된 API를 호출하는데 사용될 수 있습니다. ID Token에 대한 것을 더 배우기 위해서는 [ID Tokens](https://auth0.com/docs/secure/tokens/id-tokens)을 읽으세요. Access Token에 대해서 더 배우기 위해서는  [Access Tokens](https://auth0.com/docs/secure/tokens/access-tokens).을 읽으세요.

### Prerequisites
Auth0에 여러분들의 애플리케이션을 등록하세요. 더 배우기 위해서는  [Register Native Applications](https://auth0.com/docs/get-started/auth0-overview/create-applications/native-apps) 또는 [Register Single-Page Web Applications](https://auth0.com/docs/get-started/auth0-overview/create-applications/single-page-web-apps).을 읽으세요.
- 여러분들의 애플리케이션 타입에 따라서 네이티브 또는 싱글 페이지 애플리케이션의 애플리케이션 타입을 선택하세요. 
- 여러분들의 Callback URl을 추가하세요. 여러분들의 애플리케이션 타입과 플랫폼에 따라서 여러분들의 callback URL 형식은 다양할 것입니다. 애플리케이션 타입과 플랫폼에 대한 자세한 정보는  [Native/Mobile Quickstarts](https://auth0.com/docs/quickstart/native)및  [Single-Page App Quickstarts](https://auth0.com/docs/quickstart/spa).을 참고해주세요.
- 여러분들의 애플리케이션의 Authorization Code를 포함한 승인 타입을 포함하고 있는지 확인하세요. 더 배우기 위해서는 [Update Grant Types](https://auth0.com/docs/get-started/applications/update-grant-types).을 참고해주세요.

### Create code verifier
토큰을 요청하기 위해서 Auth0에 최종적으로 전송되는 Base64로 인코딩되고 암호학적으로 랜덤한 code verifier를 생성하세요. 

code_verifier를 생성하기 위한 알고리즘에 대해서 더 배우기 위해서는 OAuth Proof Key for Code Exchange 스펙의  [4.1 Client Creates a Code Verifier](https://datatracker.ietf.org/doc/html/rfc7636#section-4.1)을 읽으세요.


#### Java sample
```java
// Dependency: Apache Commons Codec 
// https://commons.apache.org/proper/commons-codec/ 
// Import the Base64 class. 
// import org.apache.commons.codec.binary.Base64; 
SecureRandom sr = new SecureRandom(); 
byte[] code = new byte[32]; 
sr.nextBytes(code); 
String verifier = Base64.getUrlEncoder().withoutPadding().encodeToString(code);

```

### Create code challenge
authorization code를 요청하기 위해서 Auth0에 전송될 에정인 code verifier로부터 code challenge를 생성하세요. 

code challeng가 어떻게 code verifier에서 도출되는지 더 자세히 알아보기 위해서는 [4.2 Client Creates the Code Challenge](https://datatracker.ietf.org/doc/html/rfc7636#section-4.)을 읽으세요.

#### Java sample
```java
// Dependency: Apache Commons Codec
// https://commons.apache.org/proper/commons-codec/
// Import the Base64 class.
// import org.apache.commons.codec.binary.Base64;
byte[] bytes = verifier.getBytes("US-ASCII");
MessageDigest md = MessageDigest.getInstance("SHA-256");
md.update(bytes, 0, bytes.length);
byte[] digest = md.digest();
String challenge = Base64.encodeBase64URLSafeString(digest);
```

### Authorize user
사용자의 인증을 요청하고 authorization code를 가지고 여러분들의 애플리케이션으로 리다이렉션 시킵니다.

일단 여러분들이 code verifier와 code challenge를 생성하였다면, 여러분들은 사용자의 인증을 얻어야 합니다. 이것은 기술적으로 authorization flow의 시작입니다. 그리고 이 단계는 한개이상의 프로세스들을 포함하고 있습니다. 

사용자 인증, 인증을 다루기 위해 Identity Provider에게 사용자를 리다이렉션 하기,  [Single Sign-on (SSO)](https://auth0.com/docs/authenticate/single-sign-on) 세션 활성화를 위해 체크하기, 승인이 이전에 주어진 적이 있음에도 불구하고 요청된 권한 레벨에 대한 사용자 동의 얻기.

**사용자를 인증하기 위해서 여러분들의 애플리케이션은 사용자에게 여러분들이 이전 단계에서 생성한 code challenge가 포함된 [authorization URL](https://auth0.com/docs/api/authentication#authorization-code-grant-pkce-)을 전송해야 합니다.** 

### Authorization URL example
```text
https://{yourDomain}/authorize?
    response_type=code&
    code_challenge={codeChallenge}&
    code_challenge_method=S256&
    client_id={yourClientId}&
    redirect_uri={yourCallbackUrl}&
    scope={scope}&
    state={state}
```

### Parameters
| Parameter Name          | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `response_type`         | Auth0가 `code` 또는 `token`을 반환할 인증정보 종류를 나타냅니다. 이 플로우에서는 값은 `code`여야 합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `code_challenge`        | `code_verifier`로부터 생성된 challenge 값입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `code_challenge_method` | challenge를 생성하기 사용된 방법 (ex, S256). PKCE 스펙은 두가지 방법인 `S256` 과 `plain`을 정의하고 있습니다. 이 예제에서는 S256을 사용하고 있으며, plain은 권장되지 않으므로 Auth0에서 지원하는 유일한 방법입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `client_id`             | 여러분들의 애플리케이션의 `Client ID`입니다. 여러분들은 여러분들의 [Application Settings](https://manage.auth0.com/#/Applications/{yourClientId}/settings).에서 Client ID 값을 찾을 수 있습니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `redirect_uri`          | 사용자에 의해서 인증이 승인된 다음에 브라우저로 Auth0가 리다이렉션 시킬 URL입니다. Authorization Code는 code 쿼리 파라미터에서 이용가능합니다. 여러분들은 여러분들의 애플리케이션 세팅에서 유효한 콜백 URL을 명시할 수 있습니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `scope`                 | 여러분들이 반환받기를 원하는 claims(또는 사용자 속성들)을 지시하는 권한을 요청할 범위를 지정합니다. 이것들은 공백에 의해서 구분됩니다. 응답에 ID Token을 얻기 위해서는 여러분들은 스코프에 최소 `openid`를 명시해야 합니다. 만약 여러분들이 사용자의 프로필을 반환받고 싶다면, 여러분들은 `openid profile`과 같이 요청하면 됩니다. 여러분들은 `email`과 같은 사용자에 대한  [standard OpenID Connect (OIDC) scopes](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)의 요청이나 namespace 형식을 따르는 커스텀 claims을 요청할 수 있습니다. Refresh Token을 얻기 위해서는 `offline_access`를 포함하세요.    |
| `state`                 | (권장) 여러분들의 애플리케이션 다시 리다이렉션할때 Auth0는 초기 요청에 애플리케이션이 추가하는 불투명한 임의의 영숫자 문자열을 포함합니다. cross-site request forgery(CSRF) 공격을 예방하기 위해서 이 값을 사용하는 방법을 보기 위해서는 [Mitigate CSRF Attacks With State Parameters](https://auth0.com/docs/protocols/oauth2/mitigate-csrf-attacks).을 참고해주세요.|
| `connection`            | (선택적) 사용자가 특정 연결로 로그인하도록 강제합니다. 예를 들어 `github`의 값을 전달하여 Github 계정으로 로그이하도록 사용자를 Github로 직접 전송할 수 있습니다. 명세하지 않을때는 사용자는 모든 설정된 연결들을 가진 Auth0 잠금 화면을 볼것입니다. 여러분들은 여러분들의 여러분들의 애플리케이션의 연결 탭에 설정된 연결들을 볼 수 있습니다.|
| `organization`          | (선택적) 사용자를 인증할때 사용하기 위해서 기관(organization)의 ID입니다. 제공하지 않을때는 만약 여러분들의 애플리케이션이 **Display Organization Prompt**로 설정되어 있다면, 사용자는 인증할때 기관 이름을 입력하기 하여 사용이 가능합니다.|
| `invitation`            | (선택적) origanization 초대의 Ticket ID입니다. Organization으로 멤버를 초대할때 여러분들의 애플리케이션은 사용자가 초대를 수락할때 `invitation`과 `organization` 키-값 쌍을 전송함으로써 초대 수락을 핸들링하여야 합니다.|

예를 들어 여러분들의 애플리케이션이 다음과 같이 로그인을 추가할때 인증 URL에 대한 HTML 코드를 다음과 같이 구현할 수 있다.

```html
<a href="https://{yourDomain}/authorize?
  response_type=code&
  code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM&
  code_challenge_method=S256&
  client_id={yourClientId}&
  redirect_uri={yourCallbackUrl}&
  scope=openid%20profile&
  state=xyzABC123">
  Sign In
</a>
```

### Response
만약 모든것이 잘되면, 여러분들은 302 응답을 받습니다. authorization code는 URL의 쿼리 파라미터에 포함됩니다.
```text
HTTP/1.1 302 Found
Location: {yourCallbackUrl}?code={authorizationCode}&state=xyzABC123
```

### Request tokens
토큰을 위하여 여러분들의 authorization code와 code verifier를 같이 포함하여 전송합니다.

지금 여러분들은 Authorization Code를 가지고 있고 만약 토큰으로 교환해야 한다면, 이전 단계로부터 추출된 Authorization Codecode)를 사용하여 code verifier를 포함하여  [token URL](https://auth0.com/docs/api/authentication#authorization-code-pkce-)에 전송합니다.

### POST to token URL example
```text
curl --request POST \
  --url 'https://{yourDomain}/oauth/token' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data grant_type=authorization_code \
  --data 'client_id={yourClientId}' \
  --data 'code_verifier={yourGeneratedCodeVerifier}' \
  --data 'code={yourAuthorizationCode}' \
  --data 'redirect_uri={https://yourApp/callback}'
```

### Parameters
|Parameter Name|Description|
|---|---|
|`grant_type`|"authorization_code"으로 설정하세요. 고정값입니다.|
|`code_verifier`|이 튜토리얼의 첫번째 단계에서 생성한 암호학적으로 랜덤하게 생성한 키값입니다.|
|`code`|이 튜토리얼의 이전 단계에서 발급한 `authorization_code`입니다.|
|`client_id`|여러분들의 애플리케이션의 Client ID입니다. 이것은 애플리케이션 설정에서 확인할 수 있습니다.|
|`redirect_uri`|여러분들의 애플리케이션에서 설정한 callback URL입니다. 이 튜토리얼의 이전 단계에서 Authorization URL에 전달되는 `redirect_uri`과 정확히 매칭되어야 합니다. 이것은 URL 인코딩되어야 합니다.|

### Response
만약 모든것이 잘되면, 여러분들은 200 응답을 받습니다. 페이로드에는 access token, refresh token, id token, token type 값들이 포함됩니다.
```json
{
  "access_token":"eyJz93a...k4laUWw",
  "refresh_token":"GEbRxBN...edjnXbL",
  "id_token":"eyJ0XAi...4faeEoQ",
  "token_type":"Bearer",
  "expires_in":86400
}
```


## Learn more

- [OAuth 2.0 Authorization Framework](https://auth0.com/docs/authenticate/protocols/oauth)
- [OpenID Connect Protocol](https://auth0.com/docs/authenticate/protocols/openid-connect-protocol)
- [Tokens](https://auth0.com/docs/secure/tokens)

---

## Prevent Attacks and Redirect Users with OAuth 2.0 State Parameters
Authorization 프로토콜들은 여러분들의 애플리케이션의 이전 state를 재저장하기 위해 state 파라미터를 제공합니다. `state`파라미터는 Authorization 요청에서 클라이언트가 설정한 일부 state 객체를 보존하고 응답에서 클라이언트가 이 state 객체를  사용할 수 있도록 합니다.

### CSRF attacks
`state` 파라미터를 사용하는 주된 이유는 시작되려는 각 **인증 요청과 관련된 유니크하고 추측할 수 없는 값을 사용하여 CSRF(Cross Site Request Forgery) 공격을 완화하기 위해서**입니다. 여러분들이 전송한 state 값과 인증하고 온 응답으로 state값이 일치하는지 확인해서 여러분들에게 CSRF 공격을 예방합니다.

`state` 파라미터는 문자열이므로 다른 정보를 인코딩할 수 있습니다. state 값을 인증 요청할때 전송하고 응답을 처리할 때 수신된 state 값의 유효성을 확인합니다. 유효성 검사를 수행하기 위해서 무언가를 클라이언트 애플리케이션 사이드 측(쿠키, 세션 또는 로컬 저장소)에 저장합니다. 만약 여러분들이 state가 불일치한 응답을 받은 경우 요청하지 않은 요청에 대한 응답이거나 응답을 위조하려는 사람이기 때문에 공격의 대상이 될 수 있음을 추론할 수 있습니다.

특히 CSRF 공격은 위조된 요청에 대한 응답을 볼 방법이 없기 때문에 사용자 데이터를 가져오는 대신 작업을 시작시키기 위한 상태 변경 요청을 대상으로 합니다. 가장 기본적인 경우의 state 파라미터는 요청과 인증에서 받은 응답을 연관시키는데 사용되는 nonce(재전송 공격을 탐지하고 방지하기 위해 인증 프로토콜에서 한번 발급되는 임의의 번호)가 되어야 합니다.

대부분의 현대 OIDC와 싱글 페이지 애플리케이션에서 Auth0.js를 포함하는 OAuth SDK들은 자동으로 state를 생성하고 검증합니다.

### Set and compare state parameter values
1. Identity Provider(IdP)로 요청을 리다이렉션하기 전에 애플리케이션은 랜덤한 문자열을 다음과 같이 생성합니다.
```text
xyzABC123
```
 - state는 길이 제한이 없습니다. 만약 여러분들이 `414 Request-URI Too Large`에러를 받는다면, state 길이를 작게 조절하세요.

2. 문자열을 로컬 저장소 같은 곳에 저장하세요.
```text
storeStateLocally(xyzABC123)
```

3. request URI에 state 파라미터를 추가하세요. (URL 인코딩은 필수적입니다.) 예를 들어 다음과 같습니다.
```text
// Encode the String   
tenant.auth0.com/authorize?...&state=xyzABC123
```
- 요청이 전송된 후, 사용자는 Auth0에 의해서 애플리케이션으로 다시 리다이렉션됩니다. 이 리다이렉션에는 state 값이 포함될 것입니다.
- 사용되는 연결 종류에 따라서 state 값이 request body나 쿼리 파라미터에 응답될 수 있습니다.

```text
/callback?...&state=xyzABC123
```

4. 반환된 state 값을 발급받고 여러분들이 앞서 저장한 state 값과 비교합니다. 만약 state 값이 매치된다면, 인증 응답을 승인하세요. 그렇지 않으면 거부하세요.
```javascript
// Decode the String
var decodedString = Base64.decode(encodedString);
if(receivedState === retrieveStateStoredLocally()) {
 // Authorized request
} 
else {
  // This response is not for us, reject it
}
```

### Redirect users
`state` 파라미터를 사용하여 사용자가 인증 프로세스를 시작하기 전에 있었던 위치에 놓을 애플리케이션의 state를 인코딩할 수 있습니다. 예를 들어 사용자들이 애플리케이션에 보호된 페이지에 접근하고자 한다면, 인증을 위한 요청 트리거가 발생하고 여러분들은 인증이 완료된 후 사용자가 원하는 페이지로 리다이렉션 시킬 수 있습니다.

리다이렉션 URL과 같은 state 생성하고 로컬 저장소(쿠키, 세션, 로컬 스토리지)에 저장합니다. 프로토콜 메시지에서 nonce를 state로서 사용하세요. 만약 응답된 state가 기존 저장하고 있던 nonce를 매치시켜서 검증합니다. 

### References
- [Prevent Attacks and Redirect Users with OAuth 2.0 State Parameters](https://auth0.com/docs/secure/attack-protection/state-parameters)

