Spring Security 라이브러리를 추가하면서 소셜 로그인 기능을 다시 구현한 이유는 무엇인가요?


- [[#개요|개요]]
- [[#직접 구현한 소셜 로그인 방식|직접 구현한 소셜 로그인 방식]]
	- [[#직접 구현한 소셜 로그인 방식#소셜 로그인의 수행 과정|소셜 로그인의 수행 과정]]
	- [[#직접 구현한 소셜 로그인 방식#소셜 로그인을 직접 구현한 이유는 무엇인가?|소셜 로그인을 직접 구현한 이유는 무엇인가?]]
	- [[#직접 구현한 소셜 로그인 방식#직접 구현했을 때의 문제점은 무엇이었는가?|직접 구현했을 때의 문제점은 무엇이었는가?]]
- [[#Spring Security 도입|Spring Security 도입]]
	- [[#Spring Security 도입#Spring Security 프레임워크를 도입하게 된 이유|Spring Security 프레임워크를 도입하게 된 이유]]
	- [[#Spring Security 도입#기존 인증 시스템을 Spring Security 프레임워크에 맞게 재구현한 이유|기존 인증 시스템을 Spring Security 프레임워크에 맞게 재구현한 이유]]
	- [[#Spring Security 도입#Spring Security 프레임워크를 도입했을 때 효과는 무엇인가?|Spring Security 프레임워크를 도입했을 때 효과는 무엇인가?]]
	- [[#Spring Security 도입#인증 시스템 구조는 어떻게 변경되었는가?|인증 시스템 구조는 어떻게 변경되었는가?]]
- [[#마무리 및 회고|마무리 및 회고]]


## 개요
이 글에서는 기존 시스템에서 직접 구현한 OAuth 2.0 기반 소셜 로그인 기능을, Spring Security 라이브러리를 도입하면서 해당 프레임워크에 맞게 재구현하게 된 배경과 이유를 소개합니다. 직접 구현한 방식의 한계와 유지보수성 문제를 어떻게 Spring Security가 해결해주었는지도 함께 다룹니다.

## 직접 구현한 소셜 로그인 방식
### 소셜 로그인의 수행 과정
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
- getOauthUserProfileResponse 메서드에서 인가 코드를 이용하여 액세스 토큰을 발급받고 액세스 토큰을 이용하여 프로필 정보를 조회합니다.

### 소셜 로그인을 직접 구현한 이유는 무엇인가?
구현 당시 Spring Security 프레임워크를 이용하지 않고 직접 구현한 이유는 다음과 같았습니다.
- Spring Security 프레임워크의 숙련도가 미숙하였습니다.
- 구현 당시에는 일반 사용자에 대한 인증 처리면 충분하다고 판단하였습니다.
- API 경로별 상세한 접근 권한이 필요치 않았습니다.

### 직접 구현했을 때의 문제점은 무엇이었는가?
현제 제가 구현한 소셜 로그인 서비스의 문제점은 다음과 같았습니다.
- 하나의 login() 메서드에 state 검증, 액세스 토큰 발급, 사용자 프로필 조회, 회원 생성, JWT 생성 작업 등을 수행하고 있습니다. 이는 여러가지 책임이 하나의 메서드에 몰려 있습니다.
- 소셜 로그인 수행시 보안적인 요소를 수동적으로 처리하고 있습니다. state, code_verifier, nonce 등의 중요한 보안 요소를 직접 다루고 있습니다.
- 새로운 소셜 로그인 플랫폼이 추가되는 경우 수정이 필요합니다. 예를 들어 OauthClient를 관리하는 OauthClientRepository에 새로운 소셜 플랫폼을 추가하는 코드를 추가하여야 합니다. 그리고 소셜 플랫폼 정보를 가지고 있는 OauthClient 구현체 클래스를 확장해야 합니다.

위 설명을 표로 정리하면 다음과 같습니다.

| 문제              | 설명                                                 |
| --------------- | -------------------------------------------------- |
| 책임 집중           | 인증, DB 처리, 토큰 발급 등 많은 책임이 한 메서드에 몰려 있음             |
| 보안 요소 수동 처리     | state, code_verifier, nonce 등 중요한 보안 요소를 직접 다루고 있음 |
| provider 추가 어려움 | 새로운 소셜 로그인 추가시 서비스 로직 수정 필요                        |

## Spring Security 도입
### Spring Security 프레임워크를 도입하게 된 이유
Spring Security 프레임워크를 도입하게 된 이유는 **인가 처리 필요성** 때문이었습니다. 기존에는 인증된 사용자만 사용할 수 있는 API로도 충분했지만, 시간이 지나면서 서버 관리 목적의 관리자 전용의 API가 필요하게 되었습니다. 이에 따라 API별로 권한을 세분화 할 수 있는 인가 시스템이 필요하게 되었고, Spring Security를 도입하게 되었습니다.

### 기존 인증 시스템을 Spring Security 프레임워크에 맞게 재구현한 이유
Spring Security 프레임워크를 도입하면서, 기존 인증 시스템을 유지한 상태로 프레임워크의 인가 처리 기능만 활용하려면 인증 후 별도의 추가 작업이 필요했습니다.
Spring Security는 인증이 완료되면 인증 정보를 `Authentication` 객체에 담아 `SecurityContextHolder`에 저장하며, 이렇게 저장된 정보는 `hasRole()`이나 `@Secured` 애노테이션을 통한 인가 처리에 사용됩니다.
하지만 기존 인증 시스템을 그대로 유지하는 경우, 인증이 성공한 후에도 직접 `Authentication` 객체를 생성하고 이를 `SecurityContextHolder`에 수동으로 저장해주는 구현을 매번 추가로 해야 했습니다.
따라서 이러한 불필요한 복잡성과 중복 구현을 줄이기 위해, 기존 인증 시스템을 Spring Security의 구조에 맞게 재구현하게 되었습니다.

### Spring Security 프레임워크를 도입했을 때 효과는 무엇인가?
Spring Security 프레임워크를 도입했을 때 효과는 다음과 같았습니다.
- `hasRole()`, `@Secured` 애노테이션을 추가하여 API의 경로별 상세한 접근 권한을 설정할 수 있습니다.
- OAuth 2.0, OIDC 기반 소셜 로그인을 표준화된 방식으로 처리할 수 있고, 소셜 로그인 플랫폼이 추가되어도 기존 코드를 수정하지 않고 확장이 쉽습니다.
- 기존 인증 시스템을 Spring Security에 맞게 재구현함으로써 인증한 사용자 정보를 수동으로 저장해야 하는 것이 아닌 프레임워크가 맡아서 저장하기 때문에 별도로 저장할 필요가 없습니다.
- 인증이 실패하거나 권한이 부족하여 오류가 발생할 때 예외 처리 흐름을 필터 단에서 일관되게 처리할 수 있습니다.

### 인증 시스템 구조는 어떻게 변경되었는가?
기존에 구현한 인증 시스템을 프레임워크에 맞게 재구현하면서 구조가 변경되었습니다.
- OAuth2.0, OIDC를 한 메서드안에서 처리하였지만 프레임워크에서는 별도의 커스텀 서비스로 분리하여 설정하였습니다.
- login 메서드에서 인증이 실패, 오류가 발생하거나 권한을 만족하지 못하면 GlobalExceptionHandler로 처리하였지만 Spring Security에서는 커스텀한 AuthenticationEntryPoint와 AccessDeniedHandler를 구현하여 처리합니다.
- state, code_verifier, nonce 등의 보안 요소 검증을 직접 다루었지만 Spring SecurityAuthorizationRequestRepository가 관리 및 처리해줍니다.
- 액세스 토큰 발급 및 사용자 프로필 정보 조회를 WebClient를 사용하여 직접 질의하였지만 Spring Security에서는 OAuth2AccessTokenResponseClient과 OAuth2UserService가 맡아서 수행합니다.
	- OAuth2UserService 같은 경우에는 커스텀 서비스를 만들어서 설정하였습니다. 제가 만든 커스텀 OAUth2 서비스에서는 사용자 프로필 정보를 조회하는 것까지는 동일하지만 만약 기존 회원 정보가 없다면 회원을 생성하여 저장하는 것까지 수행합니다.
- login() 메서드에서 인증에 성공하면 JWT 생성하는 작업을 수행하였지만 Spring Security에서는 별도의 AuthenticationSuccessHandler를 구현하여 설정하였습니다.

다음 설명들은 위에서 정리한 설명들에서 커스텀하게 구현하여 구조가 변경된 부분을 작성하였습니다.

**사용자 프로필 정보 조회 구조 변경**
소셜 로그인하여 인증할 때 액세스 토큰을 발급받은 다음에 플랫폼마다 프로필 정보를 조회하는 방식이 다를 수 있습니다. 첫번째는 OAuth 2.0 방식으로 액세스 토큰을 이용하여 조회하는 일반적인 방식입니다. 두번째는 OIDC 방식으로써 액세스 토큰 발급과 같이 OpenID를 발급받아서 추가적으로 액세스 토큰으로 정보를 질의하는 것이 아닌 OpenID에 있는 프로필 정보를 사용하여 처리하는 방식입니다. 기존 인증 시스템에서는 한 메서드에 조건문을 분기하여 처리하였지만 Spring Security에서는 별도의 커스텀 서비스로 분리하여 설정하였습니다. 다음 코드는 OAuth2.0 방식으로 커스텀 서비스를 구현한 것입니다.
```java
@Slf4j  
@Service  
public class CustomOAuth2UserService extends AbstractUserService  
    implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {  
  
    //...
  
    @Override  
    @Transactional    
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {  
       OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService();  
       OAuth2User oAuth2User = delegate.loadUser(userRequest);  
       OAuthAttribute attributes = getUserInfo(userRequest, oAuth2User);  
       Member member = saveOrUpdate(attributes);  
       return createOAuth2User(member, userRequest, attributes.getSub());  
    }  
  
    @Override  
    OAuth2User createOAuth2User(Member member, OAuth2UserRequest userRequest, String sub) {  
       Collection<? extends GrantedAuthority> authorities = member.getSimpleGrantedAuthorities();  
       Map<String, Object> memberAttribute = member.toAttributeMap();  
       String nameAttributeKey = userRequest.getClientRegistration()  
          .getProviderDetails()  
          .getUserInfoEndpoint()  
          .getUserNameAttributeName();  
       memberAttribute.put(nameAttributeKey, sub);  
       return new DefaultOAuth2User(authorities, memberAttribute, nameAttributeKey);  
    }  
}
```
- 위 코드를 보면 OAuth2UserRequest 객체가 액세스 토큰을 가지고 있고 위임 객체를 통하여 프로필 정보를 조회합니다.
- CustomOAuth2UserService는 기본 서비스와 다르게 추가적으로 회원을 생성하거나 갱신하는 작업을 수행하고 있습니다.

**인증 또는 인가 실패 또는 오류 발생시 구조 변경**
login 메서드에서 인증이 실패, 오류가 발생하거나 권한을 만족하지 못하면 GlobalExceptionHandler로 처리하였지만 Spring Security에서는 커스텀한 AuthenticationEntryPoint와 AccessDeniedHandler를 구현하여 처리합니다. 다음 코드는 커스텀하게 구현한 AuthenticationEntryPoint 구현체입니다.
```java
@RequiredArgsConstructor  
@Slf4j  
public class CommonLoginAuthenticationEntryPoint implements AuthenticationEntryPoint {  
    private final ObjectMapper objectMapper;  
  
    @Override  
    public void commence(HttpServletRequest request, HttpServletResponse response,  
       AuthenticationException exception) throws IOException {  
       ErrorCode errorCode = ErrorCode.UNAUTHORIZED;  
       ApiResponse<String> body = ApiResponse.error(HttpStatus.UNAUTHORIZED, errorCode);  
       response.setStatus(HttpStatus.UNAUTHORIZED.value());  
       response.setContentType(MediaType.APPLICATION_JSON_VALUE);  
       response.setCharacterEncoding("utf-8");  
       response.getWriter().write(objectMapper.writeValueAsString(body));  
    }  
}
```
위 코드를 보면 인증에 실패하게 되면 Response 헤더 및 바디를 설정하는 것을 볼수 있습니다. AccessDeniedHandler의 구현체 또한 위와 비슷하게 구현하였습니다.

인증 성공 후 작업의 구조 변경
login 메서드에서 인증이 성공하면 Response 객체를 생성하여 반환하였습니다. 해당 객체에는 JWT 정보와 생성한 회원 객체가 포함되어 있습니다. Spring Security에서는 인증이 성공후에 별도의 SuccessHandler 구현체를 통하여 처리하게 됩니다. 코드는 다음과 같습니다.
```java
@Slf4j  
@RequiredArgsConstructor  
public class OAuth2SuccessHandler extends SimpleUrlAuthenticationSuccessHandler {  
  
    private final TokenService tokenService;  
    private final OAuth2UserMapper oauth2UserMapper;  
    private final String loginSuccessUri;  
    private final TokenFactory tokenFactory;  
  
    @Override  
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,  
       Authentication authentication) throws IOException {  
       OAuth2User oAuth2User;  
       Object principal = authentication.getPrincipal();  
       if (principal instanceof DefaultOidcUser defaultOidcUser) {  
          oAuth2User = defaultOidcUser;  
       } else {  
          oAuth2User = (OAuth2User)principal;  
       }  
       MemberAuthentication memberAuthentication = oauth2UserMapper.toMemberAuthentication(oAuth2User);  
       log.debug("oAuth2User : {}", oAuth2User);  
       log.debug("userDto : {}", memberAuthentication);  
  
       Token token = tokenService.generateToken(memberAuthentication, new Date());  
       log.debug("token : {}", token);  
  
       String redirectUrl = (String)request.getSession().getAttribute("redirect_url");  
       if (redirectUrl == null) {  
          redirectUrl = loginSuccessUri;  
       }  
  
       String targetUrl = UriComponentsBuilder.fromUriString(redirectUrl)  
          .queryParam("success", "true")  
          .build()  
          .toUriString();  
  
       CookieUtils.setCookie(response, tokenFactory.createAccessTokenCookie(token));  
       CookieUtils.setCookie(response, tokenFactory.createRefreshTokenCookie(token));  
  
       log.info("Member {} has successfully logged", memberAuthentication.getNickname());  
       getRedirectStrategy().sendRedirect(request, response, targetUrl);  
    }  
}
```
- 인증 성공후에는 인증 정보를 추출하고 JWT 정보를 Response에 저장하고 특정한 경로로 페이지 이동합니다.
- 기존 인증 시스템에서는 Response Body에 필요한 정보를 담아서 응답합니다.


## 정리하며
- 이번 리팩토링의 의의
- 얻은 교훈 또는 다음에 적용할 방향

Spring Security를 도입한 이유에 대해서 작성하였습니다.
기존에 구현한 소셜 로그인 인증 시스템을 Spring Security 프레임워크를 도입하면서 재구현한 이유를 작성하였습니다.
현재 구현한 인증 시스템의 문제점을 파악하고 Spring Security를 도입하면서 어떻게 구조를 개선하였는지 알아보았습니다.

얻은 교훈은 무엇인가?
현재 구현한 기능의 문제점이나 개선점을 파악할 수 있었습니다. Spring Security 프레임워크를 도입해야 할 필요성을 생각하고 프레임워크를 도입했을 때의 효과를 분석했습니다. 

다음에 적용할 방향은 무엇인가?
현재 인증 시스템에서 인증 상태를 저장하는 전략은 JWT 방식을 사용하고 있는데 만약 세션과 같은 방식으로 변경해야 한다고 할때 쉽게 변경할 수 있도록 구조 개선을 고려해볼 수 있습니다.




References
- source code : https://github.com/fine-ants/FineAnts-was/pull/41
