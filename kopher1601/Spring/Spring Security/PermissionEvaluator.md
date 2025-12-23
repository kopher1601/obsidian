```java
boolean hasPermission(
	Authentication authentication, Object targetDomainObject, Object permission);

boolean hasPermission(
	Authentication authentication, 
	Serializable targetId, 
	String targetType, 
	Object permission);
```
`PermissionEvaluator` 계약으로 사용 권한 논리를 구현하는 방법은 두 가지다.
- 객체, 사용권한(`Object target, Object permission`) : 사용 권한 평가기는 두 객체(권한 부여 규칙의 주체가 되는 객체와 사용 권한 논리를 구현하기 위한 추가 세부 정보를 제공하는 객체)를 받는다.
- 객체 ID, 객체 형식, 사용 권한(`Object targetId, String targetType, Object permission`) : 사용 권한 평가기는 필요한 객체를 얻는 데 이용할 수 있는 객체 ID 를 받는다. 또한 같은 권한 평가기가 여러 객체 형식에 적용될 때 이용할 수 있는 객체 형식을 받으며 사용 권한을 표가하기 위한 추가 세부 정보를 제공하는 객체가 필요하다.

혼동을 피하고 확실하게 이해하기 위해 [[Authentication]] 객체를 전달할 필요가 없다는 점도 알아 두자. 스프링 시큐리티는 `hasPermission()` 메서드를 호출할 때 자동으로 이 매개 변수 값을 제공한다. 이미 [[SecurityContext]] 에 있으므로 프레임워크는 인증 인스턴스의 값을 알 수 있다.

**구현**
```kotlin
@Component  
class DocumentPermissionEvaluator: PermissionEvaluator {  
    override fun hasPermission(authentication: Authentication?, target: Any?, permission: Any?): Boolean {  
        val document = target as Document  
        val p = permission as String  
  
        val admin = authentication?.authorities?.any { it.authority == p }  
  
        return admin == true || document.owner == authentication?.name  
    }  
  
    override fun hasPermission(  
        authentication: Authentication?,  
        targetId: Serializable?,  
        targetType: String?,  
        permission: Any?  
    ): Boolean {  
		String code = targetId.toString();
        Document document = documentRepository.findDocument(code);

        String p = (String) permission;

        boolean admin =
                authentication.getAuthorities()
                        .stream()
                        .anyMatch(a -> a.getAuthority().equals(p));

        return admin || document.getOwner().equals(authentication.getName());
    }  
}
```ㅈ
**등록**
```kotlin
@Configuration  
@EnableMethodSecurity(prePostEnabled = true)  
class ProjectConfig(  
    private val evaluator: DocumentPermissionEvaluator,  
) {  
  
    @Bean  
    fun methodSecurityExpressionHandler(): MethodSecurityExpressionHandler {  
        val handler = DefaultMethodSecurityExpressionHandler()  
        handler.setPermissionEvaluator(evaluator)  
        return handler  
    }
    
    ...
}
```