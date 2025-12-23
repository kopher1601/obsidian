`JdbcUserDetailsManager` 도 자주 이용한다. `JdbcUserDetailsManager` 는 SQL 데이터베이스에 저장된 사용자를 관리하며 JDBC 를 통해 데이터베이스에 직접 연결한다. 이처럼 `JdbcUserDetailsManager` 는 데이터베이스 연결과 관련한 다른 프레임워크나 사양으로부터 독립적일 수 있다.
- `JdbcUserDetailsManager` 구현에 이용되는 모든 쿼리를 변경할 수 있다

![[Pasted image 20251028150918.png]]

`users` 및 `authorities` 테이블은 `JdbcUserDetailsManager` 가 인식하는 기본 테이블 이름이다. `JdbcUserDetailsManager` 구현은 아주 유연하므로 원하면 이러한 기본 이름을 바꿀 수 있다.
`users` 테이블의 목적은 사용자 레코드를 저장하는 것이다. `JdbcUserDetailsManager` 구현은 `users` 테이블에 사용자 이름, 암호, 그리고 사용자 활성화 여부를 저장하는 세 열이 있다고 가정한다.

**예제 스크립트**
- `schema.sql`
```sql
create schema spring;

CREATE TABLE IF NOT EXISTS `spring`.`users` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `username` VARCHAR(45) NULL,
  `password` VARCHAR(45) NULL,
  `enabled` INT NOT NULL,
  PRIMARY KEY (`id`));

CREATE TABLE IF NOT EXISTS `spring`.`authorities` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `username` VARCHAR(45) NULL,
  `authority` VARCHAR(45) NULL,
  PRIMARY KEY (`id`));
```
- `data.sql`
```sql
INSERT INTO `spring`.`authorities` VALUES (NULL, 'john', 'write');
INSERT INTO `spring`.`users` VALUES (NULL, 'john', '12345', '1');
```
