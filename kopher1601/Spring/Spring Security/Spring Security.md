인증과 접근 제어를 위해 세부적인 맞춤 구성이 가능한 강력한 프레임워크이다. 또한 스프링 애플리케이션에 보안을 적용하는 과정을 크게 간소화하는 프레임워크이다.

스프링 시큐리티는 스프링 애플리케이션에서 **애플리케이션 수준의 보안**을 구현할 때 가장 우선적인 선택이며 인증, 권항 부여 및 일반적인 공격에 대한 방어를 구현하는 세부적인 맞춤 구성 방법을 제공한다.

스프링 시큐리티 아키텍처의 필터는 일반적인 HTTP [[Filter]]다. 
- Spring Security 에서 [[CustomFilter]] 를 구현해서 추가하는 방법

> 같은 위치에 필터를 두 개 이상 추가할 수 있다. 하지만 이 경우 스프링 시큐리티는 이들 필터가 호출되는 순서를 보장하지 않는다.

스프링 시큐리티는 기본적으로 [[CSRF]] 보호 기능을 활성화 시킨다.

WebSecurityConfigurerAdapter 는 스프링 시큐리티 6부터 폐지되었고 대신에 [[SecuirtyFilterChain]] 을 사용해야 한다.
- [Spring Security without the WebSecurityConfigurerAdapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)

# Spring Security Flow Diagram
이 다이어그램이 스프링 시큐리티의 전부라고 할 수 있다.
![[Pasted image 20251026142226.png]]

## 스프링 시큐리티에서의 CSRF
[[CSRF]] 설정을 수동으로 `disable` 하지 않는 이상 스프링 시큐리티는 `POST`, `DELETE`, `PUT` 등 데이터를 변경하는 요청에 대해 CSRF 공격 방지 목적으로 403 에러를 반환한다.
- CSRF 보호의 시작점은 필터 체인의 [[CsrfFilter]] 라는 한 필터다

애플리케이션이 토큰을 관리하는 방법을 변경하려면 [[CsrfTokenRepository]] 인터페이스를 구현해 맞춤형 구현을 프레임워크에 연결해야 한다.
![[Pasted image 20251107210119.png]]
### 특정 경로 CSRF 보호 비활성화
```kotlin
http.csrf { 
	it.ignoringRequestMatchers("/ciao") 
}
```

## 스프링 시큐리티에서의 CORS
- [[@CrossOrigin]] 애노테이션으로 CORS 를 구성할 수 있다
- `http.cors` 로 CORS 를 구성할 수 있다

## 스프링 시큐리티에서의 OAuth2
- oauth2() 메서드를 호출할 때 스프링 시큐리티는 [[OAuth2LoginAuthenticationFilter]] 를 추가한다. 이 필터는 요청을 가로채고 OAuth2 인증에 필요한 논리를 적용한다.
- [[ClientRegistration]] 인터페이스는 OAuth2 아키텍처의 `Client` 를 나타낸다.

## 전역 메서드 보안
전역 메서드 보안은 엔드포인트 없이도 권한 부여 규칙을 구현할 수 있게 해준다.  **전역 메서드 보안은 기본적으로 비활성화 상태**이므로 이 기능을 이용하려면 먼저 **활성화**해야 한다.
- 전역 메서드 보안을 활성화하기 위해서는 구성 클래스에 `@EnableMethodSecurity` 애노테이션을 추가하면 된다.
```kotlin
@Configuration  
@EnableMethodSecurity(prePostEnabled = true)  
class ProjectConfig() {}
```
- 메서드 수준에 권한 부여 규칙을 적용하면 규칙이 적용되는 동작을 더 세부적으로 지정할 수 있다.
- 전역 메서드 보안을 활성화하면 스프링 애스펙트가 하나 활성화 된다. 이 애스펙트는 우리가 권한 부여 규칙을 적용하는 메서드에 대한 호출을 가로채고 권한 부여 규칙을 바탕으로 가로챈 메서드로 호출을 전달할지 결정한다.
![[IMG_0946.jpg]]

전역 메서드 보안으로 할 수 있는 일은 크게 두 가지이다.
- **호출 권한 부여** : 누군가가 메서드를 호출할 수 있는지(사전 권한 부여, `PreAuthorization`) 또는 메서드가 실행된 후 메서드가 반환하는 것에 액세스 할 수 있는지(사후 권한 부여, `PostAuthorization`) 결정한다.
- **필터링** : 메서드가 매개 변수를 통해 받을 수 있는 것(사전 필터링)과 메서드가 실행된 후 호출자가 메서드에서 다시 받을수 있는 것 (사후 필터링) 을 결정한다.

### 호출 권한 부여
#### 호출 권한 부여의 분류
- **사전 권한 부여 (`PreAuthorization`)** : 메서드 호출 전에 권한 부여 규칙을 검사
- **사후 권한 부여 (`PostAuthorization`)** : 메서드 호출 후에 권한 부여 규칙을 검사

#### **사전 권한 부여 (PreAuthorization)** 
누군가가 특정 상황에서 메서드를 호출하는 것을 완전히 금지하는 권한 부여 규칙을 적용할 때 이를 사전 권한 부여(`PreAuthorization`)이라고 한다. 이 방식은 메서드를 실행하기 전에 프레임워크가 권한 부여 조건을 확인한다고 가정한다. 호출자가 우리가 정의하는 권한 부여 규칙에 따른 사용 권한이 없으면 프레임 워크는 메서드에 대한 호출을 위임하지 않고 대신 오류를 투적한다.
![[IMG_0947.jpg]]
```kotlin
@Service  
class NameService {  
  
    @PreAuthorize("hasAuthority('write')")  
    fun getName(): String {  
        return "Fantastico"  
    }
    
	@PreAuthorize("#name == authentication.principal.username")  
	fun getSecretNames(name: String): List<String> {  
	    return secretNames[name] ?: emptyList()  
	}
	
	// PermissionEvaluator 이용
    @PreAuthorize("hasPermission(#code, 'document', 'ROLE_admin')")
    public Document getDocument(String code) {
        return documentRepository.findDocument(code);
    }  
      
}
```
- 메서드의 매개변수 값을 `#매개변수이름` 으로 참조하고 인증 개체(`authentication`)에 직접 접근해서 현재 인증된 사용자를 참조한다.
#### 사후 권한 부여 (PostAuthorization)
메서드를 호출하도록 허용하지만 메서드가 반환한 결과를 얻기 위해 권한 부여가 필요한 방식을 사후 권한 부여라고 한다. 사후 권한 부여에서 스프링 시큐리티는 메서드 호출 후에 권한 부여 규칙을 검사한다. 사후 권한 부여는 메서드 실행 후에 수행되기 때문에 메서드에서 반환된 결과를 바탕으로 권한 부여 규칙을 적용할 수 있다.
![[IMG_0948.jpg]]

> **사후 권한 부여를 이용할 때는 주의가 필요**
> 메서드가 실행중에 무엇인가를 변경하면 권한 부여 성공 여부와 관계없이 이러한 변경은 남는다.
> [[@Transactional]] 애노테이션 이용과 무관하게 사후 권한 부여가 실패해도 변경이 롤백되지는 않는다. 사후 권한 부여 기능에서 발생하는 예외는 트랜잭션 관리자가 트랜잭션을 **커밋**한 후에 발생한다.

```kotlin
@Service  
class BookService {  
  
    private val records = mapOf(...)
    
    @PostAuthorize("returnObject.roles.contains('reader')")  
    fun getBookDetails(name: String): Employee {  
        return records[name] 
	        ?: throw IllegalArgumentException("Invalid user name")  
    }
    
    // PermissionEvaluator 이용
	@PostAuthorize("hasPermission(returnObject, 'ROLE_admin')")  
	fun getDocument(code: String): Document {  
	    return documentRepository.findDocument(code)  
	        ?: throw IllegalArgumentException("Document not found")  
	}  
}
```
- `returnObject` 를 통해 반환값에 접근할 수 있다.

## 사용 권한(Permission)
긴 SpEL 식은 권장되지 않으며 코드를 읽기 어렵게 만들고 앱의 유지 보수 효율을 떨어뜨린다. 복잡한 권한 부여 규칙을 구현해야 할 때는 긴 SpEL 식을 작성하지 말고 논리를 별도의 클래스로 만들어야 한다. 스프링 시큐리티의 **사용 권한(permission)** 개념으로 권한 부여 규칙을 별도의 클래스로 작성하면 애플리케이션을 읽고 이해하기 쉽게 만들 수 있다.

사용 권한 논리를 구현할 책임은 개발자에게 있다. 이를 위해 [[PermissionEvaluator]] 계약을 구현하는 객체를 작성해야 한다. 

### 필터링
**메서드 호출은 허용**하면서도 메서드로 보내는 매개 변수가 몇 가지 규칙을 따르는지 확인하고 싶을 수 있다. 또는 메서드를 호출한 후 호출자가 반환된 값의 승인된 부분만 받을 수 있게 하려는 시나리오 등에서 쓰인다. 이런 기능을 필터링이라고 하며 다음의 두 범주로 분류한다.
- 사전 필터링(prefiltering) : 프레임워크가 메서드를 호출하기 전에 매개 변수의 값을 필터링한다.
- 사후 필터링(postfiltering) : 프레임워크가 메서드를 호출한 후 반환된 값을 필터링한다.
![[IMG_0949.jpg]]
필터링은 호출권한 부여와는 다른 방식으로 작동한다. 필터링을 적용하면 프레임워크는 매개 변수나 반환된 값이 권한 부여 규칙을 준수하지 않아도 **메서드를 호출**하며 **예외를 투척하지 않는다.** 대신 지정한 조건을 준수하지 않는 요소를 필터링한다. 또한 **필터링은 컬렉션과 배열에만 적용할 수 있다.**

> 스프링 시큐리티는 필터링 또한 권한 부여와 마찬가지로 AOP 로 구현한다.
#### 사전 필터링(Prefiltering)
필터링을 이용하면 누군가가 메서드를 호출할 때 메서드 매개 변수로 전송된 값을 검증하도록 프레임워크에 지시할 수 있다. 프레임워크는 주어진 기준을 충족하지 않는 값을 필터링하고 기준을 충족하는 값으로만 메서드를 호출한다. 이 기능을 **사전 필터링**이라고 한다.

사전 필터링을 이용하면 메서드가 구현하는 비즈니스 논리와 권한 부여 규칙을 분리할 수 있어 많은 실제 사례의 요구 사항을 해결하는 데 적합하다. 권한 부여 논리와 비즈니스 논리를 분리하면 코드를 유지 관리하기 편해지고 다른 사람이 코드를 읽고 이해하기도 쉬워진다.

```kotlin
@Service  
class ProductService() {  
  
    @PreFilter("filterObject.owner == authentication.name")  
    fun sellProducts(products: List<Product>): List<Product> {  
        return products  
    }  
      
}
```
> `listOf()` では filtering が動作しない 
> 주어진 컬렉션을 애스펙트가 변경하기 때문이다. 애스펙트가 인스턴스에서 기준에 맞지 않는 요소를 제거한다. 이 점을 감안해서 제공하는 컬렉션이 변경 불가능하지 않은지 확인해야 한다. 변경이 불가능한 컬렉션을 제공하면 필터링 애스펙트가 컬렉션의 내용을 변경할 수 없어 실행 시 예외가 발생한다.
```kotlin
@GetMapping("/sell")  
fun sellProduct(): List<Product> {  
    return productService.sellProducts(  
        // listOf では filtering が動作しない  
        mutableListOf(  
            Product("beer", "nikolai"),  
            Product("candy", "nikolai"),  
            Product("chocolate", "julien"),  
        )  
    )  
}
```

#### 사후 필터링(Postfiltering)
사전 필터링과 대부분이 같고 `@PostFilter` 애노테이션을 이용한다.

#### 스프링 데이터 리포지토리에 필터링 이용
- `@PreFilter`, `@PostFilter` 애노테이션를 이용하는 방법
- 쿼리내에 직접 필터링 적용
리포지토리에 `@PreFilter` 애노테이션을 적용하는 사전 필터링은 **애플리케이션의 다른 계층에 적용하는것과 같지만** 사후 필터링은 상황이 다르다. **리포지토리 메서드에 `@PostFilter` 를 적용하는 것은 기술적으로는 문제가 없지만 성능 관점에서는 대부분 좋지 않은 선택**이다.

## 스프링 시큐리티에서의 테스트
일반적으로 권한 부여와 인증은 따로 테스트하는 것이 합리적이다. 앱이 사용자를 인증하는 방법은 일반적으로 한 가지지만, 권한 부여가 다르게 구성된 수많은 엔드포인트를 가진 경우가 많다. 이것이 인증은 소수의 테스트로 별도로 테스트한 후 엔드포인트의 각 권한 부여 구성에 대한 테스트는 개별적으로 만든느 이유다. 논리가 변하지 않기 때문에 각 엔드포인트를 테스트할 때마다 인증을 반복하는 것은 실행 시간의 낭비다.
### MockUser 로 테스트
권한 부여 구성을 테스트하는 가장 직관적이고 자주 이용되는 방식이다. 모의 사용자를 이용한 테스트에서 인증 프로세스는 완전히 건너뛴다.
- 모의 사용자는 권한 부여를 다룰 때만 이용할 수 있다.
![[IMG_0950.jpg]]
> 테스트를 실행할 때는 스프링 시큐리티의 인증 흐름에서 **음영으로 표시된 구성 요소를 건너 뛴다.**
[[@WithMockUser]] 애노테이션을 이용하면 된다. 테스트 메서드 위에 이 애노테이션을 지정하면 [[SecurityContext]] 를 구성하고 [[UserDetails]] 구현 인스턴스를 포함하도록 스프링에 지시할 수 있다. 이는 인증을 사실상 건너뛰는 것이다.
### UserDetailsService의 사용자로 테스트
이 접근방식은 앱이 사용자 세부 정보를 로드해야 하는 데이터 원본과의 통합도 테스트하려는 경우 적합하다.
![[IMG_0951.jpg]]
이 접근방식에서는 컨텍스트에 [[UserDetailsService]] 빈이 있어야 하는 점에 주의하자. 이 [[@WithUserDetails]] 애노테이션을 테스트 메서드에 붙여준다. 이 애노테이션으로 사용자를 찾으려면 사용자 이름을 지정해야 한다.
```java
@Test
@DisplayName("Test calling /hello endpoint authenticated returns ok.")
@WithUserDetails("john")
public void helloAuthenticated() throws Exception {
	mvc.perform(get("/hello"))
			.andExpect(status().isOk());
}
```

### 맞춤형 인증 Authentication 객체를 이용한 테스트
테스트를 위한 [[Authentication]] 객체를 특정한 형식으로 만들어달라고 프레임워크에 요청할 수 있다.
![[IMG_0952.jpg]]
**이 동작을 달성하기 위한 세 단계**
1. `@WithMockUser` 또는 `@WithUserDetails` 를 이용하는 방식과 비슷하게 테스트에 이용할 애노테이션을 작성
2. `WithSecurityContextFactory` 인터페이스를 구현하는 클래스를 작성. 이 클래스는 프레임워크가 테스트에 이용하는 모의 [[SecurityContext]] 를 반환하는 `createSecurityContext()` 를 반환한다
3. 1 단계에서 만든 맞춤형 애노테이션을 2단계에서 팩터리 클래스에 [[@WithSecurityContext]] 애노테이션에 연결한다
#### 애노테이션 작성
```kotlin
@Retention(AnnotationRetention.RUNTIME)  
annotation class WithCustomUser(  
    val username: String  
)
```
#### WithSecurityContextFactory 구현
`test` 패키지 안에서만 `WithSecurityContextFactory` 를 import 가능
```kotlin
class CustomSecurityContextFactory: WithSecurityContextFactory<WithCustomUser> {  
    override fun createSecurityContext(annotation: WithCustomUser?): SecurityContext {  
        val context = SecurityContextHolder.createEmptyContext()  
          
        UsernamePasswordAuthenticationToken(annotation?.username, null, null).also {  
            context.authentication = it  
        }        return context  
    }  
}
```
#### 애노테이션을 팩토리와 연결
```kotlin
@Retention(AnnotationRetention.RUNTIME)  
@WithSecurityContext(factory = CustomSecurityContextFactory::class)  
annotation class WithCustomUser(  
    val username: String  
)
```
**애노테이션의 사용**
```kotlin
@Test
@WithCustomUser("mary")
fun test() {}
```