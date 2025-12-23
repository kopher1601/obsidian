스프링 부트는 스프링 프레임워크를 이용한 애플리케이션 개발에서 혁신적인 단계로 평가받는다. 스프링 부트는 미리 준비도니 구성을 제공하므로 모든 구성을 작성하는 대신 자신의 구현과 일치하지 않는 구성만 재정의하면 된다. 이 접근법을 **설정보다 관습(convention-over-configuration)** 이라고 한다.

스프링 부트는 개발자가 기술적인 보일러플레이트 코드나 복잡한 설정이 아니라 비즈니스 로직에 더 집중할 수 있게 해줬다. 스프링 부트는 스프링 프레임워크에서 사용되는 미리 정의된 형식을 따름으로써 애플리케이션 개발자가 빠르게 개발 과정을 시작할 수 있게 해준다. 스프링 부트는 스프링 프레임워크와 개발자 사이에 존재하는 계층으로서 설정을 단순화해주는 역할을 한다고도 볼 수 있다.

# 스프링 부트 핵심 기능
- **자동 구성(autoconfiguration)**
- **미리 정의된 방식(opinionated) :** 스프링 부트는 미리 정의된 방식을 따른다. 그래서 스프링 애플리케이션을 실행할 때 필요한 몇 가지 컴포넌트를 스타터 의존관계를 기준으로 자동으로 구성한다.
- **독립 실행형(standalone) :** 스프링 부트 애플리케이션은 웹 서버를 내장하고 있어서 외부 웹 서버나 애플리케이션 서버 없이도 독립적으로 설치되어 실행할 수 있다. 스프링 부트 애플리케이션은 실행 가능한 `JAR` 파일로 패키징 되어 `java -jar` 명령으로 간단하게 실행할 수 있다.
- **실제 서비스 환경에서 사용가능(product-ready)**

# 구성 클래스
구성 클래스도 책임을 분리하는 것이 좋다. 이러한 분리가 필요한 이유는 구성이 복잡해지기 때문이다. 또한 항상 한 클래스가 하나의 책임을 맡도록 하는 것이 바람직하다.

# application.properties
스프링 부트는 애플리케이션 환경 설정 정보를 `aaplication.properties` 또는 `application.yml` 파일에서 지정할 수 있다.
- 스프링 부트에서는 [[Profile]] 별로 프로퍼티 파일을 다르게 지정해서 사용할 수 있다. 
```properties
spring.datasource.url=jdbc:h2:mem:ssia  
spring.datasource.username=sa  
spring.datasource.password=  

# data.sql, schema.sql 실행시키기
spring.sql.init.mode=always

# data.sql 과 jpa 를 함께 사용할 때 밑의 설정을 해줘야지 데이터가 들어간다
spring.jpa.defer-datasource-initialization=true  
spring.jpa.hibernate.ddl-auto=none

# sql
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# profile 설정
spring.profiles.active=dev
```
`application.properties` 파일에서는 설정 정보를 포함하고 있는 다른 properties 파일이나 `.yml` 파일을 `spring.config.import` 프로퍼티를 통해 임포트해서 사용할 수 있다.
- `additional-application.properties`  는 당연히 `/resources` 안에 있어야 한다.
- 클래스 패스에 파일이 존재하지 않으면 `ConfigDataLocationNotFoundException` 예외를 던진다. 파일이 없어도 예외를 내지 않고 실행시키고 싶은 경우 `spring.config.on-not-found=ignore` 를 지정해주면 된다
```properties
spring.config.import=classpath:additional-application.properties
```
프러퍼티 파일의 위치를 [[@PropertySource]] 애너테이션을 사용해서 지정할 수도 있다.

`.properties` 나 `.yml` 파일에 명시된 설정 프로퍼티 정보는 [[Environment]] 객체에 로딩되고, 애플리케이션 클래스에서 `Environment` 인스턴스에 접근해서 설정정보를 읽을 수 있으며, [[@Value]] 애너테이션을 통해 접근할 수도 있다.

- `--spring.config.name` 로 `/resources` 밑의 properties 파일을 지정할 수 있다
- `--spring.config.location` 으로 다른 위치에있는 설정파일을 읽을 수 있다. `--spring.config.location=optional:data/sbip.yml` optional 지정도 가능
```shell
java -jar build/libs/spring-boot-in-practice-0.0.1-SNAPSHOT.jar --spring.config.name=sbip
```

# 커스텀 프로퍼티
[[@ConfigurationProperties]] 를 이용해 커스텀 프로퍼티를 만들 수 있다.
# 스프링 부트 애플리케이션 시작 시 코드 실행
[[CommandLineRunner]] 랑 `ApplicationRunner` 둘 다 `run()` 메서드 하나만 가지고 있는 인터페이스인데 이 인터페이스를 구현해서 빈으로 등록해두면 스프링 부트 애플리케이션 초기화 완료 직전에 `run()` 메서드가 실행된다.

# 자동 구성
`spring-boot-autoconfigure` 모듈에 스프링 부트의 자동 구성 마법의 핵심이 담겨 있고, 모든 스프링 부트 프로젝트가 `spring-boot-autoconfigure` 의존 관계를 포함하고 있다.
- `META-INF/spring.factories` 파일을 확인해보자
`spring.factories` 파일을 살펴보면 `autoconfigure` 라고 표시된 부분을 찾을 수 있는데, 여기에 여러 스프링 부트 컴포넌트와 서드파티 라이브러리에 대한 자동 구성 정보가 포함돼 있다.


