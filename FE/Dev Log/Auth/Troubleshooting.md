
# Auth Troubleshooting

## Table of Contents


## 팝업이 안 닫히는 문제
- Client URL의 끝에 `/` 제거 등 변수 값 확인.

## 팝업이 닫혀버리는 문제
- 문제?
	- `window.opener.postMessage`가 실행이 되지 않았지만 message handler가 이벤트를 어디선가 받아서 실행이 됨. 기존에는 핸들러에서 `closePopUpWindow()`를 실행하고 있었기 때문에 popup이 바로 꺼짐.
	- Incognito 모드에서는 문제가 발생이 안됨.
	- 그래서 진짜 문제의 원인은?
		- 
- 해결
	- 
	- PR: https://github.com/fine-ants/frontend/pull/63


- https://chat.openai.com/share/289933c5-67dc-440f-840d-df8a247d689d

