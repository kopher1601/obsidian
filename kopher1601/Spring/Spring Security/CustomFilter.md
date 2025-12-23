필터를 만들려면 `jakarta.servlet` 패키지의 [[Filter]] 인터페이스를 구현한다. 새 필터는 필터 체인의 다른 필터를 기준으로 추가된다 . 즉, 기존 필터 위치(`addFilterAt`) 또는 앞(`addFilterBefore`)이나 뒤(`addFilterAfter`)에 새 필터를 추가할 수 있다.

특정 위치에 필터를 추가해도 **스프링 시큐리티는 이 위치에 필터가 하나라고 가정하지 않는다**. 필터 체인의 같은 위치에 필터를 더 추가할 수 있으며 이 경우 **스프링 시큐리티는 필터가 실행되는 순서를 보장하지 않는다**. 기존 필터의 위치에 다른 필터를 적용하면 필터가 대체된다고 생각하는 경우가 많은데 그렇지 않다. 필터 체인에 필요 없는 필터는 아예 추가하지 않아야 한다.

> **순서를 보장하지 않는다**는거지 실행되지 않는다는 의미가 아니다

**필터의 구현**
```kotlin
package jp.kopher.springsecurityinaction.filters

import jakarta.servlet.Filter
import jakarta.servlet.FilterChain
import jakarta.servlet.ServletRequest
import jakarta.servlet.ServletResponse
import jakarta.servlet.http.HttpServletRequest
import jakarta.servlet.http.HttpServletResponse
import org.springframework.beans.factory.annotation.Value
import org.springframework.stereotype.Component

@Component
class StaticKeyAuthenticationFilter(
    @Value("\${authorization.key}")
    private var authorizationKey: String,
) : Filter {
    override fun doFilter(request: ServletRequest?, response: ServletResponse?, chain: FilterChain?) {
        val httpServletRequest = request as HttpServletRequest
        val httpServletResponse = response as HttpServletResponse

        httpServletRequest.getHeader("Authorization")?.let {
            if (authorizationKey == it) {
                chain?.doFilter(request, response)
                return
            }
        }
        httpServletResponse.status = HttpServletResponse.SC_UNAUTHORIZED
    }
}
```

**필터 설정**
```kotlin
package jp.kopher.springsecurityinaction.config

import jp.kopher.springsecurityinaction.filters.StaticKeyAuthenticationFilter
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.web.SecurityFilterChain
import org.springframework.security.web.authentication.www.BasicAuthenticationFilter

@Configuration
class ProjectConfig(
    private val staticKeyAuthenticationFilter: StaticKeyAuthenticationFilter,
) {

    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http
	    .addFilterAt(
		    staticKeyAuthenticationFilter, 
		    BasicAuthenticationFilter::class.java)
            .authorizeHttpRequests {
                it.anyRequest().permitAll()
            }
        return http.build()
    }
}
```
- `addFilterAt`, `addFilterAfter`, `addFilterBefore` 등이 있다
