# 1. 들어가며

[Lock을 이용한 DB Concurrency 제어 글]([TIL/cs/db/Lock 활용 concurrency control.md at main · hyunzxn/TIL (github.com)](https://github.com/hyunzxn/TIL/blob/main/cs/db/Lock 활용 concurrency control.md))에서 Lock을 이용한 방식의 한계점을 극복하기 위해 등장한 MVCC에 대해서 정리해보는 글입니다.

&nbsp;

# 2. MVCC 개념

`MVCC`란 Multi Version Concurrency Control의 약자입니다. Lock을 이용한 DB 동시성 제어의 한계점을 극복하기 위해서 등장했습니다. MVCC는 다음과 같은 구조를 가집니다.

|   구분    | read | write |
| :-------: | :--: | :---: |
| **read**  |  O   |   O   |
| **write** |  O   |   X   |

<br>

**`MVCC` 의 중요한 특징**

- 데이터를 읽을 때 특정 시점 기준으로 가장 최근에 commit 된 데이터를 읽습니다.
   - 트랜잭션의 `Isolation Level` 에 따라 달라질 수 있습니다.
     - repeatable read: 트랜잭션 시작 시간 기준으로 그 전에 commit 된 데이터를 읽습니다.
     - read committed: read 하는 시간 기준으로 그 전에 commit 된 데이터를 읽습니다.
     - read uncommitted: `MVCC` 는 커밋된 데이터를 읽기 때문에 이 level 에서는 보통 MVCC 가 적용되지 않습니다.
     - serializable: `Mysql` 의 경우에는 `MVCC` 로 동작하기 보다는 `lock`으로 동작합니다. `Postgresql` 의 경우에는 `SSI` 기법이 적용된 `MVCC` 로 동작합니다.
   - 그리고 이렇게 특정 시점을 기준으로 데이터를 읽는 것을 `Mysql` 에서는 `Consistent Read` 라고 합니다.
- 데이터 변화(write) 이력을 관리합니다.
  - 특정 시점을 충족시키기 위해서 변화 이력을 저장하기 위한 추가적인 저장 공간이 필요합니다.
- read 와 write 는 서로를 block 하지 않습니다. (위의 표 참고)
  - 동시에 처리할 수 있는 트랜잭션이 늘어나므로 성능상 이점이 있습니다.

<br>

----

**※ 참고**

`Postgresql` 은 Isolation level이 `repeatable read` 인 경우에는 같은 데이터에 먼저 update 한 트랜잭션이 commit 되면 나중 트랜잭션은 롤백 됩니다. → `first updater win`

