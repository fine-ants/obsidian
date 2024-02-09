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
- client_id : 



