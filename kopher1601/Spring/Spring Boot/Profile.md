스프링 프로파일은 애플리케이션 설정의 일부를 분리해서 환경별로 다르게 적용할 수 있게 해준다.
특정 프로파일에만 적용되는 설정정보는 `application.properties` 파일이 아닌 `application-{profile}.properties` 파일에 작성하면 된다.
- `application-dev.properties`, `application-test.properties`
사용할 프로파일 지정은 `application.properties` 파일 안에 지정하면된다.
```properties
spring.profiles.active=dev
```
`application-dev.properties` 가 로딩된다.
