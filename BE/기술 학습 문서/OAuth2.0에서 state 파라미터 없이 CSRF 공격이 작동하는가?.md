1. Mallory는 어떤 클라이언트의 웹사이트(e.g. [https://brilliantphotos.com](https://brilliantphotos.com/)에 방문하고 OAuth를 사용하여 어떤 service provider에 접근하기 위한 클라이언트의 인증 프로세스를 시작합니다. (e.g. Acebook - brilliantphotos.com에서 사용자가 사진을 Acebook 페이지에 게시할 수 있도록 합니다.)

2. brilliantphotos.com 서버는 인증이 완료되면 Mallory의 브라우저를 다시 brilliantphotos.com 서버로 리다이렉션하는 요청을 Acebook의 Authorization Server로 요청합니다.

3. Mallory는 Acebook 사용자이름/비밀번호를 입력하여 접근 권한을 부여하는 Authorization Server로 리다이렉션합니다.

4. 성공적인 로그인 이후, Mallory는 후속 리다이렉션 요청을 트랩/방지하고 URL(Mallory와 관련된 인증 코드가 있는 콜백 URL)을 저장합니다. 예를 들어 다음과 같습니다.
- https://brilliantphotos.com/exchangecodefortoken?code=malloryscode

