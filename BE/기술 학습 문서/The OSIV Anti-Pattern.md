OSIV(Open Session in View)는 영속성 컨텍스트(Persistence Context)를 뷰까지 열어준다는 뜻을 의미합니다. 영속성 컨텍스트가 살아있으면 엔티티는 영속 상태로 뷰 레이어까지 유지됩니다. 따라서 뷰(컨트롤러 레이어) 레이어에서도 **지연 로딩**을 할 수 있습니다. 

![img](./'Pasted image 20240111143832.png)


