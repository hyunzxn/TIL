
# JPA Fetch Join

>JPQL 에서 지원하는 join fetch 기능은 주로 DB에서 데이터를 조회할 때 필요했던 여러 개의 쿼리를 하나의 쿼리로 줄임으로써 쿼리 성능을 향상시킬 때 자주 사용한다. 실무에서 굉장히 자주 사용되는 기술이므로 제대로 알아두는 것이 중요하다.


### 1. 일반 조인과 페치 조인의 차이

일반 조인과 페치 조인의 가장 큰 차이를 꼽자면 **연관된 엔티티를 함께 조회하는지**의 여부일 것이다. 

- 일반 조인: 연관된 엔티티를 함께 조회하지 않는다.
- 페치 조인: 연관된 엔티티를 함께 조회한다.


[일반 조인 예시]
```
// JPQL 
select t from Team t join t.members m where t.name = ‘팀A' 

// 실제 나가는 SQL
SELECT T.* FROM TEAM T 
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID 
WHERE T.NAME = '팀A'
```

[페치 조인 예시]
```
// JPQL 
select t from Team t join fetch t.members where t.name = ‘팀A' 

// 실제 나가는 SQL 
SELECT T.*, M.* FROM TEAM T 
INNER JOIN MEMBER M ON T.ID = M.TEAM_ID 
WHERE T.NAME = '팀A'
```

페치 조인을 걸어주게 되면 연관된 엔티티를 모두 함께 조회하므로 실제 SQL에서도 Team과 Member에 관한 걸 한 번에 조회하는 것을 확인할 수 있다. 페치 조인을 사용하게 되면 연관된 엔티티도 한 번에 조회하므로 사실상 즉시로딩이라고 할 수도 있겠다.


### 2. 엔티티 페치 조인

특정 엔티티를 조회할 때 연관된 엔티티를 같이 페치 조인으로 조회하는 경우이다.

[코드 예시]
```
// JPQL
select m from Member m join fetch m.team

// 실제 나가는 SQL
SELECT M.*, T.* 
FROM MEMBER M
INNER JOIN TEAM T ON M.TEAM_I D = T.ID
```

![](https://blog.kakaocdn.net/dn/dChWcE/btrSljGMeoo/9JYGaZwOHruqGcr65UvoC0/img.png)


Member를 조회하고 있지만 연관된 Team까지 같이 조회하고 있다.


### 3. 컬렉션 페치 조인

특정 엔티티를 조회할 때 연관된 컬렉션을 같이 페치 조인으로 조회하는 경우이다. 이 경우에는 소위 '**데이터 뻥튀기**' 라는 것을 주의할 필요가 있다.

```
// JPQL 
select t from Team t join fetch t.members where t.name = ‘팀A' 

// 실제 나가는 SQL SELECT T.*, M.* FROM TEAM T 
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID 
WHERE T.NAME = '팀A'
```


![](https://blog.kakaocdn.net/dn/bpYhGE/btrSlHgivuz/jfb9Tw2BZCmVMuEQfDLwQ0/img.png)

팀A 입장에서 멤버를 두 명 가지고 있기 때문에 데이터베이스는 두 개의 데이터를 보여준다. 조회하고 싶은 데이터는 teamA라는 하나의 데이터인데 결과가 두 개가 나왔으니 이걸 소위 "데이터 뻥튀기"라고 하는 것이다. 그리고 이것을 JPA는 객체스럽게 변경해주는데 다음과 같이 객체 형태가 생성된다.

1. teamA | 회원1, 회원2

2. teamA | 회원1, 회원2

1과 2는 완전히 동일하지만 실제 데이터베이스가 보여주는 row가 2개이기 때문에 JPA도 어쩔 수 없이 동일한 객체 2개를 중복되게 만들 수밖에 없는 것이다. 

이를 해결하기 위한 방법으로 **distinct** 를 사용할 수가 있다.

```
// JPQL 
select distinct t from Team t join fetch t.members  where t.name = ‘팀A’
```

distinct를 사용하게 되면 SQL에는 distinct를 적용할 순 없다. 왜냐하면 SQL에서 distinct가 적용되기 위해서는 row의 모든 값이 동일해야되기 때문이다. 

그러나 JPQL에서 distinct는 한 가지 기능을 더 제공하는데 바로 중복된 엔티티를 제거해주는 것이다. 따라서 distinct를 사용하게 되면 객체가 다음과 같이 생성되게 된다.

1. teamA | 회원1, 회원2

distinct를 사용하게 되면 컬렉션 페치 조인을 할 때 중복을 제거하면서 join fetch 방식을 모두 사용할 수 있다는 장점이 있다.


<br>
<br>
참고: 인프런, 자바 ORM 표준 JPA 프로그래밍 - 기본편 _ 김영한