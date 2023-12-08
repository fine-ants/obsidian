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



