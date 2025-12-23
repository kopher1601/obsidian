중복 사용 가능
```kotlin
package jp.kopher.springbootinpractice  
  
import org.springframework.context.annotation.Configuration  
import org.springframework.context.annotation.PropertySource  
import org.springframework.core.env.Environment  
  
@Configuration  
@PropertySource("classpath:dbConfig.properties")  
class DbConfiguration(  
    private val env: Environment,  
) {  
    override fun toString(): String {  
        return "User: ${env.getProperty("db.user")}, Password: ${env.getProperty("db.password")}"  
    }  
}
```
스프링이 제공하는 [[Environment]] 인스턴스를 주입 받으면 `dbConfig.properties` 에 지정된 설정 정보를 읽을 수 있다.