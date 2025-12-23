[[SecurityContext]] 를 관리하는 첫 번째 전략은 `MODE_THREADLOCAL` 전략이며 스프링 시큐리티가 `SecurityContext` 를 관리하는데 이용하는 **기본 전략**이기도 하다.

스프링 시큐리티는 [[ThreadLocal]] 을 이용해 컨텍스트를 관리한다. `ThreadLocal` 은 JDK 에 있는 구현이다. 이 구현은 데이터의 컬렉션처럼 작동하지만 애플리케이션의 각 스레드가 컬렉션에 저장된 데이터만 볼 수 있도록 보장한다. 이 방식으로 각 요청은 자신의 `SecurityContext` 에 접근하며 스레드는 다른 스레드의 `ThreadLocal` 에 접근할 수 없다. 웹 애플리케이션에서 이는 각 요청이 자신의 `SecurityContext` 만 볼 수 있다는 의미이고 백엔드 웹 애플리케이션의 **일반적인 작동 방식**이다.
![[Pasted image 20251029205119.png]]
각 요청은 자신의 `SecurityContext` 에 저장된 세부 정보만 볼 수 있다. 하지만 이는 새 스레드가 생성되면(예: 비동기 메서드가 호출됨) 새 스레드도 자체 `SecurityContext` 를 가지며 상위 스레드(요청의 원래 스레드)의 세부 정보가 새 스레드의 `SecurityContext` 로 복사되지 않는다는 의미다.


## Strategy
- `MODE_THREADLOCAL` : 각 스레드가 보안 컨텍스트에 각자의 세부 정보를 저장할 수 있게 해준다. 요청당 스레드 방식의 웹 애플리케이션에서는 각 요청이 개별 스레드를 가지므로 이는 일반적인 접근법이다
- `MODE_INHERITABLETHREADLOCAL` : `MODE_THREADLOCAL` 과 비슷하지만 비동기 메서드의 경우 `SecurityContext` 를 다음 스레드에 복사하도록 스프링 시큐리티에 지시한다. 이 방식으로 [[@Async]] 메서드를 실행하는 새 스레드가 `SecurityContext` 를 상속하게 할 수 있다. `@Async` 가 아니고 코드로 직접 스레드를 만들면 이 전략을 활성화해도 같은 문제가 발생한다
- `MODE_GLOBAL` : 애플리케이션의 모든 스레드가 같은 `SecurityContext` 를 보게 한다

**Kotlin/Spring Boot에서 전략 변경**
```kotlin
@EnableAsync // @Async 사용시 설정 필요
@Configuration
class ProjectConfig {
    @Bean
    fun initializingBean(): InitializingBean {
        return InitializingBean {
            SecurityContextHolder
	            .setStrategyName(
		            SecurityContextHolder.MODE_INHERITABLETHREADLOCAL)
        }
    }
}
```

### 자체 관리 스레드
`@Async` 같이 프레임워크가 생성하는 스레드가 아니라 개발자가 직접 생성한 스레드. 즉, 프레임워크가 모르는 방법으로 코드가 새 스레드를 시작하면 여전히 빈틈이 생긴다. 이러한 스레드는 프레임워크가 관리해주지 않아 개발자가 관리해야 하므로 종종 자체 관리 스레드라고 한다. 
- `DelegatingSecurityContextCallable` 를 사용 : タスクを Decorate する方法
```kotlin
@GetMapping("/ciao")  
fun ciao(): String {  
    val task = Callable {  
        val context = SecurityContextHolder.getContext()  
        context.authentication.name  
    }  
  
    val e = Executors.newCachedThreadPool()  
    try {  
	    // タスクを Decorate する方法での解決
        val contextTask = DelegatingSecurityContextCallable(task)  
        return "Ciao, " + e.submit(contextTask).get() + "!"  
    } finally {  
        e.shutdown()  
    }  
}
```
- `DelegatingSecurityContextExecutorService` : 스레드 풀에서 관리하는 방법
```kotlin
@GetMapping("/hola")
    fun hola(): String {
        val task = Callable {
            val context = SecurityContextHolder.getContext()
            context.authentication.name
        }

        var e = Executors.newCachedThreadPool()
        // ThreadPool 을 Decorate 해서 관리하는 방법
        e = DelegatingSecurityContextExecutorService(e)
        try {
            return "Hola, " + e.submit(task).get() + "!"
        } finally {
            e.shutdown()
        }
    }
}
```
