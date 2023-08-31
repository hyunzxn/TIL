# Test Fixture Cleansing: deleteAll() vs deleteAllInBatch()

&nbsp;

# 1. 들어가며

테스트 코드를 작성할 때 통합테스트를 하다보면 데이터베이스를 사용할 때가 있습니다. 저의 경우에는 스프링을 사용하고 있고 `@SpringBootTest` 어노테이션을 사용해서 통합 테스트를 진행할 때 h2 인메모리 디비를 자주 사용하곤 합니다.

이렇게 디비를 사용하다보면 테스트 작성의 중요한 원칙 중 하나인 테스트 간 독립을 위해서 디비를 초기화 해주는 것이 대단히 중요합니다. 그렇게 하지 않으면 다른 테스트에 영향을 주면서 테스트가 의도치 않게 실패하는 경험을 자주 할 수 있습니다.

많은 분들이 디비를 초기화 할 때 제목에서 언급한 `deleteAll()` 또는 `deleteAllInBatch()` 두 가지 메서드를 주로 사용하실 것이라고 생각합니다. 둘은 이름은 비슷하지만 큰 차이가 있는데요, 이 차이로 인해 오늘 제가 겪은 에러에 대해서 정리해보려고 합니다.



&nbsp;


# 2. deleteAll() 과 deleteAllInBatch()의 차이점

## 2.1 deleteAll()

먼저 `deleteAll()` 부터 살펴보려고 합니다. `deleteAll()`은 `CrudRepository`인터페이스의 메서드입니다. `deleteAll()`의 구현 부분을 보면 다음과 같습니다.

```java
@Override
@Transactional
public void deleteAll() {

  for (T element : findAll()) {
    delete(element);
  }
}
```

우선 `deleteAll()`은 `findAll()`로 DB를 한 번 전체 조회하고 그것을 반복문에서 순회하면서 하나하나 데이터를 삭제하는 것을 확인할 수 있습니다.



예시를 보여드리기 위해 예시 코드를 보여드리면 다음과 같습니다.

```java
@Entity
public class Book {

  @Id
  @GeneratedValue(strategy = IDENTITY)
  private Long id;

  @Column(nullable = false)
  private String name;

  public Book() {

  }

  public Book(String name) {
    if (name.isBlank()) {
      throw new IllegalArgumentException("이름은 비어 있을 수 없습니다");
    }
    this.name = name;
  }

  public String getName() {
    return name;
  }

}
```

```java
@Entity
public class User {

  @Id
  @GeneratedValue(strategy = IDENTITY)
  private Long id;

  @Column(nullable = false)
  private String name;

  private Integer age;

  @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
  private final List<UserLoanHistory> userLoanHistories = new ArrayList<>();

  public User() {

  }

  public User(String name, Integer age) {
    if (name.isBlank()) {
      throw new IllegalArgumentException("이름은 비어 있을 수 없습니다");
    }
    this.name = name;
    this.age = age;
  }

  public void updateName(String name) {
    this.name = name;
  }

  public void loanBook(Book book) {
    this.userLoanHistories.add(new UserLoanHistory(this, book.getName(), false));
  }

  public void returnBook(String bookName) {
    UserLoanHistory targetHistory = this.userLoanHistories.stream()
        .filter(history -> history.getBookName().equals(bookName))
        .findFirst()
        .orElseThrow();
    targetHistory.doReturn();
  }

  @NotNull
  public String getName() {
    return name;
  }

  @Nullable
  public Integer getAge() {
    return age;
  }

  public Long getId() {
    return id;
  }

}
```

```java
@Entity
public class UserLoanHistory {

  @Id
  @GeneratedValue(strategy = IDENTITY)
  private Long id;

  @ManyToOne
  private User user;

  private String bookName;

  private boolean isReturn;

  public UserLoanHistory() {

  }

  public UserLoanHistory(User user, String bookName, boolean isReturn) {
    this.user = user;
    this.bookName = bookName;
    this.isReturn = isReturn;
  }

  @NotNull
  public String getBookName() {
    return this.bookName;
  }

  @NotNull
  public User getUser() {
    return user;
  }

  public boolean isReturn() {
    return isReturn;
  }

  public void doReturn() {
    this.isReturn = true;
  }

}
```

세 가지의 JPA 엔티티가 있습니다.

1. Book
2. User
3. UserLoanHistory



제가 테스트 하고자 하는 비즈니스 로직은 다음과 같습니다.

```java
@Transactional
public void returnBook(BookReturnRequest request) {
  User user = userRepository.findByName(request.getUserName()).orElseThrow(IllegalArgumentException::new);
  user.returnBook(request.getBookName());
}
```

유저를 조회하고 유저가 빌려간 책을 다시 반납하게 하는 로직입니다.



테스트코드는 다음과 같습니다.

```kotlin
@AfterEach
fun tearDown() {
  bookRepository.deleteAll()
  userRepository.deleteAll()
}

@Test
fun `책 반납이 성공한다`() {
  // given
  val savedUser = userRepository.save(User("홍길동", null))
  userLoanHistoryRepository.save(UserLoanHistory(savedUser, "이펙티브 코틀린", false))
  val request = BookReturnRequest("홍길동", "이펙티브 코틀린")

  // when
  bookService.returnBook(request)

  // then
  val results = userLoanHistoryRepository.findAll()
  assertThat(results).hasSize(1)
  assertThat(results[0].isReturn).isTrue()
}
```



> **[프로덕션 코드는 자바이고 테스트 코드는 코틀린인 이유]**
>
> 지금 제가 듣고 있는 강의가 자바 코드로 된 스프링 프로젝트를 코틀린 코드로 리팩토링 하는 강의이기 때문입니다. 



테스트를 돌리고 나면 어떤 쿼리가 나오는지 보여드리면 다음과 같습니다. (로직이 끝나고 `AfterEach` 부분을 보면 다음과 같습니다.)

```sql
Hibernate: 
    select
        book0_.id as id1_0_,
        book0_.name as name2_0_ 
    from
        book book0_
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.name as name3_1_ 
    from
        user user0_
Hibernate: 
    select
        userloanhi0_.user_id as user_id4_2_0_,
        userloanhi0_.id as id1_2_0_,
        userloanhi0_.id as id1_2_1_,
        userloanhi0_.book_name as book_nam2_2_1_,
        userloanhi0_.is_return as is_retur3_2_1_,
        userloanhi0_.user_id as user_id4_2_1_ 
    from
        user_loan_history userloanhi0_ 
    where
        userloanhi0_.user_id=?
Hibernate: 
    delete 
    from
        user_loan_history 
    where
        id=?
Hibernate: 
    delete 
    from
        user 
    where
        id=?

```

일단 여기까지 보고 `deleteAllInBatch()`를 살펴보고자 합니다.



&nbsp;



## 2.2 deleteAllInBatch()

`deleteAllInBatch()`는 `JpaRepository` 인터페이스의 메서드입니다. `deleteAllInBatch()`의 구현부분을 보면 다음과 같습니다.

```java
@Override
@Transactional
public void deleteAllInBatch() {
  em.createQuery(getDeleteAllQueryString()).executeUpdate();
}
```

`deleteAll()`과는 다르게 전체 DB를 조회하는 부분이 없고 한 번에 삭제하는걸 알 수 있습니다. `deleteAllInBatch()`를 사용한 코드는 다음과 같습니다.

```kotlin
@AfterEach
fun tearDown() {
  bookRepository.deleteAllInBatch()
  userLoanHistoryRepository.deleteAllInBatch()
  userRepository.deleteAllInBatch()
}

@Test
fun `책 반납이 성공한다`() {
  // given
  val savedUser = userRepository.save(User("홍길동", null))
  userLoanHistoryRepository.save(UserLoanHistory(savedUser, "이펙티브 코틀린", false))
  val request = BookReturnRequest("홍길동", "이펙티브 코틀린")

  // when
  bookService.returnBook(request)

  // then
  val results = userLoanHistoryRepository.findAll()
  assertThat(results).hasSize(1)
  assertThat(results[0].isReturn).isTrue()
}
```

쿼리를 보면 다음과 같습니다.

```sql
Hibernate: 
    delete 
    from
        book
Hibernate: 
    delete 
    from
        user_loan_history
Hibernate: 
    delete 
    from
        user
```

차이가 보이시나요? `deleteAll()`은 우선 book, user, userLoanHistory를 모두 전체 조회를 하고 그 이후에 그 전체 데이터 안에 있는 id를 하나하나 비교하면서 삭제를 진행하는 반면에 `deleteAllInBatch()`는 전체를 조회하지 않고 바로 테이블을 삭제하는 것을 확인할 수 있습니다.

이걸 통해 확인할 수 있는 것은 데이터가 적을 때는 큰 차이가 없지만 데이터가 많다면 `deleteAllInBatch()`를 사용하는 것이 더 속도 측면에서 좋을 것이라는 사실입니다. 데이터가 많다면 그 많은 데이터를 하나하나 다 돌면서 삭제하는 것은 시간이 분명히 오래 걸릴 것이기 때문입니다. 

다만 `deleteAllInBatch()`를 사용할 때는 주의할 점이 하나 있는데요, 다음 목차에서 알아보고자 합니다.



&nbsp;

# 3. deleteAllInBatch() 사용할 때 주의할 점

제가 겪은 에러가 이 부분을 놓쳐서 발생한 것인데요. 우선 에러 상황을 먼저 보여드리겠습니다.

```kotlin
@AfterEach
fun tearDown() {
  bookRepository.deleteAllInBatch()
  // userLoanHistoryRepository.deleteAllInBatch() 차이지점! 
  userRepository.deleteAllInBatch()
}

@Test
fun `책 반납이 성공한다`() {
  // given
  val savedUser = userRepository.save(User("홍길동", null))
  userLoanHistoryRepository.save(UserLoanHistory(savedUser, "이펙티브 코틀린", false))
  val request = BookReturnRequest("홍길동", "이펙티브 코틀린")

  // when
  bookService.returnBook(request)

  // then
  val results = userLoanHistoryRepository.findAll()
  assertThat(results).hasSize(1)
  assertThat(results[0].isReturn).isTrue()
}
```

`tearDown` 함수에서 처음에 제가 저지른 실수는 userLoanHistory 테이블을 비워주지 않은 것이었습니다. 그랬을 때 발생한 에러는 다음과 같습니다.



<img width="1709" height="150" alt="에러 메시지" src="https://github.com/hyunzxn/TIL/assets/100478841/f9298875-c5ea-4ff6-a6b6-ab0e0840b609">

`DataIntegrityViolationException`이 보이시나요? 네 그렇습니다. 외래키 제약 조건에 의해 데이터 무결성이 훼손됐다는 에러입니다. 왜 이런 에러가 발생한 것일까요?

## 3.1 왜 이런 에러가 발생했을까?

다시 엔티티 코드를 보면 이해가 바로 될 것입니다.

```java
@Entity
public class User {

  @Id
  @GeneratedValue(strategy = IDENTITY)
  private Long id;

  @Column(nullable = false)
  private String name;

  private Integer age;

  @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
  private final List<UserLoanHistory> userLoanHistories = new ArrayList<>();

  public User() {

  }

  public User(String name, Integer age) {
    if (name.isBlank()) {
      throw new IllegalArgumentException("이름은 비어 있을 수 없습니다");
    }
    this.name = name;
    this.age = age;
  }

  public void updateName(String name) {
    this.name = name;
  }

  public void loanBook(Book book) {
    this.userLoanHistories.add(new UserLoanHistory(this, book.getName(), false));
  }

  public void returnBook(String bookName) {
    UserLoanHistory targetHistory = this.userLoanHistories.stream()
        .filter(history -> history.getBookName().equals(bookName))
        .findFirst()
        .orElseThrow();
    targetHistory.doReturn();
  }

  @NotNull
  public String getName() {
    return name;
  }

  @Nullable
  public Integer getAge() {
    return age;
  }

  public Long getId() {
    return id;
  }

}
```

```java
@Entity
public class UserLoanHistory {

  @Id
  @GeneratedValue(strategy = IDENTITY)
  private Long id;

  @ManyToOne
  private User user;

  private String bookName;

  private boolean isReturn;

  public UserLoanHistory() {

  }

  public UserLoanHistory(User user, String bookName, boolean isReturn) {
    this.user = user;
    this.bookName = bookName;
    this.isReturn = isReturn;
  }

  @NotNull
  public String getBookName() {
    return this.bookName;
  }

  @NotNull
  public User getUser() {
    return user;
  }

  public boolean isReturn() {
    return isReturn;
  }

  public void doReturn() {
    this.isReturn = true;
  }

}
```

`User`와 `UserLoanHistory`는 일대다 관계를 맺고 있습니다. `User`가 1이고 `UserLoanHistory`가 N인 관계입니다. 그리고 `UserLoanHistory`가 외래키를 가지고 있는 구조입니다. 

제가 실수한 `tearDown` 함수를 보시면 user를 삭제해주는데 userLoanHistory는 삭제해주지 않는 것을 보실 수 있을 것입니다. 아직 외래키를 가진 userLoanHistory가 삭제되지 않았는데 user를 삭제하려고 하니 데이터 무결성이 훼손됐다는 에러를 보내준 것이죠.

그러면 왜 `deleteAll()`을 쓸 때는 이런 에러가 발생하지 않았는데 `deleteAllInBatch()`를 쓸 때만 이런 에러가 발생하는 것일까요? 이유는 먼저 설명드린 둘의 작동 방식 차이 때문입니다. `deleteAll()`이 우선 전체를 조회하고 하나하나 돌면서 삭제를 할 때 연관관계로 묶인 모든 엔티티를 검색해서 있다면 그걸 삭제해주는 반면에 `deleteAllInBatch()`는 한 번에 다 테이블을 날리려고 하기 때문에 아직 userLoanHistory를 삭제해주지 않은 상태에선 user를 삭제할 수 없기 때문이죠. 

따라서 `deleteAllInBatch()`를 쓸 때는 주의해서 사용해야합니다. 연관관계로 묶인 엔티티를 다 삭제해줘야 하고 삭제해줄 때 순서도 중요합니다. userLoanHistory도 비워주긴 하는데 user를 먼저 비워주고 userLoanHistory를 비우면 소용이 없겠죠? 제가 `deleteAllInBatch()`를 쓸 때 성공한 코드를 보면 userLoanHistoryRepository를 먼저 호출하는 이유가 이것 때문입니다.



&nbsp;

# 4. 마치며

통합테스트를 할 때는 DB를 사용하기 때문에 DB 관련 이슈가 항상 중요한 것 같습니다. 이런 이유 때문에 테스트에서 `@Transactional` 어노테이션을 사용하시는 분들도 있는 것 같습니다. `@Transactional` 어노테이션을 사용하게 되면 테스트가 끝나고 모두 롤백을 해주기 때문에 애초에 이런 걱정을 할 필요가 없기 때문이죠. 다만 `@Transactional` 어노테이션을 테스트에서 사용하게 되면 그에 따른 사이드 이펙트가 있을 수도 있기 때문에 주의해서 사용하는 것이 중요할 것 같습니다.

&nbsp;

[참고]

[실전! 코틀린과 스프링부트로 도서관리 애플리케이션 개발하기(Java 프로젝트 리팩토링)](https://www.inflearn.com/course/java-to-kotlin-2/dashboard)

[Practical Testing: 실용적인 테스트 가이드](https://www.inflearn.com/course/practical-testing-%EC%8B%A4%EC%9A%A9%EC%A0%81%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EA%B0%80%EC%9D%B4%EB%93%9C/dashboard)
