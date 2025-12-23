스프링 프레임워크에서 스프링이 관리하는 컴포넌트의 생성을 제어할 수 있도록 [[@Bean]], [[@Component]], [[@Configuration]] 애너테이션과 함께 사용할 수 있는 `@Conditional` 애너테이션을 제공한다.
```java
@Configuration
public class CommonApplicationContextConfiguration {

	@Bean
	@Conditional(RelationDatabaseCondition.class)
	public RelationDataSourceConfiguration dataSourceConfiguration() {
		return new RelationDataSourceConfiguration();
	}

}
```
`Condition` **구현체**
```java
public class RelationDatabaseCondition implements Condition {
	@Override
	public boolean matches() {}
}
```


