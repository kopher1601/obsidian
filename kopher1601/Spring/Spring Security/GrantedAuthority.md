사용자에게 허가된 작업을 권한(authority)이라고 한다. 권한은 사용자가 애플리케이션에서 수행할 수 있는 작업을 나타낸다. 

애플리케이션은 사용자가 필요로 하는 권한인 애플리케이션의 기능 요구 사항에 따라 다른 유형의 사용자를 구분해야 한다. 스프링 시큐리티에서는 `GrantedAuthority` 인터페이스로 권한을 나타낸다.
![[Pasted image 20251101124254.png]]


- **hasAnyAuthority** : 여러개를 받을 수 있다
```kotlin
hasAnyAuthority("WRITE", "READ")
```
- **hasAuthority** : 단 하나만 받을 수 있다
```kotlin
hasAuthority("WRITE")
```
- **access** : SpringEL 식을 사용해서 더 광범위하게 설정가능하나. 디버깅이 어렵다
```kotlin
access("hasAuthority('WRITE') and !hasAuthority('DELETE')")
```


