# 1. 들어가며

지난번에 정리한 MVCC - 1과 이어지는 2편의 글입니다. 

&nbsp;

# 2. Mysql의 MVCC Lost Update 대응

`Mysql` 은 `Postgresql` 에서와 Isolation Level 이 `repeatable read` 일 때, 먼저 update 한 트랜잭션이 commit 되면 나중 트랜잭션이 롤백을 시키지 않습니다. 따라서 `Mysql` 에서는 Lost Update 현상을 해결하기 위해서 별도의 추가적인 방식이 필요합니다. 이것을 `Locking Read` 라고 합니다.

예시를 보이자면 다음과 같습니다.

```SQL
SELECT balance
FROM account
WHERE id = 'x'
FOR UPDATE; // 개발자가 추가적으로 적어줘야 하는 부분!
```

<br>

`Mysql` 의 `Locking Read` 는 Isolation Level 과 관계없이 **가장 최근의** commit 된 데이터를 읽는다는 특징이 있습니다.

**Locking Read의 종류** 

- SELECT ... FOR UPDATE: Write 락을 획득
- SELECT ... FOR SHARE: Shared 락을 획득

