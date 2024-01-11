OSIV(Open Session in View)는 영속성 컨텍스트(Persistence Context)를 뷰까지 열어준다는 뜻을 의미합니다. 영속성 컨텍스트가 살아있으면 엔티티는 영속 상태로 뷰 레이어까지 유지됩니다. 따라서 뷰(컨트롤러 레이어) 레이어에서도 **지연 로딩**을 할 수 있습니다. 


![[Pasted image 20240111143832.png]]
- `OpenSessionInViewFilter`(OSIVFilter)는 기본 `SessionFactory`의 `openSession` 메소드를 호출하고 새로운 `Session`을 가져옵니다.
- `Session`은 `TransactionSynchronizeationManager`로 바인딩됩니다.
- `OpenSessionInViewFilter` 은 `javax.servlet.FilterChain` 의 `doFilter` 를 호출하고 요청이 추가로 처리됩니다.
- `DispatcherServlet` 이 호출되고, HTTP 요청을 기본 `PostController`로 라우팅합니다.
- `PostController`는 `Post` 엔티티들의 리스트를 얻기 위해서 `PostService`를 호출합니다.
- `PostService`는 새로운 트랜잭션(Transaction)을 열게 되고 `HibernateTransactionManager`는 `OpenSessionInViewFilter`이 열었던 것과 동일한 세션을 재사용합니다.
- `PostDAO`는 어떤 지연 연결을 초기화하지 않고 `Post` 엔티티들의 리스트를 가져옵니다.
- `PostService`는 기본적인 트랜잭션을 커밋하지만 `Session`은 외부에서 열렸기 때문에 닫히지 않습니다.
- `DispatcherServlet`는 UI 렌더링을 시작하고, UI 렌더링은 다시 지연 연결을 탐색하고 초기화를 트리거합니다.
- `OpenSessionInViewFilter`는 세션을 닫을 수 있으며, 기본 데이터베이스 연결도 해제됩니다.

언뜻 보기에는 오류가 나지 않을것 같지만 데이터베이스 관점에서 보면 이 결함이 분명해지기 시작합니다. 

서비스 레이어는 데이터베이스 트랜잭션을 열고 닫지만, 그 이후에는 
