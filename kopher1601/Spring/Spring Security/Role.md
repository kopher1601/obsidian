역할은 결이 굵아(`coarse-grained`). 특정 역할이 있는 사용자는 해당 역할에 허가된 작업만 할 수 있다.
[[Spring Security]] 에서 역할은 사용자가 수행할 수 있는 작업을 나타내는 다른 방법이다.
![[IMG_0857.jpg]]
역할은 사용자가 착용하는 배지와 비슷하며 사용자에게 작업 그룹에 속한 이용 권리를 제공한다.

애플리케이션에서 역할을 이용하면 더는 [[Authorization]] (권한)을 정의할 필요가 없다. 이때 권한은 개념상으로 존재하고 구현 요구사항에도 나올 수 있지만 애플리케이션에서는 사용자가 이용 권리를 가진 하나 이상의 작업을 포함하는 역할만 정의하면 된다.

한편, 내부적으로 역할도 스프링 시큐리티에서 같은 [[GrantedAuthority]] 계약으로 나타낸다. 역할을 정의할 때 역할 이름은 `ROLE_` 접두사로 시작해야 한다. 구현 수준에서 이 접두사는 역할과 권한 간의 차이를 나타낸다.

**역할의 정의** 
```kotlin
@Bean  
fun userDetailsService(): UserDetailsService {  
    val manager = InMemoryUserDetailsManager()  
  
    val user1 = User.withUsername("john")  
        .password("12345")  
        // .authorities("ROLE_ADMIN") // 역할을 정의할 떄는 ROLE_ 접두사를 붙여줘야 한다
        .roles("ADMIN") // ROLE_ 접두사를 자동으로 붙여준다.
        .build()  
    val user2 = User.withUsername("jane")  
        .password("12345")  
        .authorities("ROLE_MANAGER") // 역할의 정의
        .build()  
  
    manager.createUser(user1)  
    manager.createUser(user2)  
  
    return manager  
}
```
**역할의 사용**
```kotlin
http.authorizeHttpRequests{  
    it.anyRequest().hasRole("ADMIN") // 역할 사용시에는 접두사를 붙이지 않음
}
```
