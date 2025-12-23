[[AuthenticationManager]] 는 인증 프로세스를 성공적으로 완료한 후 요청이 유지되는 동안 [[Authentication]] 인스턴스를 저장한다. Authentication 객체를 저장하는 인스턴스를 `SecurityContext` 라고 한다.
![[Pasted image 20251029203400.png]]
`SecurityContext` 의 주 책임은 `Authentication` 객체를 저장하는 것이다. 그렇다면 `SecurityContext` 자체는 어떻게 관리될까? 스프링 시큐리티 관리자 역할을 하는 객체로 `SecurityContext` 를 관리하는 세 가지 전략을 제공한다. 이 객체를 [[SecurityContextHolder]] 라고 한다.


## 주의점
개발자가 스프링에 알려지지 않은 스레드를 정의하면 `SecurityContext` 의 세부 정보를 명시적으로 새 스레드로 복사해야 한다. 

