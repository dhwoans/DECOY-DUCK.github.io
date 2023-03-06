## TCL (transaction control language)

DCL에서 트랜젝션을 제어하는 명령인 commit 과 rollback 만 따로 분리해서 TCL이라고 한다.

`COMMIT`

> 입력, 수정, 삭제한 자료에 대해 문제가 없을 경우 COMMIT 명령어를 통해 트랜잭션을 완료

COMMIT이나 ROLLBACK을 통해 데이터베이스에 반영하기 전에는 메모리 버퍼에만 영향을 준 상태여서 변경 이전 상태로 복구 할 수 있음

COMMIT이나 ROLLBACK을 통해 데이터베이스에 반영이 되면 이전 데이터는 영원히 잃어버림

```sql
COMMIT;
```

`ROLLBACK`

> 테이블 내 입력, 수정, 삭제한 데이터에 대해서 COMMIT 이전에 변경 사항을 취소할 수 있는 기능

- **효과**
  -   데이터 무결성 보장
  -   영구적 변경을 하기 전에 데이터 변경 사항 확인 가능
  -   논리적으로 연관된 작업을 그룹핑하여 처리 가능

```sql
ROLLBACK;
```

`SAVEPOINT`

> 저장점을 정의하면 롤백할때 전체 롤백이 아닌 일부만 롤백할 수 있음

-   복수의 저장점 정의 가능
-   동일 이름으로 저장점을 저장하면 나중에 정의한 저장점이 유효
-   저장점 정의하고 저장점으로 롤백할 수 있음
-   특정 저장점까지 롤백하면 그 저장점

```sql
SAVEPOINT SVPT1;
ROLLBACK TO SVPT1;
```