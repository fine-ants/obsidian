FineAnts 애플리케이션에서 소셜 로그인을 사용하기 위해서 Google, Kakao, Naver를 제공하고 있습니다. 그중에서 Google, Kakao는 프로세스 수행과정중 PKCE(Proof Key for Code Exchange)와 OICD(Open ID Connect)를 제공하고 있습니다.

### PKCE(Proof Key for Code Exchange)
PKCE는 표준적인 Authorization Code Flow 과정에서 client-secret을 전달하지 않고 code verifier라는 랜덤 생성한 키 값을 전달하여 액세스 토큰을 발급받는 방법입니다. PKCE를 사용하는 이유는 client-secret을 저장할 수 없는 Native 또는 싱글 페이지 애플리케이션과 같은 애플리케이션을 위하여 사용되는 방법입니다.

### FineAnts 애플리케이션에서 PKCE가 강화한 Authorization Code Flow는 어떻게 작동하고 있는가?


