OpenID Connect는 OAuth2.0 규격 프레임워크(IETF RFC 6749 및 6750)를 기반으로 하는 상호 운용 가능한 인증 프로토콜입니다. 인증 서버에서 수행한 인증을 기반으로 사용자의 신원을 확인하고 상호 운용 가능하고 REST와 유사한 방식으로 사용자의 프로필 정보를 얻을 수 있는 방법을 간소화한 것입니다.

OpenID Connect를 사용하면 애플리케이션과 웹 개발자가 로그인 플로우를 시작하고 웹 기반 및 자바스크립트 클라이언트에서 사용자를 확인할 수 있습니다. 또한 OpenID Connect 스펙은 식별 데이터 암호화, OpenID Provider 검색 및 세션 로그아웃과 같은 다양한 옵션 기능을 지원하도록 확장할 수 있습니다.

개발자들을 대상으로는 "현재 브라우저나 모바일 앱에 연결된 사람들이 누구인가"에 대한 질문에 안전하고 검증이 가능한 답변을 제공합니다. 무엇보다 자격 증명 기반 데이터 침해와 관련된 비밀번호 설정, 저장 및 관리 책임을 제거합니다.

![[Pasted image 20240207110829.png]]


## OpenID 연결은 어떻게 작동하는가?
OpenID Connect는 간편한 통합 및 지원, 보안 및 개인 정보 보호 구성, 상호 운용성, 클라이언트 및 장치의 광범위한 지원, 모든 개체가 OpenID Provider가 될 수 있도록 함으로써 인터넷 신원 생태계를 가능하게 합니다.

OpenID Connect는 추상적으로 다음과 같이 작동합니다.
1. 사용자가 웹사이트나 웹 애플리케이션을 브라우저로 접근합니다.
2. 사용자는 로그인 버튼을 누르고 유저이름과 비밀번호를 입력합니다.
3. RP(Client)는 OpenID Provider로 요청을 전송합니다.
4. OpenID Provider는 사용자를 인증하고 인가를 얻습니다.
5. OpenID Provider는 식별 토큰(Identity Token)과 AccessToken을 응답합니다.
6. RP는 사용자 기기에 액세스 토큰과 같이 요청할 수 있습니다.
7. UserInfo 엔드포인트는 최종 사용자에 대한 Claims을 응답합니다.
![[Pasted image 20240207111514.png]]

### Authentication
OIDC를 통해 클라이언트는 사용자를 식별하고 사용자의 정보를 전달하는 과정

### Client
사용자의 정보에 접근하기를 원하는 사람. 이 사람은 소프트웨어의 일부분일 수 있습니다.종종 RP(Relying Party)라고 부를 수 있습니다. 클라이언트는 OP(OpenID Provider)에 등록되어야 합니다. 클라이언트는 웹 애플리케이션이나 네이티브 모바일, 데스크톱 애플리케이션 등이 될 수 있습니다.

### Relying Party
RP는 Identity Provider에 사용자 인증 기능을 위임하는 애플리케이션이나 웹사이트인 Relying Party의 약자입니다.

### OpenID Provider (OP) or Identity Provider (IDP)
OpenID Provider(OP)는 OpenID Connect와 OAuth2.0 프로토콜을 구현한 엔티티입니다. OP는 보안 토큰 서비스, IDP(Identity Provider) 또는 인증 서버와 같은 역할로 언급될 수 있습니다.

###  Identity Token
Identity Token은 인증 프로세스의 결과물입니다. Identity Token은 사용자에 대한 최소한의 식별 정보를 가지고 있습니다. 

### User
리소스에 접근하기 위해 등록된 클라이언트를 사용하는 사람입니다.





