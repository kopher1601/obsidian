[[CrudRepository]] 를 상속받는 `PagingAndSortingRepository` 인터페이스도 제공하는데 이 인터페이스에는 Pagination 과 Sorting 기능이 포함돼 있다.

페이징은 많은 양의 데이터를 여러 페이지로 잘게 나눠 조회하는 기법이다. 페이징을 사용하면 서버 자원을 효율적으로 이용하면서 사용자에게 필요한 결과를 반환해줄 수 있다.
> 페이징은 데이터를 페이지라고 부르는 작은 단위로 나눠서 처리하는 기법이다.

스프링 데이터는 페이지 단위로 데이터를 자르고 정렬할 수 있는 `PagingAndSortingRepository` 인터페이스를 제공한다. `PagingAndSortingRepository` 인터페이스도 `CrudRepository` 인터페이스를 상속받으므로 CRUD 연산을 수행할 수 있다.

