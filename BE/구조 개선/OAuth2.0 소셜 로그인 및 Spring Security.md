Spring Security 라이브러리를 추가하면서 소셜 로그인 기능을 다시 구현한 이유는 무엇인가요?

- [[#개요|개요]]
- [[#직접 구현한 소셜 로그인 방식|직접 구현한 소셜 로그인 방식]]
	- [[#직접 구현한 소셜 로그인 방식#직접 구현한 소셜 로그인의 수행 과정|직접 구현한 소셜 로그인의 수행 과정]]
	- [[#직접 구현한 소셜 로그인 방식#소셜 로그인을 직접 구현한 이유|소셜 로그인을 직접 구현한 이유]]
- [[#직접 구현 방식의 한계|직접 구현 방식의 한계]]
- [[#Spring Security 도입 배경|Spring Security 도입 배경]]
- [[#유지보수성과 확장성 측면에서의 개선 효과|유지보수성과 확장성 측면에서의 개선 효과]]
- [[#마무리 및 회고|마무리 및 회고]]


## 개요
이 글에서는 기존 시스템에서 직접 구현한 OAuth 2.0 기반 소셜 로그인 기능을, Spring Security 라이브러리를 도입하면서 해당 프레임워크에 맞게 재구현하게 된 배경과 이유를 소개합니다. 직접 구현한 방식의 한계와 유지보수성 문제를 어떻게 Spring Security가 해결해주었는지도 함께 다룹니다.

## 직접 구현한 소셜 로그인 방식
### 직접 구현한 소셜 로그인의 수행 과정
기존 직접 구현한 소셜 로그인 방식의 수행 과정은 다음과 같습니다.
1. 사용자는 소셜 플랫폼 인증을 위한 URL 생성을 서버에게 요청합니다.
2. 서버는 소셜 플랫폼에 맞는 URL을 생성하여 응답합니다.
	- 생성한 URL에는 인증 후 발급받은 인가 코드를 전달할 리다이렉션 주소가 쿼리 파라미터에 포함되어 있습니다.
3. 사용자는 URL에 접속하여 소셜 플랫폼에 인증 과정을 수행한 뒤에 인가 코드를 받습니다. 그 다음에 클라이언트는 인가 코드를 리다이렉션 주소에 포함하여 해당 주소로 전달합니다.
4. 서버는 전달받은 인가 코드를 이용하여 다음과 같은 로그인 처리를 수행합니다.
	1. 인가 코드를 이용하여 인증 서버로부터 액세스 토큰을 발급받습니다.
	2. 발급받은 액세스 토큰을 이용하여 소셜 플랫폼의 사용자에 대한 프로필 정보를 조회합니다.
	3. 만약 기존 회원 정보가 없다면 회원 정보를 생성한 다음에 데이터베이스에 저장합니다. 회원 정보가 있다면 조회한 프로필 정보로 갱신됩니다.
	4. 회원 정보를 기반으로 액세스 토큰 및 리프레시 토큰을 생성합니다.
		- 이 과정에서 생성한 액세스 토큰 및 리프레시 토큰은 소셜 플랫폼에서 발급한 액세스 토큰과 다르며 서버가 인증 정보 상태를 유지하기 위한 전략으로 JWT를 선택하였기 때문에 서버용으로 생성한 토큰들입니다.
	5. 기본적인 회원 정보 및 JWT 정보를 ResponseBody에 포함하여 응답합니다.

위와 같은 수행 과정과 같이 기존에 직접 구현한 소셜 로그인 방식은 **컨트롤러 레이어에서 요청을 받은 다음에 서비스 메서드에서 액세스 토큰 발급, 프로필 정보 조회, 회원 정보 저장, JWT 정보 생성과 같은 과정을 한 login 메서드에서 전부 처리**하고 있습니다. 로그인 처리를 하는 코드는 다음과 같습니다.
```java
@Service
public class MemberService {
	public OauthMemberLoginResponse login(String provider, String code, String redirectUrl, String state,
		LocalDateTime now) {
		log.info("로그인 서비스 요청 : provider = {}, code = {}, redirectUrl = {}, state = {}", provider, code, redirectUrl,
			state);
		AuthorizationRequest request = getCodeVerifier(state);
		OauthUserProfileResponse profileResponse = getOauthUserProfileResponse(provider, code, redirectUrl, request,
			now, state);
		Optional<Member> optionalMember = getLoginMember(provider, profileResponse);
		Member member = optionalMember.orElseGet(() ->
			Member.builder()
				.email(profileResponse.getEmail())
				.nickname(generateRandomNickname())
				.provider(provider)
				.profileUrl(profileResponse.getProfileImage())
				.build());
		Member saveMember = memberRepository.save(member);
		Jwt jwt = jwtProvider.createJwtBasedOnMember(saveMember, now);
		redisService.saveRefreshToken(saveMember.createRedisKey(), jwt);
		return OauthMemberLoginResponse.of(jwt, saveMember);
	}
}
```
- getOauthUserProfileResponse 메서드에서 WebClient를 이용하여 액세스 토큰을 발급받고 프로필 정보를 조회합니다.

### 소셜 로그인을 직접 구현한 이유
구현 당시 Spring Security 프레임워크를 이용하지 않고 직접 구현한 이유는 다음과 같았습니다.
- Spring Security 프레임워크의 숙련도가 미숙하였습니다.
- 구현 당시에는 일반 사용자에 대한 인증 처리면 충분하다고 판단하였습니다.
- API 경로별 상세한 접근 권한이 필요치 않았습니다.

## 직접 구현 방식의 한계
- 유지 보수의 어려움(예: 중복 코드, 보안 로직 누락 위험 등)
- 확장성과 일관성의 부족
- 테스트와 디버깅의 어려움

위와 같이 구현한 직접 소셜 로그인을 구현하였을 때 다음과 같은 한계점을 가지고 있습니다.

**커스텀 인증 플로우의 모든 책임이 서비스 메서드 하나에 집중되어 있습니다.** 
```java
// MemberService.login() 메서드
AuthorizationRequest request = getCodeVerifier(state);
OauthUserProfileResponse profileResponse = getOauthUserProfileResponse(...);
Optional<Member> optionalMember = getLoginMember(...);
...
Jwt jwt = jwtProvider.createJwtBasedOnMember(...);
redisService.saveRefreshToken(...);
```
- login 메서드는 너무 많은 일을 하고 있습니다. 해당 메서드는 state 검증, 액세스 토큰 요청, 사용자 정보 조회, 회원 DB 조회/생성, JWT 발급, Redis 저장 등의 작업을 수행하고 있습니다.
- 현재 구현한 메서드는 SRP(Single Responsibility Principal, 단일 책임 원칙)을 위반하고 있으며 변경이나 테스트가 어려워집니다.
- 예를 들어 토큰 발급 방식이 변경되거나 프로필 정보 구조가 변경되면, 이 메서드 하나를 동시에 수정해야 합니다.

**상태 검증(state, code_verifier, nonce)를 직접 관리합니다**
보안 핵심 요소인 state와 nonce, code_verifier를 직접 구현 및 상태 관리하고 있습니다. 실수 또는 동시성 이슈로 인해 보안 취약점이 있을 수 있습니다. Spring Security에서는 state, code_verifier, nonce를 자동으로 안전하게 처리합니다. 직접 구현시 누락/오류의 가능성이 크고 디버깅이 어렵습니다.

**provider별 분기 처리가 서비스 내부에 강하게 엮어 있습니다.**
OAuth/OIDC 구분, provider별 요청/응답 처리 로직이 코드 흐름 안에 직접적으로 노출되어 있습니다. 그리소 새로운 소셜 로그인 추가시 MemberService 코드까지 수정해야 합니다. 그래서 유지보수성이 낮고 확장성이 떨어지는 구조입니다.

**에러 처리가 분산되고 일관되지 않습니다.**
OAuth 과정 중 발생할 수 있는 다양한 실패(토큰 발급 실패, 사용자 정보 조회 실패 등)에 대한 처리 로직이 부족하거나 일관되지 않습니다. 운영 문제가 발생해도 어디서 실패했는지 로그/에러 파악이 어렵습니다. 예외 발생시에도 사용자에게 일관된 응답 포맷으로 처리되지 않을 수 있습니다.

**Spring Security의 필터 체인과 분리된 인증 처리**
인증 로직이 Spring Security와 완전히 분리되어 있어, Spring Security의 인증 흐름(필터 체인, SecurityContext 등)을 활용하지 못합니다. 로그인 이후 사용자 인증 상태를 SecurityContextHolder에 수동으로 올려야 하며, 표준적인 방식이 아닙니다. 따라서 Spring Security 기반 권한 체크(@PreAuthorize, hasRole) 등 기능도 자연스럽게 연동되지 않습니다.

위 설명을 표로 정리하면 다음과 같습니다.

| 문제                     | 설명                                                 |
| ---------------------- | -------------------------------------------------- |
| 책임 집중                  | 인증, DB 처리, 토큰 발급 등 많은 책임이 한 메서드에 몰려 있음             |
| 보안 요소 수동 처리            | state, code_verifier, nonce 등 중요한 보안 요소를 직접 다루고 있음 |
| provider 추가 어려움        | 새로운 소셜 로그인 추가시 서비스 로직 수정 필요                        |
| 에러 처리 부족               | 예외 발생 시 사용자 응답 일관성이 없고 디버깅 어려움                     |
| Spring Security와 통합 부족 | 필터 기반 인증 처리 흐름과 통합되지 않아서 인증 상태 관리가 불안정함            |


## Spring Security 도입 배경
- 어떤 구조로 변경했는지
- 커스터마이징한 부분이 있다면 간략히 소개
- 도입 과정에서 겪은 이슈나 고민



## 유지보수성과 확장성 측면에서의 개선 효과
- 코드 구조 정리
- 보안 정책 일관성
- 기능 확장이나 정책 변경 시 유연함

## 마무리 및 회고
- 이번 리팩토링의 의의
- 얻은 교훈 또는 다음에 적용할 방향


References
- source code : https://github.com/fine-ants/FineAnts-was/pull/41
