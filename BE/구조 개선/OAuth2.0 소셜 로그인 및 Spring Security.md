Spring Security 라이브러리를 추가하면서 소셜 로그인 기능을 다시 구현한 이유는 무엇인가요?

- [[#개요|개요]]
- [[#직접 구현한 소셜 로그인 방식|직접 구현한 소셜 로그인 방식]]
	- [[#직접 구현한 소셜 로그인 방식#직접 구현한 소셜 로그인의 수행 과정|직접 구현한 소셜 로그인의 수행 과정]]
	- [[#직접 구현한 소셜 로그인 방식#소셜 로그인을 직접 구현한 이유|소셜 로그인을 직접 구현한 이유]]
- [[#직접 구현 방식의 한계|직접 구현 방식의 한계]]
- [[#Spring Security 도입 배경|Spring Security 도입 배경]]
	- [[#Spring Security 도입 배경#도입 배경|도입 배경]]
	- [[#Spring Security 도입 배경#커스터마이징 소개|커스터마이징 소개]]
- [[#유지보수성과 확장성 측면에서의 개선 효과|유지보수성과 확장성 측면에서의 개선 효과]]
	- [[#유지보수성과 확장성 측면에서의 개선 효과#코드 구조 개선 효과|코드 구조 개선 효과]]
		- [[#코드 구조 개선 효과#state 검증 개선 비교|state 검증 개선 비교]]
		- [[#코드 구조 개선 효과#액세스 토큰 발급 개선 효과 비교|액세스 토큰 발급 개선 효과 비교]]
		- [[#코드 구조 개선 효과#사용자 프로필 조회 개선 비교|사용자 프로필 조회 개선 비교]]
		- [[#코드 구조 개선 효과#회원 생성 및 갱신 부분 비교|회원 생성 및 갱신 부분 비교]]
	- [[#유지보수성과 확장성 측면에서의 개선 효과#보안 정책 일관성 개선 효과|보안 정책 일관성 개선 효과]]
	- [[#유지보수성과 확장성 측면에서의 개선 효과#기능 확장 또는 정책 변경시 개선 효과|기능 확장 또는 정책 변경시 개선 효과]]
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
- 유지 보수의 어려움(예: 중복 코드, 보안 로직 누락 위험 등)
- 확장성과 일관성의 부족
- 테스트와 디버깅의 어려움

현제 제가 구현한 소셜 로그인 서비스의 문제점은 다음과 같았습니다.
- 하나의 login() 메서드에 state 검증, 액세스 토큰 발급, 사용자 프로필 조회, 회원 생성, JWT 생성 작업 등을 수행하고 있습니다. 이는 여러가지 책임이 하나의 메서드에 몰려 있습니다.
- 소셜 로그인 수행시 보안적인 요소를 수동적으로 처리하고 있습니다. state, code_verifier, nonce 등의 중요한 보안 요소를 직접 다루고 있습니다.
- 새로운 소셜 로그인 플랫폼이 추가되는 경우 수정이 필요합니다. 예를 들어 OauthClient를 관리하는 OauthClientRepository에 새로운 소셜 플랫폼을 추가하는 코드를 추가하여야 합니다. 그리고 소셜 플랫폼 정보를 가지고 있는 OauthClient 구현체 클래스를 확장해야 합니다.
- Spring Security 프레임워크를 적용하는 경우 기존 인증 시스템과의 통합이 부족합니다. Spring Security의 인증 처리 같은 경우에는 인증에 성공한 후 인증 정보를 `Authentication`이라는 객체에 담아서 SecurityContextHolder에 저장합니다. 이렇게 저장된 정보는 hasRole(), `@Secured` 애노테이션과 같은 인가 처리시 사용됩니다. 하지만 기존 인증 시스템에서는 이러한 저장 과정이 없기 때문에 통합하기 위해서는 인증후에 별도의 `Authentication`객체를 생성하여 SecurityContextHolder에 담는 구현을 해야 합니다. 
	- 즉, 정리하면 기존 인증 시스템을 



위 설명을 표로 정리하면 다음과 같습니다.

| 문제                     | 설명                                                 |
| ---------------------- | -------------------------------------------------- |
| 책임 집중                  | 인증, DB 처리, 토큰 발급 등 많은 책임이 한 메서드에 몰려 있음             |
| 보안 요소 수동 처리            | state, code_verifier, nonce 등 중요한 보안 요소를 직접 다루고 있음 |
| provider 추가 어려움        | 새로운 소셜 로그인 추가시 서비스 로직 수정 필요                        |
| Spring Security와 통합 부족 | 필터 기반 인증 처리 흐름과 통합되지 않아서 인증 상태 관리가 불안정함            |


## Spring Security 도입 배경
- 어떤 구조로 변경했는지
- 커스터마이징한 부분이 있다면 간략히 소개
- 도입 과정에서 겪은 이슈나 고민

### 도입 배경
Spring Security 프레임워크를 도입을 고려한 배경은 API의 경로별 접근 권한 때문이었습니다. 서버가 커질수록 단순 인증한 사용자들이 사용하는 API뿐만 아니라 관리자 권한을 요구하는 API도 필요하게 되었습니다. 이 상황에서 인가 처리 시스템 또한 직접 구현할 지 Spring Security 프레임워크를 적용하여 구현할지 고민하였습니다. 결과적으로 Spring Security 프레임워크를 적용하여 문제를 해결하기로 하였습니다. 도입하게 된 이유는 다음과 같았습니다.
- `@Secured`, `hasRole()`과 같은 설정을 사용해서 API 경로별 접근 제어가 쉽습니다.
- OAuth 2.0, OIDC 기반 소셜 로그인을 표준화된 방식으로 처리합니다.
	- 기존 구현한 코드같은 경우에는 provider별로 조건문을 통해서 OAuth 2.0, OIDC를 처리하고 있습니다.
- 소셜 로그인에서 추가적인 플랫폼이 추가되어도 기존 코드를 수정하지 않고 확장이 쉽습니다.
- 인증이 실패하거나, 권한이 부족하여 오류가 발생할 때 예외 처리 흐름을 필터 단에서 일관되게 처리합니다.

### 커스터마이징 소개
예를 들어 Spring Security 프레임워크를 적용하여 다음과 같이 API 경로별 접근 권한 제어를 쉽게 설정할 수 있습니다.
```java
@Bean  
@Order(2)  
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {  
    http  
       .authorizeHttpRequests(authorize ->  
          authorize  
             .requestMatchers(  
                "/oauth2/authorization/**",  
                "/login/oauth2/code/**",  
                "/api/oauth/redirect",  
                "/api/auth/signup",  
                "/api/auth/signup/duplicationcheck/nickname/**",  
                "/api/auth/signup/duplicationcheck/email/**",  
                "/api/auth/signup/verifyEmail",  
                "/api/auth/signup/verifyCode",  
                "/api/auth/refresh/token",  
                "/api/stocks/search",  
                "/api/stocks/**",  
                "/health-check",  
                "/error"  
             ).permitAll()  
             .anyRequest().authenticated());
    // ...
}
```
위와 같은 설정을 보면 여러가지 API 경로와 경로 패턴이 존재하는데 위 경로들은 인증이 필요치 않고 이용할 수 있는 API들입니다. 그 외의 API 경로들은 인증이 요구됩니다.

이번에는 특정 컨트롤러의 API 메서드입니다.
```java
@RestController  
@RequestMapping("/api/exchange-rates")  
@RequiredArgsConstructor  
public class ExchangeRateRestController {
	// ...
    @GetMapping  
    @Secured(value = {"ROLE_MANAGER", "ROLE_ADMIN"})  
    public ApiResponse<ExchangeRateListResponse> readExchangeRates() {  
       ExchangeRateListResponse response = service.readExchangeRates();  
       return ApiResponse.success(ExchangeRateSuccessCode.READ_EXCHANGE_RATE, response);  
    }
}
```
readExchangeRates() 메서드는 환율 정보들을 조회하는 메서드입니다. `@Secured` 애노테이션을 설정해서 해당 API는 매니저 또는 관리자 권한을 가진 사용자만 접근할 수 잇습니다. 위와 같이 Spring Security 프레임워크를 도입하게 되면 hasRole()을 이용해서 경로별로 접근 권한을 설정할 수 있거나 아니면 컨트롤러 메서드에 `@Secured` 애노테이션을 사용하여 요구되는 권한을 설정할 수 있습니다.

다음은 OAuth 2.0, OIDC 기반 소셜 로그인 방식을 표준화하여 처리하기 위한 설정 코드입니다.
```java
@Bean  
@Order(2)  
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {  
    // ...
    http  
       .oauth2Login(configurer -> configurer  
          .userInfoEndpoint(config -> config  
             .userService(customOAuth2UserService())  
             .oidcUserService(customOidcUserService())  
          )  
          .successHandler(oauth2SuccessHandler()));  
    // ...
    return http.build();  
}
```
- 설정을 보면 OAuth 2.0 로그인 설정에서 별도의 customOAuth2UserService와 customOidcUserService를 주입해서 처리하는 것을 볼수 있습니다.
- custom으로  시작하는 서비스는 개발자가 직접 구현해서 주입해야 합니다.

예를 들어 customOAuth2UserService의 구현은 다음과 같습니다.
```java
@Slf4j  
@Service  
public class CustomOAuth2UserService extends AbstractUserService  
    implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {  
  
    public CustomOAuth2UserService(MemberRepository memberRepository,  
       NotificationPreferenceRepository notificationPreferenceRepository,  
       NicknameGenerator nicknameGenerator, RoleRepository roleRepository) {  
       super(memberRepository, notificationPreferenceRepository, nicknameGenerator, roleRepository);  
    }  
  
    @Override  
    @Transactional    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {  
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
- loadUser() 메서드에서 OAuth2UserRequest 객체를 매개변수로 받아서 delegate.loadUser() 메서드 실행시 전달하여 액세스 토큰을 이용하여 프로필 정보를 받아옵니다. 

다음 읿부 메서드 코드는 OAuth2LoginAuthenticationProvider 클래스의 인증 메서드입니다. this.userService를 보면 OAuth2UserService 인터페이스로써 프로필 정보를 가져옵니다. 이와 같이 로그인하는 소셜 플랫폼이 OAuth2.0 기반인 경우에는 다음과 같이 표준화되어 처리됩니다. OIDC 기반 같은 경우에는 OidcAuthorizationCodeAuthenticationProvider 라는 클래스에서 별도로 표준적으로 처리됩니다.
```java
@Override  
public Authentication authenticate(Authentication authentication) throws AuthenticationException {     // ...
    OAuth2User oauth2User = this.userService.loadUser(new OAuth2UserRequest(  
          loginAuthenticationToken.getClientRegistration(), accessToken, additionalParameters));  
    Collection<? extends GrantedAuthority> mappedAuthorities = this.authoritiesMapper  
       .mapAuthorities(oauth2User.getAuthorities());  
    OAuth2LoginAuthenticationToken authenticationResult = new OAuth2LoginAuthenticationToken(  
          loginAuthenticationToken.getClientRegistration(), loginAuthenticationToken.getAuthorizationExchange(),  
          oauth2User, mappedAuthorities, accessToken, authorizationCodeAuthenticationToken.getRefreshToken());  
    authenticationResult.setDetails(loginAuthenticationToken.getDetails());  
    return authenticationResult;  
}
```

다음은 Spring Security를 도입하여 필터단에서 인증/인가 처리를 수행하던 중에 오류가 발생하면 어떻게 일관되게 예외 처리를 수행하는지 확인합니다.
```java
@Bean  
@Order(2)  
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {  
    // ...
    http.exceptionHandling(configurer -> configurer  
       .authenticationEntryPoint(commonLoginAuthenticationEntryPoint)  
       .accessDeniedHandler(customAccessDeniedHandler()));  
	// ...
    return http.build();  
}
```
- 위 설정을 보면 인증 관련해서 오류 처리는 commonLoginAuthenticationEntryPoint 객체가 처리하고 인가 관련한 오류 처리는 customAccessDeniedHandler가 처리하는 것을 알 수 있습니다.

예를 들어 필터 단에서 인증 오류가 발생하면 예외 처리되서 위에서 주입한 commonLoginAuthenticationEntryPoint로 전달됩니다.
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
- 코드를 보면 클라이언트에게 응답하기 위해서 헤더 설정을 하고 Body에 에러 객체를 직렬화하여 설정하는 것을 볼수 있습니다.

인가 관련해서 오류가 발생하면 customAccessDeniedHandler로 전달되어 다음과 같이 처리됩니다.
```java
public class CustomAccessDeniedHandler implements AccessDeniedHandler {  
  
    @Override  
    public void handle(HttpServletRequest request, HttpServletResponse response,  
       AccessDeniedException exception) throws IOException {  
       ApiResponse<Object> body = ApiResponse.of(HttpStatus.FORBIDDEN, exception.getMessage(),  
          exception.toString());  
       response.setContentType(MediaType.APPLICATION_JSON_VALUE);  
       response.setStatus(HttpStatus.FORBIDDEN.value());  
       response.getWriter().write(body.toString());  
       response.getWriter().flush();  
       response.getWriter().close();  
    }  
}
```
- 위 구현을 보면 HTTP Response에 헤더 및 Body를 설정하는 것을 볼수 있습니다.

위와 같이 Spring Security 프레임워크를 적용하면 API 경로별 접근 권한 제어를 쉽게 할 수 있고 소셜 로그인 구현시 OAuth2.0, OIDC 기반을 구분하지 않고 표준화되어 처리할 수 있습니다. 그리고 인증 및 인가 처리시 별도의 핸들러를 주입하면 오류를 일관되게 처리할 수 있습니다.

## 유지보수성과 확장성 측면에서의 개선 효과
- 코드 구조 정리
	- 기존에 서비스 클래스 한곳에서 처리하던 복잡한 인증 흐름(토큰 발급, 사용자 정보 조회, JWT 생성 등)을 Spring Security의 구성 요소로 분리한점을 강조하세요.
	- AuthenticationFilter, OAuth2UserService, AuthenticationSuccessHandler 등을 활용해 관심사 분리(SRP)가 잘 이루어졌음을 설명하세요.
	- 그 결과 테스트하기 쉬워졌고, 한 기능의 변경이 다른 부분에 영향을 주지 않게 됬다는 효과를 언급하세요.
- 보안 정책 일관성
	- state, nonce, code_verifier 등 보안 필드를 직접 구현할 때의 오류 가능성과 유지 부담을 강조하세요.
	- Spring Security는 OAuth 2.0 / OIDC에서 요구하는 보안 요소를 자동으로 관리하고 있으므로, 보안 누락 가능성을 줄이고 일관된 정책을 적용할 수 있다는 점을 쓰세요.
	- 예외 처리, 인증 실패 응답 같은 보안 이벤트도 일관되게 처리할 수 있다는 점도 포함시키면 좋습니다.
- 기능 확장이나 정책 변경 시 유연함
	- 새로운 소셜 로그인 플랫폼 추가시 필요한 작업이 간단한 설정 또는 컴포넌트 추가(확장)만으로 해결된다는 점을 언급하세요.
	- 인증 정책이 변경되더라도 개별 컴포넌트만 수정하면 되므로 전체 시스템의 안정성을 해치지 않기때문에 빠르게 적용이 가능하다는 점을 쓰세요.
		- 인증 정책 변경의 예시로 JWT 토큰 구조 변경, 로그인 성공 후 리다이렉션 동작 변경, 로그인 실패에 대한 처리 방식 변경, OAuth Provider 인증 흐름 커스터마이징, JWT 저장소 변경, 사용자 인증 방식 변경
	- 예: 토큰 저장소 변경, 로그인 성공 후 리다이렉션 경로 변경, 사용자 정보 매핑 방식 변경 등이 쉬워졌다는 것을 강조하세요.

## 마무리 및 회고
- 이번 리팩토링의 의의
- 얻은 교훈 또는 다음에 적용할 방향


References
- source code : https://github.com/fine-ants/FineAnts-was/pull/41
