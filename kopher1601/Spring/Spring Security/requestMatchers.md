```kotlin
@Bean  
fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {  
    http.httpBasic(Customizer.withDefaults())  
    http.authorizeHttpRequests {  
        it.requestMatchers("/a/b/**").authenticated()  
	        .requestMatchers("/product/{code:^[0-9]*$}").permitAll()
            .anyRequest().permitAll()  
    }  
  
    http.csrf { it.disable() }  
    return http.build()  
}
```
**`**` 연산자는 수에 제한 없이 경로 이름을 나타낸다.**
- `/a/**/b` -> `/a/c/b`, `/a/c/d/e/f/b` .. 일치함
- `/a/b/**` -> `/a/b/c/d/e/f/g` .. 일치함

**이름을 하나만 일칳파게 하려면 `*` 문자를 하나만 지정한다.**
- `/a/*/b` -> `/a/c/b` 는 일치하지만`/a/c/d/e/f/b` 는 일치하지 않음
