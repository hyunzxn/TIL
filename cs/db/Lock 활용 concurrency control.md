# 1. 들어가며

여러 트랜잭션이 동시에 실행될 때 같은 데이터에 대해서 또 다른 read / write 작업이 있다면 예상치 못 한 동작이 발생할 수 있습니다. 이것을 방지하기 위해 `Lock` 이라는 것을 사용할 수 있습니다. 이번 글에서는 `Lock` 을 이용해서 데이터베이스의 동시성(Concurrency) 문제를 제어하는 방법에 대해 정리해보도록 하겠습니다.

참고: [쉬운 코드 - Lock을 이용한 데이터베이스 Concurrency Control]([LOCK을 활용한 concurrency control 기법을 배워봅니다. 2PL(two-phase locking)도 같이 설명드려요~ (youtube.com)](https://www.youtube.com/watch?v=0PScmeO3Fig&list=PLcXyemr8ZeoREWGhhZi5FZs6cvymjIBVe&index=18))

&nbsp;

# 2. 락(Lock)이란?

#### 개념

데이터마다 락(Lock)이 있어서 데이터를 read / write 하려면 그 락을 취득해야만 작업을 진행할 수 있습니다. 락을 취득하지 못 하면 데이터 조회 / 변경이 불가능 합니다.

#### 종류

- write-lock (exclusive lock)
  - read / write(insert, modify, delete) 할 때 사용 가능 → write-lock 이라 해서 반드시 write 작업에 대해서만 lock을 걸지 않는다는 것 주의!
  - 다른 트랜잭션이 동일한 데이터에 대해서 read / write 하는 것을 허용 X
- read-lock (shared lock)
  - read 할 때 사용되는 락입니다. 
  - 다른 트랜잭션이 동일한 데이터에 대해 read 하는 것을 허용합니다.
  - 다른 트랜잭션이 동일한 데이터에 대해 write 하는 것은 허용하지 않습니다.

<br>

락 호환성을 정리해보면 다음과 같습니다.

|    구분    | read-lock | write-lock |
| :--------: | :-------: | :--------: |
| read-lock  |     O     |     X      |
| write-lock |     X     |     X      |

&nbsp;

# 3. 락을 이용한 데이터베이스 동시성 제어

 락을 이용한다고 해서 바로 데이터베이스 동시성 문제가 해결되는 것은 아닙니다. 서로 다른 여러 트랜잭션 내에서 락을 취득하고 다시 락을 반환하는 오퍼레이션의 순서에 따라 스케줄이 None Serializable 할 수도 있습니다.

이런 문제를 방지 하기 위해서는 락을 취득하고 반환하는 순서가 중요해집니다. 가장 일반적인 방법으로는 `2PL Protocol` 이 있습니다. 

> **2PL Protocol**
>
> 트랜잭션 내에서 모든 락을 취득하는 오퍼레이션이 최초의 락을 반환하는 오퍼레이션보다 먼저 수행되도록 하는 것

`2PL Protocol` 을 사용하게 되면 스케줄을 Serializable 하게 만들 수 있다는 장점이 있지만 한계점도 존재합니다. 

예를 들어 트랜잭션 2가 x라는 데이터에 대한 read-lock을 가지고 있습니다. 그리고 트랜잭션 1은 y에 대한 read-lock을 가지고 있습니다. 이런 상황에서 트랜잭션 1이 x에 대한 write-lock을 획득하려면 트랜잭션 2가 가진 x에 대한 read-lock이 반환되기를 기다리고 있어야 합니다. 당연히 그동안 작업은 불가능하죠. 마찬가지로 트랜잭션 2가 y에 대한 write-lock을 획득하려면 트랜잭션 1이 가지고 있는 y에 대한 read-lock이 반환되기를 기다리고 있어야 합니다.

즉, 서로 다른 트랜잭션이 서로 다른 데이터에 대한 락을 각각 가지고 있을 때, 그 데이터에 대한 새로운 락을 획득하려면 한쪽에서 먼저 반환하기 전까지 아무런 작업도 진행되지 못 하는 상태가 발생할 수 있다는 것입니다. 그리고 이것을 `Deadlock` 현상이라고 합니다.

<br>

`2PL Protocol` 종류

1. Conservative 2PL
   - 모든 락을 취득한 뒤 트랜잭션을 시작
   - 데드락이 발생하지 않음
   - 실용적이지 않음
2. Strict 2PL
   - Strict Schedule을 보장하는 2PL
   - Recoverability를 보장
   - write-lock을 commit / rollback 될 때 반환
3. Strong Strict 2PL
   - Strict Schedule을 보장하는 2PL
   - Recoverability를 보장
   - read-lock / write-lock 모두 commit / rollback 될 때 반환
   - Strict 2PL 보다 구현이 쉬움

&nbsp;

# 4. lock 호환성의 한계 극복 - MVCC

위에서 소개한 락 호환성은 문제가 하나 있습니다. 동일한 데이터에 대해 write-write 작업을 동시에 하게 해주는 것은 데이터가 꼬일 수 있으니 lock으로 동시에 작업하지 못 하게 하는 것이 바람직하지만 read-write 같은 경우에는 굳이 그렇게까지 엄격하게 락을 걸 필요가 있는가? 하는 의문에서 더 개선된 방법이 나오게 됐는데 그것이 바로 `MVCC(Multi  Version Concurrency Control)` 입니다. 오늘날 많은 RDBMS 에서 Lock 과 MVCC 를 함께 사용하고 있습니다.