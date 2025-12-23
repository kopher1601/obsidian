- `AuthenticationProvider` 는 인증 논리를 정의하고 사용자와 암호의 관리를 위임한다.
- `AuthenticationProvider` 는 인증 논리를 구현한다. 사용자의 암호를 검증하기 위해 [[PasswordEncoder]] 가 필요하다
- 사용자를 찾은 후 `AuthenticationProvider` 는 [[PasswordEncoder]] 를 이용해 암호를 검증한다
- `AuthenticationProvider` 인터페이스의 기본 구현은 시스템의 사용자를 찾는 책임을 [[UserDetailsService]] 에 위임하고 [[PasswordEncoder]] 로 인증 프로세스에서 암호를 관리한다. 
- `AuthenticationProvider` 책임은 [[Authentication]] 계약과 강하게 결합돼 있다. 
## AuthenticationProvider 의 재정의
[[UserDetailsService]] 와 [[PasswordEncoder]] 두 개의 빈을 꼭 이용해야 하는 것은 아니지만, 인증을 위해 사용자와 암호를 관리한다면 논리를 분리하는것이 좋다. 즉 인증 구현을 재정의할 때도 스프링 시큐리티 아키텍처의 설계를 따르는 것이 좋다.
```kotlin
@Component  
class CustomAuthenticationProvider: AuthenticationProvider {  
    override fun authenticate(authentication: Authentication?): Authentication {  
        val username = authentication?.name  
        val password = authentication?.credentials.toString()  
  
        // 여기의 if-else 절의 조건은 UserDetailsService 및 
        // PasswordEncoder 의 책임을 대체하고 있다.  
        if (username == "john" && password == "12345") {  
            return UsernamePasswordAuthenticationToken(
			            username, 
			            password, 
			            arrayListOf())  
        } else {  
            throw AuthenticationCredentialsNotFoundException(
		            "Error in authentication!")
        }  
    }  
  
    override fun supports(authentication: Class<*>?): Boolean {  
        return UsernamePasswordAuthenticationToken::class
			        .java
			        .isAssignableFrom(authentication)  
    }  
}
```

## support() 메서드의 목적 
![[Pasted image 20251029101900.png]]
2 번의 경우가 `support` 는 `true` 를 반환하지만 `authenticate` 는 `null` 을 반환하는 예이다.
