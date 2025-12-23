HTTP Basic 인증은 자격 증명의 기밀성을 보장하지 않는다. Base64는 단지 전송의 편의를 위한 인코딩 방법이고 암호화나 해싱 방법이 아니므로 전송 중에 자격 증명을 가로채면 누구든지 볼 수 있다.

**영역(realm)** 은 특정 인증 방식을 이용하는 보호 공간으로 생각할 수 있다. 
```kotlin
http.httpBasic {
	it.realmName("OTHER") // realm 이름을 변경 가능
} 
```
영역은 HTTP Header 에서 확인할 수 있는데 200 성공 응답일때는 나타나지 않는다. 401 권한 없을 때만 표시 된다.

httpBasic 메서드를 호출하면 필터 체인에 `BasicAuthenticationFilter` 가 추가된다.

- `WWW-Authenticate: Basic realm="Realm"`
```HTTP
HTTP/1.1 401 
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Connection: keep-alive
Content-Length: 0
Date: Thu, 30 Oct 2025 02:54:40 GMT
Expires: 0
Keep-Alive: timeout=60
Pragma: no-cache
Set-Cookie: JSESSIONID=A71410B674AE82A17FF3257A18283E60; Path=/; HttpOnly
WWW-Authenticate: Basic realm="Realm"
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0
```
