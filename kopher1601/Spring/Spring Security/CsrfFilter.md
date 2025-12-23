`CsrfFilter` 는 요청을 가로채고 `GET`, `HEAD`, `TRACE`, `OPTIONS` 를 포함하는 HTTP 방식의 요청을 모두 허용하고 다른 모든 요청에는 토큰이 포함된 헤더가 있는지 확인한다. 이 헤더가 없거나 헤더에 잘못된 토큰 값이 포함된 경우 애플리케이션은 요청을 거부하고 응답의 상태를 `403 Forbidden` 으로 설정한다.
![[Pasted image 20251107195559.png]]
`CsrfFilter` 는 [[CsrfTokenRepository]] 구성 요소를 이용해 새 토큰 생성, 토큰 저장, 토큰 검증에 필요한 CSRF 토큰 값을 관리한다. 기본적으로 `CsrfTokenRepository` 는 토큰을 HTTP 세션에 저장하고 랜덤 UUID 로 토큰을 생성한다. 
![[Pasted image 20251107195831.png]]

`CsrfFilter` 는 생성된 CSRF 토큰을 HTTP Request의 `_csrf` 특성에 추가한다.
```html
<!DOCTYPE HTML>  
<html lang="en" xmlns:th="http://www.thymeleaf.org">  
<head>  
</head>  
<body>  
<form action="/product/add" method="post">  
    <span>Name:</span>  
    <span><input type="text" name="name" /></span>  
    <span><button type="submit">Add</button></span>  
    <input type="hidden"  
           th:name="${_csrf.parameterName}"  
           th:value="${_csrf.token}" />  
</form>  
</body>  
</html>
```
![[Screenshot 2025-11-07 at 20.46.52.png]]
![[Screenshot 2025-11-07 at 20.47.06.png]]

