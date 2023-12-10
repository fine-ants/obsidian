
FineAnts 애플리케이션에서 소셜 로그인을 사용하기 위해서 Google, Kakao, Naver를 제공하고 있습니다. 그중에서 Google, Kakao는 프로세스 수행과정중 PKCE(Proof Key for Code Exchange)와 OIDC(Open ID Connect)를 제공하고 있습니다.

### PKCE(Proof Key for Code Exchange)
PKCE는 표준적인 Authorization Code Flow 과정에서 client-secret을 전달하지 않고 code verifier라는 랜덤 생성한 키 값을 전달하여 액세스 토큰을 발급받는 방법입니다. PKCE를 사용하는 이유는 client-secret을 저장할 수 없는 Native 또는 싱글 페이지 애플리케이션과 같은 애플리케이션을 위하여 사용되는 방법입니다.

### FineAnts 애플리케이션에서 PKCE가 강화한 Authorization Code Flow는 어떻게 작동하고 있는가?

![[AuthorizationCodeFlowWithPCKE.drawio.png]]
- 해당 프로세스는 PKCE가 강화한 표준 Authorization Code Flow를 따르고 있고 스프링 애플리케이션에서 ID Token과 Access Token을 발급받은 상태에서 Access Token을 사용하여 Resource Server로부터 사용자 정보를 가져오지 않습니다. 대신 ID Token을 검증 및 디코딩하여 추출한 사용자 정보를 가지고 로그인 프로세스를 수행합니다.
- 만약 사용자가 OAuth 로그인 프로세스를 처음 수행하는 것이라면 스프링 서버쪽에 회원 정보를 생성하여 저장합니다.
- 회원가입 처리가 완료되면 회원 정보를 기반으로 스프링 서버 내에서 자체 JWT(Access Token, Refresh Token)을 생성하여 React 애플리케이션에 응답합니다.
- Kakao 같은 경우 ID Token 및 Access Token 발급 요청시 client-secret을 선택적으로 보낼 수 있지만 Google 같은 경우는 PKCE 적용함에도 불구하고 필수적으로 client-secret을 같이 보내야 한다.

### Authorization Code Flow With PKCE 장점
- Native 또는 Single Page Application과 같은 애플리케이션 타입에 적용하여 client-secret을 저장 및 노출시키지 않고 code verifier와 code challenge를 이용하여 액세스 토큰을 발급받을 수 있다. 즉, **client-secret을 저장하지 않을 수 있다.**
	- 단, Google에서는 PKCE 플로우임에도 액세스 토큰 요청시 client-secret 요구를 강제하고 있다. 구글은 client-secreet과 Code Verifier를 같이 전송하여 추가적인 보안을 하고 있는 상황이다.
- PKCE를 사용하게 되면 악의적인 공격자가 Authorization Code를 탈취하여 요청할 수는 있지만 Code Verifier를 요구하게 되면서 액세스 토큰 발급을 막을 수 있습니다. 클라이언트에서는 Code Verifier를 S256 방식으로 암호화한 Code Challenge를 인가코드 요청할때 전송하므로 Code Verifier가 노출되는 길이 없다. **즉, PKCE 사용시 악의적인 공격자가 Authorization Code를 탈취해도 Access Token을 발급받을 수 없도록 할 수 있다.**


### 왜 인가 코드 URL 생성시 state, nonce 프로퍼티를 생성하는가?
- 카카오 및 구글에서는 인가 코드를 요청할때 쿼리 파라터로 state를 추가하여 전송할 수 있습니다.
	- state를 사용하여 **Cross-Site-Request-Forgery(CSRF) 공격으로부터 로그인 요청을 보호하기 위해 사용**됩니다. state를 임의의 문자열로 생성하거나 쿠키의 해시 또는 클라이언트의 상태를 캡처하는 다른 값을 인코딩하는 경우 응답을 검증하여 요청과 응답이 동일한 브라우저에서 시작되었는지 추가로 확인하여 CSRF 공격으로부터 보호할 수 있습니다.
	- 각 사용자의 로그인 요청에 대한 state 값은 고유해야 합니다.
	- 인가 코드 요청, 인가 코드 응답, 토큰 발급 요청의 state 값 일치 여부로 요청 및 응답 유효성을 확인합니다
- Cross-Site-Request-Forgery(CSRF)란 무엇인가?
	- 사용자가 자신의 의지와 무관하게 공격자가 의도한 행동을 해서 특정 웹 페이지를 보안에 취약하게 한다거나 수정, 삭제 등의 작업을 하게 만드는 방법입니다.
	- OAuth2.0에서 state 파라미터를 사용하지 않고 CSRF 공격을 받을 수 있는 예시는 링크를 통해서 참고해주세요.
- ID 토큰 재생 공격이란?
		- 인증된 사용자의 ID 토큰을 가로채어 그 토큰을 사용하여 다른 사용자로 위장하여 악의적인 행동을 시도하는 공격입니다.

### 왜 OpenID Connect를 사용하는가?

### 왜 스프링 서버 내에서 AuthorizationRequest 객체들을 관리하는가?


