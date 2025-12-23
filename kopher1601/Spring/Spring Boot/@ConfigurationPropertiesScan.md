[[@EnableConfigurationProperties]] 대신에 `@ConfigurationPropertiesScan` 를 사용해서 기준 패키지를 지정하면, 지정 패키지 하위에 있는 `@ConfigurationProperties` 가 붙어있는 클래스를 모두 탐색해서 스프링 컨테이너에 등록해준다. `@ConfigurationPropertiesScan` 애너테이션은 `@Component` 같은 애너테이션이 붙은 클래스가 아니라 `@ConfigurationProperties` 가 붙어 있는 클래스만 탐색해서 등록한다.

