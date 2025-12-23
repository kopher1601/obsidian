`SecuirtyFilterChain` 을 여러개 분리해서 등록 가능하다
- `anyRequest` 는 가장 마지막 순서에 위치해야 한다
```kotlin  
@Configuration  
class WebAuthorizationConfig {  
  
    // 1. 특정 URL 패턴에만 적용되는 필터 체인 (Order가 낮아 먼저 체크됨)  
    //    '/api/**' 요청에만 Basic Auth를 적용하고 싶다고 가정  
    @Order(1)  
    @Bean  
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {  
        // !!! 필터 체인이 적용될 명시적인 URL 패턴을 지정 !!!        http.securityMatcher("/api/**") // <-- 이 부분이 핵심  
  
        http.httpBasic(Customizer.withDefaults()) // API 요청에 Basic Auth 적용  
  
        http.authorizeHttpRequests { auth ->  
            auth.anyRequest().authenticated()  
        }  
        return http.build()  
    }  
  
    // 2. 나머지 모든 요청을 처리하는 필터 체인 (Order가 높아 마지막에 체크됨)  
    //    anyRequest()를 포함하는 필터 체인은 Order를 가장 높게 설정하거나,  
    //    아예 Order를 생략하여 기본값(가장 낮음)으로 설정해 다른 필터 체인이 
	//    먼저 체크되게 해야 합니다.  
    //    여기서는 securityFilterChain2가 기본 체인 역할을 하도록 @Order(2)를 제거하거나 
    //    더 큰 값을 부여합니다.  
    @Bean // @Order를 제거하고 기본값으로 두거나, 매우 큰 값을 줍니다.  
    fun securityFilterChain2(http: HttpSecurity): SecurityFilterChain {  
        // securityMatcher를 사용하지 않으면 'any request'로 간주됩니다.  
        // 모든 요청에 대해 /login만 허용하고 나머지는 인증 요구  
        http.authorizeHttpRequests { auth ->  
            auth.requestMatchers("/login").permitAll()  
                .anyRequest().authenticated() // <-- 모든 요청에 대한 마지막 처리  
        }  
        return http.build()  
    }  
}
```


