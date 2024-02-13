## 시나리오 1

  1. 기존에 데스크탑 알림을 server 값을 true이고 권한도 granted이다.

  2. 유저가 브라우저(크롬) setting에서 권한을 denied로 변경해 버렸다.
	 1. denied가 아니라 default일 경우도 비슷하다(4-2)

 3. 해당 dialog에 접근했을 경우 server에 알림 값은 true이지만 권한은 denied가 되어있다.

 4. 이 상황에서 유저가 submit을 하지 않으면 server 값을 계속 true가 되어 있는데 이 상황이 맞을까?
	1. 그럼 dialog에 접근했을 경우 server 값은 true이지만 권한이 denied면로직에서 server 값을 false로 변경해서 submit해야 할까?
		- 권한에 의해서 알림을 받지 못하는 상황인데 server에서는 true이다
		- 유저의 action에 대한 값을 받지 못했기 때문에 submit하는건 좀 이상적이지 못한 느낌이다.
		-  denied이면 권한 허용 요청을 할 수도 없다.
		- 화면에 toggle을 지우고, 차단되었다는 문구를 보여주며 직접 브라우저(크롬)에서 허용을 유도하는 방법이 괜찮을 것 같다.
	 2. default로 변경되어 있지만 server 값은 true이다 그렇다면 dialog에 접근하자 마자 권한 요청을 보내야 할까?
		- 권한 요청에 대해서 차단을 해버리면 denied 값을 알 수있다. server 값은 true이지만 유저가 차단해버림
		- server 값을 false로 변경해줘야 하는가?
		- 유저에게 권한 요청에 대한 값을 받았기 때문에 false로 submit해도 괜찮을 것 같다.

### 결론
####  유저에게 권한 요청을해서 받은 값이 아닌 유저 개인적으로 브라우저에서의 권한 변경값이 바뀌었다고 로직내에서 임의로 submit을 하는 건 이상적이지 못한것 같다.