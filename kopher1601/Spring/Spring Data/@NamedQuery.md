> `@NamedQuery` 는 잘 안쓰일뿐더러 안쓰는게 좋다.

NamedQuery 는 비즈니스 엔티티와 연계돼 있는 미리 정의된 쿼리를 말한다. NamedQuery 는 [[JPQL]] 를 사용해서 쿼리를 정의한다. NamedQuery 는 엔티티 클래스 또는 엔티티 클래스의 슈퍼 클래스에 정의할 수 있다.

NamedQuery 는 `@NamedQuery` 애노테이션을 엔티티 클래스에 붙여서 정의할 수 있다.
```java
@Entity
@NamedQuery(
	name = "Course.findAllByCategoryAndRating",
	query = "select c from Course c where c.category = ?1 and c.rating = ?2"
)
public class Course { ... }
```
`?1`, `?2` 는 리포지토리 메서드가 호출될 때 메서드에 전달된 첫 번째 인자값과 두 번째 인자값으로 대체된다.
`@NamedQuery` 는 하나만 붙일 수 있는게 아니라 필요한 리포지토리 메서드의 수만큼 여러 번 붙일 수 있다.

