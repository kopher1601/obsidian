`OncePerRequestFilter` 는 이름이 의미하듯이 [[Filter]] 의 `doFilter()` 메서드가 요청당 한 번만 실행되도록 논리를 구현한다.

`OncePerRequestFilter` 는 기본적으로 **비동기 요청이나 오류 발송 요청에는 적용되지 않는다**. 이 동작을 변경하려면 `shouldNotFilterAsyncDispatch()` 및 `shouldNotFilterErrorDispatch()` 메서드를 override(재정의) 하면 된다.

**구현 예제**
```kotlin
class AuthenticationLoggingFilter: OncePerRequestFilter() {

    private val log = org.slf4j.LoggerFactory.getLogger(this::class.java)

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        request.getHeader("Request-Id").let {
            log.info("Request-Id: $it")
        }
        filterChain.doFilter(request, response)
    }
}
```
