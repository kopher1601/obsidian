`@Configuration` 애노테이션을 사용해 프로퍼티 정보를 담는 클래스를 만들어서 타입 안정성을 보장하고 유효성을 검증한다.. 이렇게 하면 [[@Value]] 애너테이션이나 [[Environment]] 객체를 사용하지 않고도 프로퍼티값을 읽어서 사용할 수 있다.

 `@ConfigurationProperties` 애너테이션이 붙어 있는 클래스에 대한 메타데이터를 생성하려면 스프링 부트 설정 처리기가 필요하다. 이 메타데이터는 IDE 가 `application.properties` 파일에 기술된 프로퍼티에 대한 자동 완성이나 문서화를 지원하는 데 사용 된다.

스프링 부트 애플리케이션에서 `@ConfigurationProperties` 애너테이션을 사용하면 커스텀 프로퍼티의 타입 안정성을 쉽게 보장받을 수 있다. `spring.config.import` 와 `@ConfigurationProperties` 애너테이션을 함께 사용하면 연관된 프로퍼티끼리 그룹지어 별도의 프로퍼티 파일로 관리할 수 있다.

 **의존성 추가**
```gradle
implementation("org.springframework.boot:spring-boot-configuration-processor:3.5.7")
```
**사용**
```kotlin
package jp.kopher.springbootinpractice.properties  
  
  
import org.springframework.boot.context.properties.ConfigurationProperties  
  
@ConfigurationProperties("app.sbip.ct")  
data class AppProperties(  
    /**  
     * Application Name     */    val name: String,  
    /**  
     * Application IP     */    val ip: String,  
    /**  
     * Application IP     */    val port: Int,  
    /**  
     * Application Security configuration     */    val security: Security  
) {  
    data class Security(  
        /**  
         * Enable Security. Possible values true/false         */        val isEnabled: Boolean,  
        /**  
         * Token Value         */        val token: String,  
        /**  
         * Available roles         */        val roles: List<String>  
    )  
}
```
[[@EnableConfigurationProperties]] 를 추가해주어야 한다. 또는 [[@ConfigurationPropertiesScan]] 을 사용
```kotlin
@SpringBootApplication  
@EnableConfigurationProperties(AppProperties::class)  
class SpringBootInPracticeApplication
```
