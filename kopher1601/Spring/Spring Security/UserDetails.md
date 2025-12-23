
두 책임을 가진 코드는 정말 좋지 않다. 
```java
// 두 책임을 가진 User 클래스
@Entity
public class User implements UserDetails { ... }
```
책임을 분리한 코드
```java
@Entity
public class User {}

public class SecurityUser implements UserDetails {
	private final User user; // User 가 반드시 필요하다는것을 알리기위한 final
	
	public SecurityUser(User user) {
		this.user = user;
	}
	
	...
}
```
