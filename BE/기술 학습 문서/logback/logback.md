
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
	- 설정된 일정 수의 로그 파일을 보유하고 그 이상이 되면 오래된 로그 파일을 삭제합니다.
- CronTriggeringPolicy
	- 특정 시간에 로그 파일이 롤오버되며, 주기적으로 새로운 로그 파일이 생성됩니다.

### Rolling 설정 예시
```xml
<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
    <fileNamePattern>${LOG_FILE_PATH}/app-%d{yyyy-MM-dd_HH}.log</fileNamePattern>
    <maxHistory>30</maxHistory>          <!-- 최대 보관일 수 -->
    <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <maxFileSize>10MB</maxFileSize>    <!-- 최대 파일 크기 -->
    </timeBasedFileNamingAndTriggeringPolicy>
</rollingPolicy>
```

#### timeBasedFileNamingAndTriggeringPolicy 클래스 종류
- ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP
	- **크기 기준과 시간 기준**을 동시에 적용하며 로그 파일 이름을 시간 기반으로 자동 생성합니다.
	- 예: `app-yyyy-MM-dd_HH.log`
- ch.qos.logback.core.rolling.DefaultTimeBasedFileNamingAndTriggeringPolicy
	- 기본 시간 기반 파일 명명 및 롤오버 정책으로, 로그 파일이 **특정 시간 기준**에 따라 자동 생성됩니다.
	- 예: `app-yyyy-MM-dd.log`
- ch.qos.logback.core.rolling.CronTriggeringPolicy
	- 특정 크기 및 시간 외에 크론 표현식을 통해 주기적으로 로그 롤오버를 관리합니다.
	- 예: 매일 특정 시간(예: `0 0 0 * * ?`)에 로그 파일 생성
- ch.qos.logback.core.rolling.TimeBasedFileNamingAndTriggeringPolicy
	- 기본 시간 기반 파일 명명 정책으로, 파일명이 시간 기준(`yyyy-MM-dd_HH`)로 생성됩니다.

**SizeAndTimeBasedFNATP**
```xml
<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
    <maxFileSize>10MB</maxFileSize>
</timeBasedFileNamingAndTriggeringPolicy>
```

**DefaultTimeBasedFileNamingAndTriggeringPolicy**
```xml
<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.DefaultTimeBasedFileNamingAndTriggeringPolicy">
    <fileNamePattern>${LOG_FILE_PATH}/app-%d{yyyy-MM-dd}.log</fileNamePattern>
</timeBasedFileNamingAndTriggeringPolicy>
```
**CronTriggeringPolicy**
```xml
<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.CronTriggeringPolicy">
    <cronExpression>0 0 0 * * ?</cronExpression>
</timeBasedFileNamingAndTriggeringPolicy>
```
