# CompletableFuture란?

`CompletableFuture` 에 대해서 알아보기 전에 우선 **비동기**에 대해서 간단하게 알아보고자 합니다. `CompletableFuture` 가 자바 비동기 프로그래밍에 쓰이는 클래스이기 때문이죠.

#### 비동기

비동기 처리는 간단하게 생각해보면 **특정 작업이 다른 작업과 독립적으로 동시에 실행될 수 있게 처리**하는 것을 의미합니다. 다른 작업의 완료 여부와 상관없이 특정 작업을 수행할 수 있게 되는 것이죠. 그렇다면 왜 이렇게 비동기적으로 작업을 처리하도록 하는 것일까요? 제일 중요한 이유는 **성능적인 이점**이 있기 때문입니다. 예를 들어서 AWS S3에 파일을 업로드 하는 것처럼 네트워크 I/O 시간이 오래 걸리는 요청을 처리해야 되는 경우 이 작업이 완료될 때까지 기다리는 것은 서버 입장에선 아쉬울 것입니다. 오래 걸리는 사진 업로드 요청은 일단 알아서 돌아가게 하고 그 밖에 다른 요청을 처리할 수 있다면 정말 좋겠죠. 이런 경우에 비동기적으로 프로그래밍 할 수 있습니다. 별도의 스레드에서 시간이 오래 걸리는 작업이 실행되도록 하고 다른 스레드에서는 또 다른 요청을 처리하게 하는 것입니다.

흔히 사용하는 동기적인 처리 방식과 비동기 처리 방식을 도식화해서 보면 다음과 같습니다.

![1_V5syja2casc0gCuu9zKV5g](https://github.com/user-attachments/assets/0468bc90-8775-4b3e-9d48-95a116002574)

그림에서 볼 수 있듯이 비동기적으로 처리하게 되면 동기적으로 처리했을 때보다 요청을 처리하는 시간이 훨씬 짧은 것을 확인할 수 있습니다. `CompletableFuture` 는 자바에서 이런 비동기적인 처리를 도와주는 클래스로서 Java8 에서 추가된 클래스입니다.

---

#### 등장 배경

사실 자바에서 비동기 처리를 위해 등장한 것은 `CompletableFuture` 가 처음이 아닙니다. 처음으로 등장한 것은 Java5 에서 처음 등장한 `Future`  입니다. 그렇다면 왜 `CompleatableFuture` 가 등장했을까요? `Future` 는 다음과 같은 한계점을 가지고 있었습니다.

- 여러 비동기 연산을 결합하기 어려움

- 비동기 연산 중 발생하는 예외 처리의 어려움

- 외부에서 완료 할 수 없음

  등등 ...



이런 한계점을 극복하기 위해 등장한 것이 `CompleatableFuture` 입니다. 이러한 등장배경에 따라 `CompletableFuture` 는 `Future` 인터페이스와 Java8 에서 추가된 `CompletionStage` 인터페이스를 구현하고 있습니다.

---

#### CompletableFuture 인스턴스 생성하기

`CompletableFuture` 는 클래스이기 때문에 사용하기 위해서는 인스턴스를 생성해줘야 합니다. 인스턴스를 생성하기 위해서는 `new` 키워드를 사용해서 객체를 생성할 수도 있지만 `supplyAsync()` 라는 static 메서드를 통해서도 생성할 수 있습니다. 

`new` 키워드로 생성하게 되면 비동기 작업이 자동으로 실행되지 않지만, `supplyAsync()` 를 사용해서 생성하게 되면 비동기 작업이 자동으로 실행되게 됩니다. 뿐만 아니라 `new` 키워드로 생성한 `CompletableFuture` 인스턴스는 수동으로 `complete` 을 사용해서 종료해줘야 하지만, `supplyAsync`() 를 사용해서 생성한 인스턴스는 그럴 필요가 없습니다. 

`supplyAsync()` 를 사용해서 `CompletableFuture` 를 생성하게 되면 `ForkJoinPool.commonPool()` 에서 새로운 스레드를 가져와서 해당 스레드에서 `supplyAsync()` 메서드의 인자로 넘겨준 함수형 인터페이스 `Supplier` 타입의 함수가 실행됩니다. 

```java
void supplyAsync() {
  String userId = "44492";
  CompletableFuture<String> userNameFuture = CompletableFuture.supplyAsync(() -> getUserName(userId));
  // ...
}

private String getUserName(String userId) {
  try {
    Thread.sleep(1000);
  } catch (Exception e) {
    // ...
  }
  return "Jack"
}
```

&nbsp;

# CompletableFuture의 다양한 사용법

#### 순차적으로 연산 처리하기

**thenApply**

`thenApply()` 메서드는 이전 단계의 결과값(반환되는 값을 의미합니다)을 다음 단계의 인수(파라미터)로 사용합니다. `thenApply()` 메서드는 함수형 인터페이스 `Function` 을 파라미터로 받습니다.

```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn) {
  return uniApplyStage(null, fn);
}
```

```java
void thenApply() {
  String userId = "44492";
  CompletableFuture<String> userNameFuture = CompletableFuture.supplyAsync(() -> getUserName(userId));
  CompletableFuture<Integer> userAgeFuture = userNameFuture.thenApply(s -> getUserAge(s));
  // ...
}

private String getUserName(String userId) {
  try {
    Thread.sleep(1000);
  } catch (Exception e) {
    // ...
  }
  return "Jack"
}

private int getUserAge(String username) {
  try {
    Thread.sleep(1000);
  } catch (Exception e) {
    // ...
  }
  return 30
}
```

<br>

**thenAccept**

`thenAccept()` 메서드는 함수형 인터페이스 `Consumer` 를 인자로 받고, `CompletableFuture<Void>` 를 반환합니다. 리턴 타입이 없는 경우에 사용합니다.

```java
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
   return uniAcceptStage(null, action);
}
```

```java
void thenAccept() {
  String userId = "44492";
  CompletableFuture<String> userNameFuture = CompletableFuture.supplyAsync(() -> getUserName(userId));
  CompletableFuture<Void> updateUserNameFuture = userNameFuture.thenAccept(s -> updateUserName(s));
  
  updateUserNameFuture.get(); // "Update name: Jack" 출력
}

private void updateUserName(String name) {
  System.out.println("Update name: " + name);
}
```

---

#### 연산 결합하기

**thenCompose**

`thenCompose()` 메서드를 사용하면 `CompletableFuture` 인스턴스를 결합해서 연산을 처리할 수 있습니다. `thenCompose()` 메서드는 두 개의 `Future` 를 **순차적으로 연결**한다는 특징을 가지고 있습니다. 이전 단계의 결과를 `get()` 없이 다음 `CompletableFuture` 인스턴스에서 사용할 수 있습니다. 

```java
public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn) {
  return uniComposeStage(null, fn);
}
```

```java
void thenCompose() {
  String userId = "44492";
  CompletableFuture<String> userInfoFuture = CompletableFuture.supplyAsync(() -> getUserName(userId))
    .thenCompose(s -> CompletableFuture.supplyAsync(() -> getUserAge(s)));
  // ...
}

private String getUserName(String userId) {
  try {
    Thread.sleep(1000);
  } catch (Exception e) {
    // ...
  }
  return "Jack"
}

private int getUserAge(String username) {
  try {
    Thread.sleep(1000);
  } catch (Exception e) {
    // ...
  }
  return 30
}
```

<br>

**thenCombine**

`thenCombine()` 메서드는 두 개의 독립적인 `Future` 를 처리하고 각각의 결과를 결합하여 추가적인 작업을 수행합니다. 

```java
public <U,V> CompletableFuture<V> thenCombine(
  CompletionStage<? extends U> other,
  BiFunction<? super T,? super U,? extends V> fn) {
  return biApplyStage(null, other, fn);
}
```

```java
void thenCombine() {
  String userId = "44492";
  String companyId = "15124123"
  CompletableFuture<String> userFuture = CompletableFuture.supplyAsync(() -> getUserName(userId))
        .thenCombine(CompletableFuture.supplyAsync(() -> getCompanyInfo(companyId)), (s1, s2) -> combineUserInfo(s1, s2));
}

private String getUserName(String userId) {
  try {
    Thread.sleep(1000);
  } catch (Exception e) {
    // ...
  }
  return "Jack"
}

private String getCompanyInfo(String comPanyId) {
  return "Naver"
}

private String combineUserInfo(String userName, String companyName) {
  return userName + "의 회사는 " + companyName + "입니다."; 
}
```

<br>

`thenApply()` 와 `thenCompose()` 의 차이에 대해서 정리해보자면 다음과 같습니다. `thenApply()` 는 이전 단계의 리턴값을 다음 단계의 인수로 사용하는 것이고, `thenCompose()` 는 이전 단계의 `CompletionStage` 자체를 인수로 사용하는 것입니다. `thenApply()` 와 `thenCompose()` 의 파라미터를 잘 확인해보면 좋을 것 같습니다. 

---

#### 병렬 처리

**allOf**

`allOf()` 메서드를 사용하면 여러 `Future` 를 병렬적으로 처리할 수 있습니다.  `allOf()` 메서드는 모든 `Future` 가 완료될 때까지 기다립니다. 모든 `Future`  가 완료될 때까지 기다리다가 완료되면 `CompletableFuture<Void>` 를 리턴합니다. 

```java
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs) {
  return andTree(cfs, 0, cfs.length - 1);
}
```

<br>

**anyOf**

`anyOf()` 메서드는 `allOf()` 메서드와 반대로 여러 `CompletableFuture` 중 완료된 것이 하나라도 있다면 바로 후속 작업을 진행하고 싶을 때 사용합니다. 

```java
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs) {
	int n; Object r;
  if ((n = cfs.length) <= 1)
      return (n == 0)
          ? new CompletableFuture<Object>()
          : uniCopyStage(cfs[0]);
  for (CompletableFuture<?> cf : cfs)
      if ((r = cf.result) != null)
          return new CompletableFuture<Object>(encodeRelay(r));
  cfs = cfs.clone();
  CompletableFuture<Object> d = new CompletableFuture<>();
  for (CompletableFuture<?> cf : cfs)
      cf.unipush(new AnyOf(d, cf, cfs));
  // If d was completed while we were adding completions, we should
  // clean the stack of any sources that may have had completions
  // pushed on their stack after d was completed.
  if (d.result != null)
      for (int i = 0, len = cfs.length; i < len; i++)
          if (cfs[i].result != null)
              for (i++; i < len; i++)
                  if (cfs[i].result == null)
                      cfs[i].cleanStack();
  return d;
}
```

---

#### 예외 처리

**handle**

`handle()` 메서드는 `CompletableFuture` 클래스를 사용해서 별도의 메서드에서 예외를 처리하는 것이 가능합니다. 두 가지 파라미터를 받을 수 있는데 연산이 성공적으로 끝난 경우의 결과와 연산이 실패했을 경우에 발생한 예외를 받을 수 있습니다.

```java
public <U> CompletableFuture<U> handle(BiFunction<? super T, Throwable, ? extends U> fn) {
  return uniHandleStage(null, fn);
}
```

```java
void handle() {
  String userId = "44492";
  // ...
    
  CompletableFuture<String> userInfoFuture = CompletableFuture.supplyAsync(() -> {
      if (userId == null) {
          throw new IllegalArgumentException("The userId must not be null!");
      }
      return getUserName(userId);
  }).handle((s, t) -> { 
      return t == null ? s : "Default Name";
  });
}

// 1. s: 연산이 성공한 경우에 결과가 담기는 부분
// 2. t: 연산이 실패한 경우에 발생한 예외가 담기는 부분

private String getUserName(String userId) {
  try {
    Thread.sleep(1000);
  } catch (Exception e) {
    // ...
  }
  return "Jack"
}
```

<br>

**completeExceptionally**

`completeExceptionally()` 메서드는 연산이 정상적으로 완료되지 않은 경우에 예외를 발생시키면서 비동기 연산을 종료합니다. 

```java
public boolean completeExceptionally(Throwable ex) {
    if (ex == null) throw new NullPointerException();
    boolean triggered = internalComplete(new AltResult(ex));
    postComplete();
    return triggered;
}
```

```java
void completeExceptionally() {
  String userId = "44492";
  CompletableFuture<String> userNameFuture = new CompletableFuture<>();

  if (userId == null) {
      userNameFuture.completeExceptionally(new IllegalArgumentException("The userId must not be null!"));
  }

  // do something..
}
```

---

**참고**

- [11번가 Tech Blog](https://11st-tech.github.io/2024/01/04/completablefuture/)
- [wbluke Blog](https://wbluke.tistory.com/50)
- 모던 자바 인 액션