

## 커스텀 CsrfTokenRepository
스프링 시큐리티 기본 구현과 같이 요청 헤더와 특성의 이름을 `X-CSRF-TOKEN` 및 `_csrf` 로 유지한다.
- [spring-security/ssia-ch10-ex3/src/main/java/com/laurentiuspilca/ssia/csrf/CustomCsrfTokenRepository.java at main · wikibook/spring-security](https://github.com/wikibook/spring-security/blob/main/ssia-ch10-ex3/src/main/java/com/laurentiuspilca/ssia/csrf/CustomCsrfTokenRepository.java)
```java
package com.laurentiuspilca.ssia.csrf;

import com.laurentiuspilca.ssia.entities.Token;
import com.laurentiuspilca.ssia.repositories.JpaTokenRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.web.csrf.CsrfToken;
import org.springframework.security.web.csrf.CsrfTokenRepository;
import org.springframework.security.web.csrf.DefaultCsrfToken;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Optional;
import java.util.UUID;

public class CustomCsrfTokenRepository
        implements CsrfTokenRepository {

    @Autowired
    private JpaTokenRepository jpaTokenRepository;

    @Override
    public CsrfToken generateToken(HttpServletRequest httpServletRequest) {
        String uuid = UUID.randomUUID().toString();
        return new DefaultCsrfToken("X-CSRF-TOKEN", "_csrf", uuid);
    }

    @Override
    public void saveToken(CsrfToken csrfToken, HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) {
        String identifier = httpServletRequest.getHeader("X-IDENTIFIER");
        Optional<Token> existingToken = jpaTokenRepository.findTokenByIdentifier(identifier);

        if (existingToken.isPresent()) {
            Token token = existingToken.get();
            token.setToken(csrfToken.getToken());
        } else {
            Token token = new Token();
            token.setToken(csrfToken.getToken());
            token.setIdentifier(identifier);
            jpaTokenRepository.save(token);
        }
    }

    @Override
    public CsrfToken loadToken(HttpServletRequest httpServletRequest) {
        String identifier = httpServletRequest.getHeader("X-IDENTIFIER");
        Optional<Token> existingToken = jpaTokenRepository.findTokenByIdentifier(identifier);

        if (existingToken.isPresent()) {
            Token token = existingToken.get();
            return new DefaultCsrfToken("X-CSRF-TOKEN", "_csrf", token.getToken());
        }

        return null;
    }
}
```
- `generateToken` : CSRF 보호 메커니즘은 애플리케이션이 새 토큰을 생성해야 할 때 generateToken 을 호출한다
- `saveToken` : 특정 클라이언트를 위해 생성된 토큰을 저장한다. 기본구현은 HTTP 세션을 이용한다. 이 경우에는 클라이언트에 고유한 식별자가 있다고 가정한다. 클라이언트는 요청에 이 고유ID 의 값을 `X-IDENTIFIER` 라는 헤더와 함께 보낸다
- `loadToken` : 토큰 세부 정보가 있으면 이를 로드하고 없으면 `null` 을 반환

### 커스텀 CsrfTokenRepository 등록
```java
@Bean
public CsrfTokenRepository customTokenRepository() {
	return new CustomCsrfTokenRepository();
}

@Override
protected void configure(HttpSecurity http) throws Exception {
	http.csrf(c -> {
		c.csrfTokenRepository(customTokenRepository());
		c.ignoringAntMatchers("/ciao");
	});

	http.authorizeRequests()
		 .anyRequest().permitAll();
}
```


