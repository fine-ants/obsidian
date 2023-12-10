## CSRF란 무엇인가?
CSRF란 Cross Site Request Forgery의 약자로, 사이트간 요청 위조를 뜻합니다. CSRF는 웹 보안 취약점의 일종이며, **사용자가 자신의 의지와 무관하게 공격자가 의도한 행위(데이터 수정, 삭제, 등록 등)를 특정 웹사이트에 요청하게 하는 공격**입니다.

예를 들어, 피해자의 전자 메일 주소를 변경하거나 암호를 변경하거나 자금 이체를 하는 등의 동작을 수행하게 할 수 있습니다. 특성에 따라 공격자는 사용자 계정에 대한 완전한 제어권을 얻을 수도 있습니다.

### CSRF 동작원리
CSRF가 성공하려면, 아래 3가지 조건을 만족되어야 합니다.
1. 사용자는 보안이 취약한 서버로부터 이미 로그인되어 있는 상태여야 합니다.
2. 쿠키 기반의 서버 세션 정보를 흭득할 수 있어야 합니다.
3. 공격자는 서버를 공격하기 위한 요청 방법에 대해 미리 파악하고 있어야 합니다. 또한 예상하지 못한 요청 매개변수가 없어야 합니다.

위 조건이 만족되면, 다음과 같은 과정을 통해 CSRF 공격을 할 수 있습니다.

1. 사용자는 보안이 취약한 서버에 로그인합니다.
2. 서버에 저장된 세션 정보를 사용할 수 있는 session ID가 사용자의 브라우저 쿠키에 저장됩니다.
3. 공격자는 사용자가 악성 스크립트 페이지를 클릭하도록 유도합니다.
- 악성 스크립트 페이지 클릭 유도 방식 예시
	- 게시판이 있는 웹사이트에 악성 스크립트를 게시글로 작성하여 사용자들이 게시글을 클릭하도록 유도
	- 메일 등으로 악성 스크립트를 직접 전달하거나, 악성 스크립트가 적힌 페이지 링크를 전달
4. 사용자가 악성 스크립트가 작성된 페이지 접근시 웹 브라우저에 의해 쿠키에 저장된 sessionID와 함께 취약한 서버로 패스워드 변경과 같은 요청을 합니다.
5. 서버는 쿠키에 담긴 sessionID를 통해 해당 요청이 인증된 사용자로부터 온것으로 판단하고 해커가 요청한 비밀번호 변경을 처리합니다.

![[Pasted image 20231210151143.png]]

예를 들어 취약점이 있는 웹사이트에서 패스워드를 변경 API의 요청 형식이 다음과 같다고 가정하겠습니다.
```http
POST /password/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

password=passw0rd
```

해커는 아래와 같은 html 문서를 사용자에게 열도록 유도하기만 하면, 해커가 비밀번호를 변경하고자 하는 "mypassword" 값으로 취약한 웹 사이트에 비밀번호 변경 요청을 합니다.


```HTML
<form action="https://vulnerable-website.com/password/change" method="POST">
	<input type="hidden" name="password" value="mypassword">
</form>
<script>          
document.forms[0].submit();
</script>
```

만약 GET 형식으로도 간단히 처리할 수 있는 것이라면, 아래와 같이 img 태그가 포함된 글을 보기만 해도 로그아웃 시킬수 있습니다.
```HTML
<img src="http://vulnerable-website.com/logout" />
```
- img 태그로 요청된 src 주소는 GET 요청으로 처리됩니다.

### CSRF 방어방법 - 사용자 입장
- 사용자 입장에서는 이상한 URL을 함부로 클릭하지 않고, 의심이 되는 메일을 함부로 열어보지 않습니다.
- CSRF는 내가 클릭만 해도 내가 의도하지 않은 action이 발생할 수 있습니다.


### CSRF 방어방법 - 웹개발자/운영자 입장
#### 1. Referer check(리퍼러 체크)
HTTP Request Header에서 Referer 정보를 확인할 수 있습니다. 보통이라면 호스트(host)와 Referer의 값이 일치하므로 둘을 비교합니다. CSRF 공격의 대부분이 Referer 값에 대한 검증만으로도 많은 수의 공격을 방어할 수 있다고 합니다. Java Servelt을 사용한다면 아래와 같이 interceptor 클래스를 만들어 모든 요청에 대해 Referer Check 할 수 있도록 방어가 가능합니다.

```Java
public class ReferrerCheck implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String referer = request.getHeader("Referer");
        String host = request.getHeader("host");
        if (referer == null || !referer.contains(host)) {
            response.sendRedirect("/");
            return false;
        }
        return true;
    }
}
```

> [!info] 
> Referer
> Referer HTTP Request Header에는 리소스가 요청된 절대 주소 또는 부분 주소가 포함되어 있습니다. Referer 헤더를 통해 서버는 사용자가 방문하거나 요청된 리소스가 사용되는 참조 페이지를 식별할 수 있습니다. Referer 헤더의 값은 분석, 로깅, 최적화된 캐싱 등에 사용할 수 있습니다.
> 여러분들이 링크를 클릭했을 때, Referer는 링크가 포함된 페이지의 주소를 포함하고 있습니다. 여러분들이 다른 도메인에 리소스를 요청할 때 Referer에는 요청한 리소스를 사용하는 페이지의 주소가 포함됩니다
> Referer 헤더에는 origin, path 및 

