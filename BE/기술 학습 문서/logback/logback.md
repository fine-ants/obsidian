
## RollingPolicy
### RollingPolicy 개념
- RollingPolicy는 로그 파일이 크기, 시간 또는 다른 조건을 기준으로 자동으로 롤오버되도록 설정하는 방식입니다.
	- #롤오버 : 로그 파일이 특정 조건(예: 크기, 시간 등)에 도달 시 새로운 로그 파일로 전환되는 과정
- 새로운 로그 파일이 생성되면, 이전 파일은 보관되거나 삭제될 수 있습니다.
	- 예를 들어 로그 파일이 일정 크기(maxFileSize)에 도달하거나, 특정 시간(maxHistory)에 도달하면 새로운 로그 파일이 생성되며, 기존 파일은 관리됩니다.

### RollingPolicy 종류
- SizeBasedRollingPolicy
	- 로그 파일이 설정된 크기(maxFileSize)에 도달할 때 새로운 로그 파일로 롤오버됩니다.
- TimeBasedRollingPolicy
	- 로그 파일이 설정된 시간 단위(maxHistory)에 도달할 때 새로운 로그 파일로 롤오버됩니다.
- SizeAndTimeBasedRollingPolicy
	- 크기와 시간 기준을 동시에 결합하여 로그 파일이 롤오버됩니다.
	- 예: 특정 크기(maxFileSize)에 도달하거나, 시간 기준(maxHistory)에 도달하면 새로운 로그 파일로 전환됩니다.
- FixedWindowRollingPolicy
	- 설정된 일정 수의 로그 파일을 보유하고 그 이상이 되면 오
