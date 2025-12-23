- **DML (Data Manipulation Language)**: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE` 등 - 데이터 조작

**외래키 제약 조건 설정**
```sql
set foreign_key_checks = 0;  
set foreign_key_checks = 1;
```


`DROP TABLE` : 테이블의 존재 자체를 삭제한다.
- `DROP TABLE orders;` 를 실행하면, `orders` 테이블의 모든 데이터는 물론, `orders` 라는 테이블의 구조(설계도)까지 완전히 사라진다. 테이블을 다시 사용하려면 `CREATE TABLE` 부터 다시 해야 한다. 마치 건물을 통째로 철거하는 것과 같다.

`TRUNCATE TABLE` : 테이블의 구조는 남기고, 내부 데이터만 모두 삭제한다.
- `TRUNCATE TABLE orders;` 를 실행하면, `orders` 테이블 안의 모든 주문 데이터가 순식간에 사라진다. 하지만 `orders` 라는 테이블의 구조(열, 제약조건 등)는 그대로 남아있어서, 바로 새로운 데이터를 `INSERT` 할 수 있다. 건물의 내부만 싹 비우고 뼈대는 그대로 두는 것과 같다.
**특징**
- `DELETE FROM orders;` (WHERE 절 없는 DELETE)와 결과적으로는 같아 보이지만, `TRUNCATE` 가 훨씬 빠르다. `DELETE` 는 한 줄씩 지우면서 삭제 기록을 남기는 반면, `TRUNCATE` 는 테이블을 초기화하는 개념이라 내부 처리 방식이 더 간단하고 빠르다.
- `TRUNCATE` 는 `AUTO_INCREMENT` 값도 초기화한다. 만약 `orders` 테이블에 1000개의 주문이 있어서 다음 `order_id` 가 1001일 차례였다면, `TRUNCATE` 이후에는 다시 1부터 시작한다. (DELETE는 `AUTO_INCREMENT` 값을 초기화하지 않는다.)
