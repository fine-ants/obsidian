1. Mallory는 어떤 클라이언트의 웹사이트(e.g. [https://brilliantphotos.com](https://brilliantphotos.com/)에 방문하고 OAuth를 사용하여 어떤 service provider에 접근하기 위한 클라이언트의 인증 프로세스를 시작합니다. (e.g. Acebook - brilliantphotos.com에서 사용자가 사진을 Acebook 페이지에 게시할 수 있도록 합니다.)

2. brilliantphotos.com 서버는 인증이 완료되면 Mallory의 브라우저를 다시 brilliantphotos.com 서버로 리다이렉션하는 요청을 Acebook의 Authorization Server로 요청합니다.

3. Mallory는 Acebook 사용자이름/비밀번호를 입력하여 접근 권한을 부여하는 Authorization Server로 리다이렉션합니다.

4. 성공적인 로그인 이후, Mallory는 후속 리다이렉션 요청을 트랩/방지하고 URL(Mallory와 관련된 인증 코드가 있는 콜백 URL)을 저장합니다. 예를 들어 다음과 같습니다.
- https://brilliantphotos.com/exchangecodefortoken?code=malloryscode

5. 이제 Mallory는 어떻게든 Alice가 그 URL을 방문하도록 유도합니다. (아마도 포럼 게시물의 링크 같은 것으로..) Alice는 이미 자신의 계정으로 brilliantphotos.com에 로그인 되어 있을 수 있습니다.

6. Alice는 brilliantphotos.com으로 가는 링크를 클릭하고 Authorization Code는 Access Token으로 교환됩니다. (장난스러운 Mallory의 계정에 대한 access). 만약 Alice가 로그인 되어 있으면, brilliantphotos.com는 Alice의 계정을 새로 만든 Access Token과 연결할 수 있습니다.

7. 이제 Alice가 brilliantphotos.com 웹사이트를 계속 이용할 경우, 클라이언트는 실수로 service provider(Acebook)의 Mallory 계정에 사진을 게시할 수 있습니다.

만약 state 파라미터가 brilliantphotos.com 웹사이트에 의해서 유지되고 있다면, Mallory의 state 코드는 Mallory의 브라우저에는 바이딩되지만, Alice의 state 코드는 바인딩 되지 않습니다. 따라서 Alice가 악성 URL을 클릭하였을 때 brilliantphotos.com 웹사이트는 state 코드와 Alice의 브라우저 세션을 연관시킬수 없습니다.

위 OAuth2.0에서 CSRF 공격에 대한 흐름을 그림으로 표현하면 다음과 같을 수 있습니다.
![[Pasted image 20231210233654.png]]

