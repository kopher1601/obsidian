인증이 실패했을 때의 응답을 커스텀 구성 하려면 `AuthenticationEntryPoint` 를 구현하면 된다. 
`AuthenticationEntryPoint` 인터페이스는 인증이 실패했을 때 이용된다는 점에서 이름의 의미에 약간 혼동의 소지가 있다. 이 인터페이스는 스프링 시큐리티 아키텍처에서 [[ExceptionTranslationManager]] 라는 구성 요소에서 직접 사용되며, 그 구성 요소는 필터 체인에서 투척된 모든 `AccessDeniedException` 및 `AuthenticationException` 을 처리한다. `ExceptionTranslationManager` 는 자바 예외와 HTTP 응답을 연결하는 다리라고 할 수 있다.

**구현**
```kotlin
class CustomEntryPoint: AuthenticationEntryPoint {  
    override fun commence(  
        request: HttpServletRequest?,  
        response: HttpServletResponse?,  
        authException: AuthenticationException?  
    ) {  
        response?.addHeader("message", "Luke, I am your father!")  
        response?.sendError(HttpStatus.UNAUTHORIZED.value(), HttpStatus.UNAUTHORIZED.reasonPhrase)  
    }  
}
```

**설정**
```kotlin
@Configuration
class ProjectConfig {

    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http.httpBasic{
            it.realmName("OTHER")
            // 이름에 혼동의 여지가 있지만 401 에러가 났을 경우 불리는 곳이다.
            it.authenticationEntryPoint(CustomEntryPoint())
        }
        http.authorizeHttpRequests{
            it.anyRequest().authenticated()
        }
        return http.build()
    }
}
```
