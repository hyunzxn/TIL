# 1. 들어가며

코틀린에는 영역 함수(Scope Function)이라는 것이 있습니다. 영역 함수는 **객체의 이름을 사용하지 않아도 그 객체에 접근할 수 있는 임시 영역을 만들어주는 함수**입니다. 정의가 거창해보이지만 사실 영역함수의 본질적인 기능은 코드를 간결하게 하고 가독성을 높이는데 초점이 맞춰져 있습니다. 

영역 함수는 5가지 종류가 있습니다. 각각의 영역 함수는 공통점과 차이점으로 다시 그룹핑을 할 수 있습니다. 이번 글에서는 5가지 영역 함수의 공통점 및 차이점을 중점으로 정리를 해보고 영역 함수에 대한 주의점 등을 정리해보고자 합니다.

&nbsp;

# 2. 영역 함수

영역 함수는 앞서 5가지 종류가 있습니다.

>  영역 함수 종류: `let`, `run`, `apply`, `also`, `with`

이 중 `with` 을 제외한 나머지 4개는 그룹핑이 가능한데 이제 그것을 정리해보고자 합니다. 우선 예시에서 계속 사용할 클래스를 하나 만들어 보겠습니다.

```kotlin
class Person(
  var age: Int,
) {
  fun increaseAge() {
    this.age += 1
  }
}
```

&nbsp;

## 2.1 결과 반환 차이로 Grouping 해보기

### 2.1.1 람다의 결과를 반환하는 let 과 run

우선 `let` 과 `run` 다음과 같이 사용합니다.

```kotlin
val person: Person = Person(30)

val value1 = person.let {
  it.age
}

val value2 = person.run {
  this.age
}
```

영역 함수를 사용할 때는 `람다`를 사용합니다. `let` 과 `run` 은 람다의 결과를 반환합니다. 따라서 `value1`과 `value2` 에는 `Person` 객체인 `person` 의 age의 값인 30이 들어가게 됩니다. 

&nbsp;

### 2.1.2 호출하는 객체를 반환하는 also 와 apply

마찬가지로 `also` 와 `apply` 는 다음과 같이 사용합니다. 

```kotlin
val person: Person = Person(30)

val value3 = person.also {
  it.increaseAge()
}

val value4 = person.apply {
  this.increaseAge()
}

```

`also` 와 `apply` 는 호출하는 객체를 반환합니다. 이 말의 의미는 `value3` 와 `value4` 에는 `Person` 의 객체인 `person` 이 바로 저장된다는 것입니다. `also` 와 `apply` 람다에서 `increaseAge` 함수를 호출해줬으므로 `person` 의 age의 값은 31이 되어있을 것입니다. 

&nbsp;

## 2.2 객체에 접근하는 지시어 차이로 Grouping 해보기

### 2.2.1 it 사용

첫번째 Grouping에서 눈치 채신 분들도 계시겠지만 객체에 접근하는 지시어로 `it` 과 `this` 두 가지를 사용합니다. `let` 과 `also` 는 `it` 을 사용합니다.

```kotlin
val person: Person = Person(30)

val value1 = person.let {
  it.age
}

val value3 = person.also {
  it.increaseAge()
}
```

&nbsp;

### 2.2.2 this 사용

`run` 과 `apply` 는 `this` 를 사용합니다.

```kotlin
val person: Person = Person(30)

val value2 = person.run {
  this.age
}

val value4 = person.apply {
  this.increaseAge()
}
```

&nbsp;

# 3. 왜 이런 차이가 있는 것인가?

위에서 정리한 바와 같이 4개의 영역 함수는 반환 결과 및 객체 지시어에서 공통점 및 차이점을 보입니다. 그렇다면 왜 이런 결과가 발생하는걸까요? 

```kotlin
// let: it 사용
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}

// also: it 사용
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}

// run: this 사용
public inline fun <T, R> T.run(block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}

// apply: this 사용
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

영역 함수가 코틀린 라이브러리에 정의된 모습들입니다. 여기 모습들을 보면서 차이점을 알아보도록 하겠습니다.

`let` 과 `also` 의 경우에는 파라미터가 되는 `block` 이 일반 함수입니다. 반면에 `run` 과 `apply` 는 `block` 이 확장함수입니다. 확장 함수를 공부해보신 분들은 아시겠지만 확장 함수는 `수신객체타입` 과 `수신객체` 라는 개념을 가지고 있습니다. 그리고 수신 객체는 `this` 라는 키워드로 접근할 수 있습니다. 

이러한 차이 때문에 확장함수를 파라미터로 받는 `run` 과 `apply` 만 객체에 접근할 때 `this`를 사용하게 되는 것입니다. 



&nbsp;

## 3.1 주의점

`it` 과 `this` 를 사용하는 차이점이 발생하는 이유를 알아보았는데 주의할 점이 있습니다. 

`this`  키워드는 생략이 가능하지만 다른 것으로 대체가 불가능합니다. 반대로 `it` 은 생략은 불가능하지만 다른 것으로 대체가 가능합니다.

```kotlin
val person: Person = Person(30)

val value3 = person.also {
  age.increaseAge() // it 대신에 age를 쓰는 것 가능!
}

val value4 = person.apply {
  increaseAge() // this 생략 가능!
}

```

&nbsp;

# 4. 영역 함수 with

영역 함수의 종류는 총 5가지인데 위에서는 `with` 을 제외하고 4가지만 다뤘습니다. `with` 는 성질이 조금 다르기 때문입니다. 나머지 4개의 영역 함수는 모두 확장 함수이지만 `with` 는 그렇지 않습니다. 

```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```

나머지 4개의 영역 함수와 달리 `with` 는 파라미터를 2개 받습니다. `with` 를 사용하는 방법은 다음과 같습니다. 

```kotlin
val person: Person = Person(30)

with(person) {
  println(this.age)
}
```

`with` 도 `run` 과 `apply`  와 마찬가지로 파라미터로 받는 함수의 모양이 확장 함수이기 때문에 `this`  키워드로 수신객체에 접근할 수 있습니다. 따라서 `this` 키워드를 생략하는 것 역시 가능합니다. 

&nbsp;

지금까지 정리한 것들을 간략하게 정리해보면 다음과 같습니다.

|   구분    | 람다 결과 반환 | 객체 그 자체 반환 |
| :-------: | :------------: | :---------------: |
|  it 사용  |      let       |       also        |
| this 사용 |      run       |       apply       |

&nbsp;

# 5. 마치며

지금까지 영역 함수가 무엇이고 각각이 가지는 특징에 대해서 간단하게 정리해봤습니다. 맨 처음에 영역 함수는 코드를 간결하게 가독성을 높이는데 초점이 맞춰져 있다고 언급을 한 바 있는데요. 과연 그러면 영역 함수를 사용하는 것이 무조건 좋은 코드일까요? 

영역 함수를 사용하면 보다 더 함수형 프로그래밍스럽게 코드를 작성하는 것이 가능한 것 같습니다. 그래서인지 코드를 더 간결하게 작성하는 것도 가능한 것 같습니다(Method Chaining 을 사용할 수 있으니까요). 그렇지만 이렇게 작성된 코드가 무조건 더 좋은 코드인 지는 생각을 해봐야 할 것 같습니다.

우선 과도한 영역 함수 사용은 코틀린에 익숙하지 않은 개발자에게는 오히려 가독성에 도움이 되지 않습니다. 그리고 작성한 개발자조차 시간이 지나서 다시 코드를 보게 되면 코드의 의미를 파악하는데 시간이 더 걸릴 수도 있습니다.

따라서 영역 함수를 무분별하게 사용하기보다는 필요에 따라서 최적화하여 사용하는 것이 좋은 코드를 작성하는 것이라고 생각합니다.