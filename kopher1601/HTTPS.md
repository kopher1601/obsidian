개인 키 & 공개 인증서 생성
```shell
openssl req -newkey rsa:2048 -x509 -keyout key.pem -out cert.pem -days 365
```
위에 생성한 두 파일을 받아 그 결과로 자체 서명 인증서를 생성
```shell
openssl pkcs12 -export -in cert.pem -inkey key.pem -out certificate.p12 -name "certificate"
```
자체 서명 인증서를 이용할 때는 엔드포인트를 호출하는 툴에서 인증서 신뢰성 테스트를 생략하도록 구성해야 한다.
```shell
https --verify=no -a user:9cd7f942-b39e-4ad5-a2ad-77731331e63b -v :8080/hello
```
