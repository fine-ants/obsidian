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

만약 GET 형식ㅇ