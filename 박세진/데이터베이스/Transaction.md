## Transaction

- 단일한 논리적인 작업 단위(a single logical unit of work)
- 논리적인 이유로 여러 SQL문들을 단일 작업으로 묶어서 나눠질 수 없게 만든 것이 transaction
- transaction의 SQL문들 중에 일부만 성공해서 DB에 반영되는 일은 일어나지 않는다
- 예시
    
    ```sql
    START TRANSACTION; # transaction을 시작
    UPDATE account SET balance = balance - 200000 WHERE id = 'J';
    UPDATE account SET balance = balance + 200000 WHERE id = 'H';
    COMMIT # 지금까지 작업한 내용을 DB에 영구적으로(permanently) 저장, transaction을 종료
    ```
    
    - START TRANSACTION 실행과 동시에 autocommit은 off된다
    - COMMIT / ROLLBACK과 함께 transaction이 종료되면 원래 autocommit 상태로 돌아간다

### Rollback

```sql
START TRANSACTION;
UPDATE account SET balance = balance - 300000 WHERE id = 'J';
ROLLBACK; #지금까지 작업들을 모두 취소하고 transaction 이전 상태로 되돌림, transaction 종료
```

### Autocommit

```sql
SELECT @@AUTOCOMMIT; 
#각각의 SQL문을 자동으로 transaction 처리
#SQL문이 성공적으로 실행하면 자동으로 commit
#실행 중에 문제가 있었다면 알아서 rollback
#MYSQL에서는 default로 autocommit이 enabled
#다른 DBMS에서도 대부분 같은 기능 제공
```

### 일반적인 transaction 사용 패턴

- transaction을 시작한다
- 데이터를 읽거나 쓰는 등의 SQL문들을 포함해서 로직을 수행한다
- 일련의 과정들이 문제없이 동작했다면 transaction을 commit 한다
- 중간에 문제가 발생했다면 transaction을 rollback 한다

## ACID

### Atomicity

- ALL or NOTHING
- transaction은 논리적으로 쪼개질 수 없는 작업 단위이기 때문에 내부의 SQL문들이 모두 성공해야 함
- 중간에 SQL문이 실패하면 지금까지의 작업을 모두 취소하여 아무 일도 없었던 것처럼 rollback

<aside>
💡 - commit 실행 시 DB에 영구적으로 저장하는 것은 DBMS가 담당하는 부분
- rollback 실행 시 이전 상태로 되돌리는 것도 DBMS가 담당하는 부분
- 개발자는 언제 commit 하거나 rollback 할지를 챙겨야 함

</aside>

### Consistency

- transaction은 DB 상태를 consistent 상태에서 또 다를 consistent 상태로 바꿔줘야 함
- constraints, trigger등을 통해 DB에 정의된 rules을 transaction이 위반했다면 rollback 해야 함
- transaction이  DB에 정의된 rule을 위반했는지는 DBMS가 commit전에 확인하고 알려줌
- 그 외에 application 관점에서 transaction이 consistent하게 동작하는지는 개발자가 챙겨야 함

### Isolation

- 여러 transaction들이 동시에 실행될 때도 혼자 실행되는 것처럼 동작하게 만듬
- DBMS는 여러 종류의 isolation level을 제공
- 개발자는 isolaction level중에 어떤 level로 transaction을 동작시킬지 설정할 수 있음
- concurrency control의 주된 목표가 isolation

### Durability

- commit된 transaction은 DB에 영구적으로 저장
- 즉, DB system에 문제(power fail or DB crash)가 생겨도 commit된 transaction은 DB에 남아 있음
- ‘영구적으로 저장한다’라고 할 때는 일반적으로 ‘비휘발성 메모리(HDD, SDD, …)에 저장함’을 의미
- 기본적으로 transaction의 durability는 DBMS가 보장

## Schedule

- 여러 transaction들이 동시에 실행될 때 각 transaction에 속한 operation들의 실행 순서
- 각 transaction 내의 operations들의 순서는 바뀌지 않음

### **Serial schedule**: transaction들이 겹치지 않고 한 번에 하나씩 실행되는 schedule

- sched.1: r1(k)→w1(k)→r1(H)→w1(H)→c1→r2(H)→w2(H)→c2
- sched.2: r2(H)→w2(H)→c2→r1(K)→w1(K)→r1(H)→w1(H)→c1

<aside>
💡 **Serial schedule 성능**
한 번에 하나의 transaction만 실행되기 때문에 좋은 성능을 낼 수 없고 현실적으로 사용할 수 없는 방식

</aside>

### Nonserial schedule: transaction들이 겹쳐서(interleaving) 실행되는 schedule

- sched.3: r1(K)→w1(K)→r2(H)→w2(H)→c2→r1(H)→w1(H)→c1
- sched.4: r1(K)→w1(K)→r1(H)→r2(H)→w2(H)→c2→w1(H)→c1

<aside>
💡 **Nonserial schedule 성능**
transaction들이 겹쳐서 실행되기 때문에 동시성이 높아져서 같은 시간 동안 더 많은 transaction들을 처리할 수 있음

**Nonserial Schedule 단점**
transaction들이 어떤 형태로 겹쳐서 실행되는지에 따라 이상한 결과가 나올 수 있음

</aside>

### Conflict of two operations

세 가지 조건을 모두 만족하면 conflict

- 서로 다른 transaction 소속
- 같은 데이터에 접근
- 최소 하나는 write operation

conflict operation은 순서가 바뀌면 결과도 바뀐다

### Conflict equivalent for two schedules

두 조건 만족하면 conflict equivalent

1. 두 schedule은 같은 transaction들을 가진다
2. 어떤(any) conflicting operations의 순서도 양쪽 schedule 모두 동일하다

### Conflict serializable

serial schedule과 conflict equivalent 일 때

<aside>
💡 여러 transaction을 동시에 실행해도 schedule이 conflict serializable하도록 보장하는 프로토콜을 적용

</aside>

## Recoverability

### unrecoverable schedule

- rollback 해도 이전 상태로 회복 불가능할 수 있기 때문에 이런 schedule은 DBMS가 허용하면 안됨

### recoverable schedule

- schedule 내에서 그 어떤 transaction도 자신이 읽은 데이터를 write한 transaction이 먼저 commit/rollabck 전까지는 commit 하지 않는 경우
- rollback할 때 이전 상태로 온전히 돌아갈 수 있기 때문에 DBMS는 이런 schedule만 허용해야 함
- **cascading rollback**
    - 하나의 transaction이 rollback하면 의존성이 있는 다른 transaction도 rollback 함
    - 여러 transaction의 rollback이 연쇄적으로 일어나면 처리하는 비용이 많이 듦
- **cascadeless schedule**
    - schadule 내에서 어떤(any) transaction도 commit되지 않은 transaction들이 write한 데이터는 읽지 않는 경우
- **strict schedule**
    - schadule 내에서 어떤(any) transaction도 commit되지 않은 transaction들이 write한 데이터는 쓰지도 읽지도 않는 경우
    - rollback 할 때 recovery가 쉬움. transaction 이전 상태로 돌려놓기만 하면 됨

## Isolation Level

- **Dirty read**: commit되지 않은 변화를 읽음
- **Non-repeatable read**: 같은 데이터의 값이 달라짐
- **Phantom read**: 없던 데이터가 생김
- **Dirty Wirte**: commit 안된 데이터를 write 함
- **Lost update**: 업데이트를 덮어 씀
- **Read skew**: inconsistent한 데이터 읽기
- **Wirte skew**: inconsistent한 데이터 쓰기

| Isolation level | Dirty read | Non-repeatable read | Phantom read |
| --- | --- | --- | --- |
| Read uncommitted | O | O | O |
| Read committed | X | O | O |
| Repeatable read | X | X | O |
| Serializable | X | X | X |

<aside>
💡 세 가지 이상 현상을 정의하고 어떤 현상을 허용하는지에 따라서 각각의 isolation level이 구분된다. 애플리케이션 설계자는 isolation level을 통해 전체 처리량(throughput)과 데이터 일관성 사이에서 어느 정도 거래(trade)를 할 수 있다.

</aside>

### Snapshot Isolation