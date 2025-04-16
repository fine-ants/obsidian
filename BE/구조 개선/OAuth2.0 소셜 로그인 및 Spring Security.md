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



