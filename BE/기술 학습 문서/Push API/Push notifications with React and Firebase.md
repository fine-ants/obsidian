## Sample React app setup
fire_client라는 이름의 리액트 프로젝트를 생성합니다.
```
$ create-react-app fire_client
```

Firebase Cloud Messaing의 클라이언트 사이드 기능을 위한 의존성을 추가합니다.
```
$ npm install --save firebase react-bootstrap bootstrap
```


## Firebase setup
이번 단계에서는 Firebase 프로젝트를 생성합니다.

1. 프로젝트 추가를 클릭합니다.
![[Pasted image 20240131133011.png]]

2. 프로젝트 이름을 입력합니다.
![[Pasted image 20240131133131.png]]

3. Google 애널리스틱 구성을 선택합니다.
![[Pasted image 20240131133226.png]]

4. 새로 만든 프로젝트 개요 페이지에서 웹 앱을 클릭하여 앱을 추가합니다.
![[Pasted image 20240131133358.png]]

5. 앱 닉네임을 입력합니다.
![[Pasted image 20240131133431.png]]

6. 앱 추가를 확인합니다.
![[Pasted image 20240131133540.png]]

7. firebase.js 파일을 생성하고 다음과 같이 Firebase 설정을 입력합니다.
