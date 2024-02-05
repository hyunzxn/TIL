# Functional Dependecny(함수 종속)란?

함수 종속이란 한 테이블에 있는 **두 개의 attribute(RDB에서의 Column) 집합 사이의 제약**을 의미합니다.

| empl_id | empl_name | birth_date | position  |  salary   | dept_id |
| :-----: | :-------: | :--------: | :-------: | :-------: | :-----: |
|    1    |  홍길동   | 1988-05-05 | DEVELOPER | 100000000 |    1    |
|    2    |  박철수   | 1997-12-10 | MARKETER  | 50000000  |    2    |



 `empl_id` 를 집합 X, 그 외 나머지를 집합 Y 라고 구분을 했다고 가정을 했습니다. `empl_id` 가 각 row의 고유한 식별을 위한 Column이라고 한다면 집합 Y의 값은 집합 X에 완전히 종속되게 됩니다.

조금 더 쉽게 설명을 해 보겠습니다. `empl_id`가 1이라면 이건 무조건 홍길동이라는 직원을 의미하는 것이기 때문에 나머지 값들은 반드시 홍길동 사원의 값이어야 합니다. 마찬가지로 `empl_id`가 2라면 나머지 값들은 박철수 사원의 값이어야 하는 것입니다.

이렇게 X 값에 따라 Y 값이 유일하게 결정될 때 **X가 Y를 함수적으로 결정한다** 또는 **Y가 X에 함수적으로 의존한다** 라고 표현합니다. 그리고 이런 두 집합 사이의 관계를 함수 종속(FD)이라고 합니다. 기호로는 **X → Y** 이렇게 표현합니다.

주의할 점으로는 **X → Y** 라고 하여 **Y → X** 는 아니라는 것입니다. 우연의 일치로 이름, 생년월일, 포지션, 연봉, 부서가 다 같은데 `empl_id` 가 다른 경우가 있을 수도 있기 때문입니다.

&nbsp;

# Functional Dependency의 종류

## 1. Trivial Functional Dependency

> when X → Y holds, if Y is subset of X, then X → Y is Trivial FD
>
> ex) { a, b, c } → { c }  
>
> ex) { a, b, c } → { a , b }

## 2. Non-Trivial Function Dependency

> when X → Y holds, if Y is not subset of X, then X → Y is Non-Trivial FD
>
> ex) { a, b, c } → { b, c, d }
>
> ex) { a, b, c } → { d, e } => 이 경우는 X와 Y가 겹치는게 하나도 없다. 이런 경우를 Completely Non-Trivial FD 라고 한다.

## 3. Partial Functional Dependency

> when X → Y holds, if any proper subset of X can determine Y, then X → Y is Partial FD
>
> Proper Subset => 집합 X의 Proper Subset은 X의 부분 집합이지만 X와 동일하지는 않은 집합
>
> ex) { empl_id, empl_name } → { birth_date } 이런 FD가 있는데 생각해보면 empl_id 만으로도 birth_date를 결정하는 것이 가능하므로 { empl_id } → { birth_date } 가능하다. 이런 경우를 Partial FD라고 한다.

## 4. Full Functional Dependency

> when X → Y holds, if every proper subset of X can not determine Y, then X → Y is Full FD
>
> ex) { stu_id, class_id } → { grade } 그런데 이런 FD는 X의 부분 집합만으로는 grade를 결정할 수 없다. 학생이 여러 수업을 들었을 수도 있고, 한 수업에서도 학생마다 여러 점수를 받을 수 있기 때문이다.