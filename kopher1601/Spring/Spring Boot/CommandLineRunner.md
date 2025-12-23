여러개의 `CommandLineRunner` 를 등록할 수 있고 [[@Order]] 애노테이션을 사용해서 등록한 `CommandLineRunner` 의 실행순서를 정해줄 수 있다 `@Order`에 지정된 숫자가 낮을 수록 우선순위가 높다.

`CommandLineRunner` 안에서는 스프링의 의존관계주입으로 빈을 주입받아 사용할 수도 있다. `CommandLineRunner` 구현체는 스프링 부트 애플리케이션이 빈 등록을 포함한 초기화 과정 수행을 거의 다 마친 뒤에 실행되므로 어떤 빈이든 주입받아 사용할 수 있다.
```kotlin
@Bean  
fun commandLineRunner(repository: Repository): CommandLineRunner {  
    return CommandLineRunner {  
        log.info("CommandLineRunner")  
        repository.findAll()
    }  
}
```
## SpringApplication 에서 직접 구현
```kotlin
@SpringBootApplication  
class SpringBootInPracticeApplication: CommandLineRunner {  
    private val log = org.slf4j.LoggerFactory.getLogger(SpringBootInPracticeApplication::class.java)  
  
    override fun run(vararg args: String?) {  
        log.info("CommandLineRunner run")  
    }  
}  
  
fun main(args: Array<String>) {  
    runApplication<SpringBootInPracticeApplication>(*args)  
}
```
## @Bean 을 사용한 방법
```kotlin
@SpringBootApplication  
class SpringBootInPracticeApplication {  
    private val log = org.slf4j.LoggerFactory.getLogger(SpringBootInPracticeApplication::class.java)  
  
    @Bean  
    fun commandLineRunner(): CommandLineRunner {  
        return CommandLineRunner {  
            log.info("CommandLineRunner")  
        }  
    }  
}  
  
fun main(args: Array<String>) {  
    runApplication<SpringBootInPracticeApplication>(*args)  
}
```
