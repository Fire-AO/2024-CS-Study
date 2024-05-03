## Lock

### write-lock (exclusive lock)

- read/write(insert, modift, delete) 할 때 사용
- 다른 transaction이 같은 데이터를 read/write 하는 것을 허용하지 않음

### read-lock (shared lock)

- read 할 때 사용
- 다른 transaction이 같은 데이터를 read 하는 것을 허용

### lock 호환성

|  | read-lock | write-lock |
| --- | --- | --- |
| read-lock | O | X |
| write-lock | X | X |

read-read를 제외하고는 한 쪽이 block이 되니까 전체 처리량이 좋지 않다.

## 2PL Protocol (two-phase locking)

### 2PL Protocol

- TX에서 모든 locking operation이 최초의 unlock operation 보다 먼저 수행되도록 하는 것

### Expanding phase (growing phase)

- lock을 취득하기만 하고 반환하지는 않는 phase

### Shrinking phase (contracting phase)

- lock을 반환만 하고 취득하지는 않는 phase

### conservative 2PL

- 모든 lock을 취득한 뒤 transaction을 시작
- deadlock-free
- 실용적이진 않다

### strict 2PL (S2PL)

- strict schedule을 보장하는 2PL
- recoverability 보장
- write-lock을 commit / rollback 될 때 반환

### strong strict 2PL (SS2PL or rigorous 2PL)

- strict schedule을 보장하는 2PL
- recoverability 보장
- read-lock / write-lock 모두 commit / rollback 될 때 반환
- S2PL보다 구현이 쉽다