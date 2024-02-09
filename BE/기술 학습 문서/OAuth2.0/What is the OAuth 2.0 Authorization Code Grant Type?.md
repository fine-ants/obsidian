Authorization Code Grant Type은 OAuth2.0 Grant Type들 중에서 가장 일반적인 유형입니다. 해당 타입은 웹 애플리케이션과 네이티브 앱 모두에서 사용자가 애플리케이션을 승인(Grant)한 후 액세스 토큰을 얻는데 사용됩니다.

## What is an OAuth 2.0 Grant Type?
OAuth2.0에서 "Grant Type"이라는 용어는 애플리케이션이 액세스 토큰을 얻는 방법을 의미합니다. OAuth2.0에서는 Authorization Code Flow를 포함한 여러개의 Grant Type을 정의하고 있습니다.  OAuth 2.0 Extension들 또한 새로운 Grant Type으로 정의할 수도 있습니다.

각각의 Grant Type은 특정한 사용 사례에 최적화되어 있습니다.
- 웹 애플리케이션
- 네이티브 애플리케이션
- 웹 브라우저 실행 기능이 없는 디바이스
- 서버 간 애플리케이션들

## The Authorization Code Flow
Authorization Code Grant Type은 웹과 모바일 애플리케이션에서 사용됩니다. 먼저 애플리케이션이 Flow를 시작하기 위해서 브라우저를 실행해야 한다는 점에서 대부분의 다른 Grant Type들과 다릅니다. 상위 레벨에서 Flow는 다음 단계를 수행합니다.

- 애플리케이션은 브라우저를 열어서 사용자를 OAuth 서버로 보냅니다.
- 사용자는 Authorization 프롬프트를 보고 애플리케이션의 권한 위임 요청을 승인합니다.
- 사용자는 쿼리 스트링에 authorization code를 가지고 애플리케이션으로 다시 리다이렉션합니다.
- 애플리케이션은 액세스 토큰을 얻기 위해서 authorization code를 가지고 액세스 토큰과 교환합니다.

## Get the User’s Permission
OAuth는 사용자가 애플리케이션에게 특정 리소스에 접근할 수 있는 권한을 부여할 수 있도록 하는 프로토콜입니다. 애플리케이션은 우선 리소스에 접근할 수 있는 권한을 결정한 다음에 사용자를 브라우저를 통하여 권한 선택 창으로 이동시키고 사용자로부터 권한을 받아야 합니다.

인가 플로우를 진행하기 위해서는 애플리케이션은 다음과 같은 URL을 구성하고 URL을 브라우저로 엽니다.
```
https://authorization-server.com/auth
 ?response_type=code
 &client_id=29352915982374239857
 &redirect_uri=https%3A%2F%2Fexample-app.com%2Fcallback
 &scope=create+delete
 &state=xcoiv98y2kd22vusuye3kch
```

위 URL의 쿼리 파라미터의 설명은 다음과 같습니다.
- response_type=code : 이것은 authorization server(ex, 구글 인가 서버)에게 애플리케이션이 authorization code flow를 시작하고 있음을 알려줍니다.
- client_id : client_id는 애플리케이션의 공개 식별자입니다. client_id는 개발자가 애플리케이션을 등록할때 얻습니다.
- redirect_uri : authorization server에게 사용자가 권한 위임 요청을 승인한 다음에 사용자가 다시 돌아갈때 리다이렉션 주소입니다.
- scope : 한개이상의 공백으로 구성된 애플리케이션이 요청하는 권한들을 의미합니다.
- state : 애플리케이션이 생성한 랜덤한 문자열이고 생성한 문자열은 요청의 쿼리 파라미터에 포함됩니다. state는 그런 다음에 사용자가 애플리케이션을 인가한 다음에 반환되는 값과 같은 값인지 검사합니다. state는 CSRF 공격을 예방하는데 사용됩니다.

사용자가 위 URL에 방문한 다음에 authorization server는 이 애플리케이션의 요청을 인가하는 것과 같은 프롬프트 요청을  보여줄 것입니다.
![[Pasted image 20240209140535.png]]

## Redirect Back to the Application
만약 사용자가 요청을 승인하면 authorization server는 애플리케이션에 의해서 명세된 redirect_uri 주소로 리다이렉트합니다. 이때 redirect_uri에는 쿼리 파라미터에 code와 state가 같이 포함되어 전송됩니다.

예를 들어 사용자가 다음 URL과 같이 리다이렉트 된다고 가정합니다.
```
https://example-app.com/redirect
 ?code=g0ZGZmNjVmOWIjNTk2NTk4ZTYyZGI3
 &state=xcoiv98y2kd22vusuye3kch
```
`state` 값은 애플리케이션이 초기에 설정한 값(리퀘스트 쿼리 파라미터에 같이 첨부하여 전송한)과 같은 값일 것입니다. 애플리케이션은 초기에 요청에서 전달한 state값과 리다이렉트로 전달된 state값이 같은 값인지 검사할 것입니다. 이 검사는 CSRF 공격을 막을 것입니다.

`code` 값은 authorization server에 의해서 생성된 **인가 코드(authorization code)** 입니다. 이 인가 코드는 상대적으로 짧게 살아있고 OAuth 서비스에 따라서 1~10분정도입니다.

## Exchange the Authorization Code for an Access Token
애플리케이션은 authorization code를 가지고 있고 이제 authorization code를 이용하여 액세스 토큰(Access Token)으로 교환할 할 수 있습니다.

애플리케이션은 다음 파라미터를 포함하는 서비스의 액세스 토큰 엔드포인트로 전송할 POST 요청을 만듭니다.
- grant_type=authorization_code : token 엔드포인트에게 애플리케이션이 Authorization Code Grant Type을 사용하고 있음을 알려줍니다.
- code : 애플리케이션이 리다이렉트시 받은 authorization code를 의미합니다.
- redirect_uri : authorization code를 요청할 때 사용되었던 같은 redirect URI를 의미합니다. 몇몇 API들은 이 파라미터를 요구하지 않지만, 여러분들은 여러분들이 접근하는 특정 API의 문서를 다시 확인해서 redirect_uri 파라미터가 필수적인지 선택적인지, 아니면 없어도 되는지 확인해야합니다.
- client_id : 애플리케이션의 client ID.
- client_secret : 애플리케이션의 client secret입니다. 이 정보는 액세스 토큰을 얻고자 하는 요청이 애플리케이션으로부터 만들어진 것인지 보장하도록 합니다. 그리고 이 정보는 authorization code를 가로챈 해커가 아님을 보장합니다.

token 엔드포인트는 code(authorization code)가 만료되지 않았는지, client ID와 client secret이 매치되는지 요청의 모든 파라미터를 검증할 것입니다.  만약 모든것이 일치하면, authorization server는 액세스 토큰을 생성하고 response에 담아서 반환합니다.

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"MTQ0NjJkZmQ5OTM2NDE1ZTZjNGZmZjI3",
  "token_type":"bearer",
  "expires_in":3600,
  "refresh_token":"IwOGYzYTlmM2YxOTQ5MGE3YmNmMDFkNTVk",
  "scope":"create delete"
}
```

위와 같은 http response를 받는하면 Authorization Code Flow는 완료입니다.

## When to use the Authorization Code Flow
Authorization Code Flow는 웹과 모바일 애플리케이션에서 가장 잘 사용됩니다. Authorization Code Grant는 액세스 토큰을 위해 authorization code를 교환하는 추가적인 단계를 가지기 때문에Implicit Grant Type에서 나오지 않는 추가적인 보안 레이어(security layer)가 제공됩니다.

만약 여러분들이 모바일 애플리케이션이나 client secret를 저장할 수 없는 다른 애플리케이션 종류에서 Authorization Code Flow를 사용한다면, 여러분들은 [PKCE extension](https://www.oauth.com/oauth2-servers/pkce/)을 사용할 수 있습니다. PKCE extension은 authorization code가 가로챌 때와 같은 다른 공격을 방어하기 위해서 제공됩니다.

authorization code를 액세스 토큰으로 교환하는 단계에서 해커는 액세스 토큰을 가로챌 수없도록 보장합니다. 액세스 토큰은 항상 애플리케이션과 OAuth server 간에 보안 백채널(secure backchannel)을 통해 전송되기 때문입니다.

## References
https://developer.okta.com/blog/2018/04/10/oauth-authorization-code-grant-type


