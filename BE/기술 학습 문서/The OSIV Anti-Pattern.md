OSIV(Open Session in View)는 영속성 컨텍스트(Persistence Context)를 뷰까지 열어준다는 뜻을 의미합니다. 영속성 컨텍스트가 살아있으면 엔티티는 영속 상태로 뷰 레이어까지 유지됩니다. 따라서 뷰(컨트롤러 레이어) 레이어에서도 **지연 로딩**을 할 수 있습니다. 


![[Pasted image 20240111143832.png]]
- `OpenSessionInViewFilter`(OSIVFilter)는 기본 `SessionFactory`의 `openSession` 메소드를 호출하고 새로운 `Session`을 가져옵니다.
- `Session`은 `TransactionSynchronizeationManager`로 바인딩됩니다.
- `OpenSessionInViewFilter` 은 `javax.servlet.FilterChain` 의 `doFilter` 를 호출하고 요청이 추가로 처리됩니다.
- `DispatcherServlet` 이 호출되고, HTTP 요청을 기본 `PostController`로 라우팅합니다.
- `PostController`는 `Post` 엔티티들의 리스트를 얻기 위해서 `PostService`를 호출합니다.
- 
