# 1. 들어가며

이번 글에서는 데이터베이스의 복구 가능성(recoverability) 및 트랜잭션들이 동시에 실행될 때 rollback이 발생하면 어떻게 되는지에 관해서 정리해보도록 하겠습니다.

&nbsp;

# 2. Unrecoverable Schedule

> 예제
>
> - A 잔액 100만원, B 잔액 200만원 시작.
> - A가 20만원을 B에게 이체하고, B가 30만원을 자신의 계좌에 입금하는 상황 가정
>
> 
>
> t1r(A): 100 → t1w(A): 80 → t2r(B): 200 → t2w(B): 230 → t1r(B): 230 → t1w(B): 250 → t1 커밋 → t2 롤백

<br>



예제에 따르면 t2를 롤백했으므로 t2에서 작업했던 +30만원의 작업은 무효가 됩니다. 그러면 B의 잔액은 다시 처음으로 200만원이 되게 됩니다. 그런데 지금 t1은 B의 잔액을 230만원으로 읽고 +20만원을 하고 커밋까지 했습니다. 

결과적으로 A가 20만원을 이체하여 A의 잔액은 80만원이 됐지만 t2의 롤백으로 인해 B의 잔액은 200만원이 되어 데이터의 불일치가 생기는 Lost Update 상황이 발생했습니다.

그런데 데이터베이스의 ACID 속성 중 Durability(지속성) 특성으로 인해 한 번 커밋된 트랜잭션은 취소할 수가 없습니다. 따라서 이런 경우는 아예 복구가 불가능하게 되어버립니다.

정리하자면 **스케줄 내에서 커밋된 트랜잭션이 롤백된 트랜잭션이 쓰기 작업을 한 데이터를 읽은 경우** 이런 스케줄을 `Unrecoverable Schedule` 이라 합니다. 이런 스케줄은 롤백을 해도 이전 상태로 복구하는 것이 불가능하기 때문에 DBMS 차원에서 사전에 제대로 방지하는 것이 중요합니다.

&nbsp;

# 3. Recoverable Schedule

> 예제
>
> - A 잔액 100만원, B 잔액 200만원 시작.
> - A가 20만원을 B에게 이체하고, B가 30만원을 자신의 계좌에 입금하는 상황 가정
>
>
> t1r(A): 100 → t1w(A): 80 → t2r(B): 200 → <u>t2w(B): 230 → t1r(B): 230</u> → t1w(B): 250 → **t2 커밋** → **t1 커밋**

<br>



복구 가능한 스케줄이 되게 하기 위해서는 서로 다른 트랜잭션의 커밋 순서가 중요합니다. 지금 예제에서 트랜잭션2 를 먼저 커밋하는 것을 확인할 수 있습니다. 이것은 트랜잭션1 이 트랜잭션 2가 쓰기 작업을 한 데이터에 의존을 하고 있기 때문입니다(예제 밑줄 친 부분). 이렇게 하면 설사 트랜잭션 2를 롤백하더라도 트랜잭션 1도 같이 롤백하면 되므로 복구가 가능하게 됩니다. 

포인트는 **임의의 트랜잭션이 다른 트랜잭션에 의존하고 있을 때 의존의 대상이 되는 트랜잭션이 커밋하기 전까지는 의존하고 있는 트랜잭션은 커밋을 해서는 안 되는 것**입니다.

정리하자면 **스케줄 내에 그 어떤 트랜잭션도 자신이 읽은 데이터를 쓰기 작업한 트랜잭션이 먼저 커밋 / 롤백 하기 전까지는 커밋 하지 않는 스케줄**을 `Recoverable Schedule`이라 합니다.

`Recoverable Schedule`에서 의존의 대상이 되는 트랜잭션이 롤백된다면 여기에 의존하고 있던 트랜잭션도 데이터 정합성 유지를 위해 롤백이 되어야 합니다. 선택의 여지가 없는 것이죠. 이렇게 **여러 트랜잭션들이 연쇄적으로 롤백이 되는 것**을 `Cascading Rollback`이라고 합니다.  `Cascading Rollback` 은 여러 트랜잭션 각각을 이전 상태로 되돌려야 하기 때문에 처리하는 비용이 많이 들게 됩니다.

<br>

### Cascadeless Schedule

`Cascading Rollback`의 한계점을 해소하기 위한 아이디어는 다음과 같습니다.

> 데이터를 write 한 트랜잭션이 커밋 / 롤백한 뒤에 데이터를 읽는 스케줄만 허용한다.

<br>

위의 예제에 적용해보면 다음과 같습니다. t1r(B) 오퍼레이션을 실행하기 전 무조건 t2를 커밋 / 롤백을 하고 나서 t1 의 다른 오퍼레이션들을 실행하는 것이죠. 이렇게 하면 트랜잭션 하나가 종료되기 전 다른 트랜잭션이 의존하게 되는 것을 방지할 수 있습니다. 문제가 생기더라도 하나의 트랜잭션만 롤백을 할 수 있게 되므로 비용이 적게 들게 됩니다.

이런 방식을 적용해서 **스케줄 내에서 어떤 트랜잭션도 커밋 되지 않은 트랜잭션이 write 한 데이터를 읽지 않는** 스케줄을 `Cascadeless Schedule`이라 합니다. 그러나 이 방식 역시도 한계점은 존재합니다. 

&nbsp;

### Strict Schedule

> 예제
>
> 피자 가게 사장(B)이 원래 3만원이던 피자의 가격을 2만원으로 낮추려고 할 때, 동시에 피자 가게 직원(A)이 피자의 가격을 1만원으로 낮추려 하는 경우 
>
>
> t1w(A): 피자 1만원 → t2w(B): 피자 2만원 → t2 커밋 → t1 롤백 → 피자 가격은 최종적으로 3만원

<br>

일단 예제에서 소개한 스케줄은 `Cascadeless Schedule` 입니다. 왜냐하면 아예 읽기 작업 자체가 없기 때문입니다. 그럼에도 불구하고 지금 피자 가격을 2만원으로 낮춘 t2의 결과가 사라지게 됩니다. 

따라서 이런 문제를 해결하기 위해서 추가적인 보완이 필요합니다. **스케줄 내에서 어떤 트랜잭션도 커밋 되지 않은 트랜잭션들이 write 한 데이터를 읽지 않을 뿐만 아니라 쓰지도 않아야 하는 것**입니다. 이것을 `Strict Schedule` 이라고 합니다.

예제를 `Strict Schedule`로 바꾸면 이렇게 됩니다. 

> t1w(A): 피자 1만원 → t1 커밋 / 롤백 → t2w(B): 피자 2만원



 

