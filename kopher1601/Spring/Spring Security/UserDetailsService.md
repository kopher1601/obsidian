**기본** `UserDetailsService` 를 이용하면 [[PasswordEncoder]] 도 자동 구성되지만, UserDetailsService 를 재정의 하면 PasswordEncoder 도 다시 선언해야 한다.

```kotlin
public interface UserDetailsService {
	UserDetails loadUserByUsername(String username) 
		throws UsernameNotFoundException;
}
```
인증 구현은 `loadUserByUsername` 메서드를 호출해 주어진 사용자 이름을 가진 사용자의 세부 정보를 얻는다. 물론 여기에서 **사용자 이름은 고유하다고 간주**된다. 

