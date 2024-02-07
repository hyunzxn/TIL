# 1. DB 정규화(Normalization)란?

정규화란 데이터 중복과 insertion, update, deletion anomaly 를 최소화하기 위해 일련의 `Normal Forms(NF)`에 따라 관계형 DB를 구성하는 과정을 의미합니다. 여기서 `Normal Form` 이란 정규화 되기 위해 준수해야되는 룰(rule)들을 의미합니다. NF는 순차적으로 만족을 해야된다는 특징이 있습니다. 따라서 앞 단계의 NF를 만족하지 못 하면 뒷 단계로의 NF는 당연히 진행이 불가합니다. 단계는 아래와 같습니다.

> 1NF → 2NF → 3NF → BCNF → 4NF → 5NF → 6NF  

<br>



**1NF ~ BCNF 특징**

- FD 와 key 만으로 정규화 가능
- 보통 3NF 까지 달성하면 정규화 됐다고 말함
- 실무에서는 3NF 또는 BCNF 까지만 정규화하는 것이 일반적

&nbsp;

# 2. 예제 테이블 스키마

| bank_name |  account_num   | <u>account_id</u> | class  | ratio | empl_id | empl_name |  card_id  |
| :-------: | :------------: | :---------------: | :----: | :---: | :-----: | :-------: | :-------: |
|   Woori   | 010-9231-1121  |        a11        | BRONZE |  0.1  |   e1    |   Sony    |   c101    |
|   Woori   | 102-992-180124 |        a12        | SILVER |  0.2  |   e1    |   Sony    |   c102    |
|  Kookmin  | 010-9231-1121  |        a13        | LOYAL  |  0.7  |   e1    |   Sony    |   c103    |
|  kookmin  | 010-1121-1732  |        a21        | LOYAL  |   1   |   e2    |   Messi   | c201 c202 |

[참고]

- 은행의 종류는 국민은행, 우리은행 2가지만 있다고 가정한다.
- class(등급)는 국민은행, 우리은행이 종류가 달라 중복되지 않는다고 가정한다.
  - 국민: STAR → PRESTIGE → LOYAL
  - 우리: BRONZE → SILVER → GOLD

---

예제 테이블에서 key가 될 수 있는 것들을 뽑아보면 다음과 같습니다.

- **Super key**: table에서 tuple들을 유니크하게 식별할 수 있는 attribute들의 집합

- **(Candidate) Key**: 어느 한 attribute라도 제거하면 유니크하게 tuple을 식별할 수 없는 Super Key
  - {account_id} : 각 계좌를 식별하는 고유한 id 이기 때문에 가능하다.
  - {bank_name, account_num} : 은행마다 계좌번호는 고유하기 때문에 은행 + 계좌번호 조합은 key 가 될 수 있다.
- **Primary Key**: 테이블에서 tuple들을 유니크하게 식별하기 위해 선택된 (Candidate) key 
  - {account_id} : (Candidate) Key 후보 중 account_id 가 attritbute 수가 적기 때문에 여러가지 이점이 있어서 이것을 PK로 선택
- **Prime Attribute**: 임의의 key에 속하는 attribute
  - account_id
  - bank_name
  - account_num
- **Non-Prime Attribute**: Prime Attribute 가 아닌 나머지 모든 Attribute

<br>

Functional Dependency(FD) 에 따르면 다음과 같이 표현할 수 있습니다.

> {account_id} → {bank_name, account_num, class, ratio, empl_id, empl_name, card_id}
>
> {bank_name, accout_num} → {account_id, class, ratio, empl_id, empl_name, card_id}
>
> {empl_id} → {empl_name}
>
> {class} → {bank_name}

&nbsp;

# 3. 정규화 시작

## 1NF: attribute의 value는 반드시 나눠질 수 없는 단일한 값이어야 한다.

위에서 소개한 예제 테이블의 맨 마지막 행(row)의 card_id를 살펴보면 c201, c202 두 가지 value를 동시에 가지고 있습니다. 이것은 `1NF` 에 위배됩니다. 이것을 위해서 별도의 행(row)을 추가하면 다음과 같습니다.

| bank_name |  account_num   | <u>account_id</u> | class  | ratio | empl_id | empl_name | <u>card_id</u> |
| :-------: | :------------: | :---------------: | :----: | :---: | :-----: | :-------: | :------------: |
|   Woori   | 010-9231-1121  |        a11        | BRONZE |  0.1  |   e1    |   Sony    |      c101      |
|   Woori   | 102-992-180124 |        a12        | SILVER |  0.2  |   e1    |   Sony    |      c102      |
|  Kookmin  | 010-9231-1121  |        a13        | LOYAL  |  0.7  |   e1    |   Sony    |      c103      |
|  kookmin  | 010-1121-1732  |        a21        | LOYAL  |   1   |   e2    |   Messi   |      c201      |
|  kookmin  | 010-1121-1732  |        a21        | LOYAL  |   1   |   e2    |   Messi   |      c202      |

→ 이 테이블은 `1NF` 를 만족합니다. 그러나 중복 데이터가 생기고 PK도 변경을 해줘야 합니다. PK로 선택한 account_id 가 a21로 동일하게 존재하기 때문입니다. 그래서 card_id 까지 PK에 추가해줘야 합니다.

&nbsp;

## 2NF: 모든 Non-Prime Attribute는 모든 key에 대하여 Fully Functionally Dependent 해야한다.

`1NF` 를 만족시키고 난 뒤의 Key를 살펴보면 다음과 같습니다.

- {account_id, **card_id**}
- {bank_name, account_num, **card_id**}

기존의 key에 card_id 가 추가된 것을 확인할 수 있습니다.

<br>

이 상황에서 Non-Prime Attribute를 찾아보도록 하겠습니다. 

- class
- ratio
- empl_id
- empl_name

그런데 Non-Prime Attribute 들을 자세히 살펴보면 card_id 없이도 account_id 만으로도 결정지을 수 있다는 것을 확인할 수 있습니다. 마찬가지로 bank_name + account_num 만으로도 결정지을 수 있습니다. 

결론적으로 지금 데이터 중복이 발생하는 이유의 근원은 card_id 때문입니다. 따라서 이 부분을 손보면 데이터 중복을 방지할 수 있을 것 같습니다. 이럴 때 등장하는 것이 이제 `2NF`  입니다. 완성된 테이블의 모습은 다음과 같습니다.

| bank_name |  account_num   | <u>account_id</u> | class  | ratio | empl_id | empl_name |
| :-------: | :------------: | :---------------: | :----: | :---: | :-----: | :-------: |
|   Woori   | 010-9231-1121  |        a11        | BRONZE |  0.1  |   e1    |   Sony    |
|   Woori   | 102-992-180124 |        a12        | SILVER |  0.2  |   e1    |   Sony    |
|  Kookmin  | 010-9231-1121  |        a13        | LOYAL  |  0.7  |   e1    |   Sony    |
|  kookmin  | 010-1121-1732  |        a21        | LOYAL  |   1   |   e2    |   Messi   |

<br>

| <u>account_id</u> | <u>card_id</u> |
| :---------------: | :------------: |
|        a11        |      c101      |
|        a12        |      c102      |
|        a13        |      c103      |
|        a21        |      c201      |
|        a21        |      c202      |

&nbsp;

## 3NF: 모든 Non-Prime Attribute는 어떤 key에 대하여 Transitively Dependent 하면 안 된다.  => Non-Prime Attribute 와 Non-Prime Attribute 사이에는 FD가 있으면 안 된다.

`2NF` 에서 완성된 테이블을 보면 empl_name 이 Sony인 데이터가 3개가 있는데요. empl_name 과 empl_id 는 다음과 같은 FD 관계가 있습니다.

> {empl_id} → {empl_name}

그리고  account_id 와 empl_id 도 다음과 같은 FD 관계가 있습니다.

> {account_id} → {empl_id}

그러면 3단 논법에 의해서 최종적으로 다음과 같은 FD 관계를 도출할 수 있습니다.

>  {account_id} → {empl_name}

한편으로 {bank_name, account_num} → {empl_id} FD 관계도 발견할 수 있습니다. 그러면 결과적으로 또 다음과 같은 FD 관계를 도출할 수 있습니다.

> {bank_name, account_num} → {empl_name}

<br>

이렇게 연결-연결해서 새로운 FD 를 도출할 수 있는데 이 때의 FD 를 `Transitive FD` 라고 합니다.

> **Transitive FD**
>
> if X → Y & Y → Z holds, then X → Z is Transitive FD. Unless either Y or Z is not subset of any key.

&nbsp;

[EMPLOYEE_ACCOUNT]

| bank_name |  account_num   | <u>account_id</u> | class  | ratio | empl_id |
| :-------: | :------------: | :---------------: | :----: | :---: | :-----: |
|   Woori   | 010-9231-1121  |        a11        | BRONZE |  0.1  |   e1    |
|   Woori   | 102-992-180124 |        a12        | SILVER |  0.2  |   e1    |
|  Kookmin  | 010-9231-1121  |        a13        | LOYAL  |  0.7  |   e1    |
|  kookmin  | 010-1121-1732  |        a21        | LOYAL  |   1   |   e2    |

[ACCOUNT]

| <u>account_id</u> | <u>card_id</u> |
| :---------------: | :------------: |
|        a11        |      c101      |
|        a12        |      c102      |
|        a13        |      c103      |
|        a21        |      c201      |
|        a21        |      c202      |

[EMPLOYEE]

| empl_id | empl_name |
| :-----: | :-------: |
|   e1    |   Sony    |
|   e2    |   Messi   |

→ 이렇게 테이블을 3개 구성하면 `3NF` 를 만족해서 테이블을 설계한 것입니다. `3NF` 까지 되면 정규화 됐다고 말할 수 있습니다.

&nbsp;

## BCNF: 모든 유효한 Non-Trivial FD X → Y는 X가 Super Key여야 한다.

EMPLOYEE_ACCOUNT 테이블을 보게 되면 {class} → {bank_name} FD가 있음을 확인할 수 있습니다. 예제 테이블 참고 사항에 의해서 말이죠. 최종적으로 `BCNF` 까지 만족한 테이블들의 모습은 다음과 같습니다.

[EMPLOYEE_ACCOUNT]

|  account_num   | <u>account_id</u> | class  | ratio | empl_id |
| :------------: | :---------------: | :----: | :---: | :-----: |
| 010-9231-1121  |        a11        | BRONZE |  0.1  |   e1    |
| 102-992-180124 |        a12        | SILVER |  0.2  |   e1    |
| 010-9231-1121  |        a13        | LOYAL  |  0.7  |   e1    |
| 010-1121-1732  |        a21        | LOYAL  |   1   |   e2    |

[ACCOUNT]

| <u>account_id</u> | <u>card_id</u> |
| :---------------: | :------------: |
|        a11        |      c101      |
|        a12        |      c102      |
|        a13        |      c103      |
|        a21        |      c201      |
|        a21        |      c202      |

[EMPLOYEE]

| empl_id | empl_name |
| :-----: | :-------: |
|   e1    |   Sony    |
|   e2    |   Messi   |

[ACCOUNT_CLASS]

|  class   | bank_name |
| :------: | :-------: |
|  BRONZE  |   Woori   |
|  SILVER  |   Woori   |
|   GOLD   |   Woori   |
|   STAR   |  Kookmin  |
| PRESTIGE |  Kookmin  |
|  LOYAL   |  Kookmin  |