# 1. 들어가며

첫번째 [제네릭(Generic) 정리글](https://github.com/hyunzxn/TIL/blob/main/kotlin/generic.md)에서 제네릭이란 무엇이고 제네릭의 중요한 성질인 **변성**에 관해서 정리를 해봤습니다. 그런데 지난번 글에서는 함수의 파라미터에만 변성을 줬었는데요. 이번에 적을 글에서는 제네릭 클래스를 만들 때부터 변성을 주는 방법에 관해서 정리를 해보고자 합니다.

참고) 예시에 사용되는 코드들이 첫번째 정리글과 이어집니다.

&nbsp;

# 2. 선언 지점 변성 vs 사용 지점 변성

코틀린에서는 자바와는 다르게 클래스 자체에 변성을 줄 수 있습니다. 이렇게 만들면 특정 함수의 파라미터에 복잡하게 변성 어노테이션을 붙여주지 않아도 되기 때문에 함수 파라미터 타입을 작성할 때 복잡하지 않다는 장점이 있습니다. 

지난번과 이어서 `Box` 클래스를 만든다고 해보겠습니다.

```kotlin
class Box<T> {
  private val items: MutableList<T> = mutableListOf()
  
  fun getFirst(): T {
    return items.first()
  }
  
  fun getAll(): List<T> {
    return items
  }
}
```

현재 Box 클래스는 데이터를 **생산**하는 역할만을 담당하고 있습니다. 이런 경우에는 `out` 키워드를 붙여줘도 상관없습니다. 첫번째 제네릭 정리 글에서 `out` 키워드가 붙은 파라미터는 데이터를 생산하는 역할만을 담당할 수 있다고 정리한 바 있습니다. 클래스도 동일합니다. 클래스의 함수들이 데이터를 생산하는 역할만을 할 때는 클래스 자체를 **공변**하게 만들 수 있습니다. 따라서 이런 코드가 가능합니다.

```kotlin
class NewBox<out T> {
  private val items: MutableList<T> = mutableListOf()
  
  fun getFirst(): T {
    return items.first()
  }
  
  fun getAll(): List<T> {
    return items
  }
}
```

```kotlin
fun main() {
  val iphoneBox: NewBox<Iphone> = NewBox()
  val phoneBox: NewBox<Phone> = iphoneBox
}
```

&nbsp;

반대로 어떤 클래스가 데이터를 **소비**하는 함수만 있다면 클래스에 `in`키워드를 줌으로써 클래스 자체를 **반공변**하게 만들어 줄 수 있습니다. 

```kotlin
class NewBox2<in T> {
  private val items: MutableList<T> = mutableListOf()
  
  fun putFirst(item: T) {
    return items.add(item)
  }
  
  fun putAll(items: List<T>) {
    return items.addAll(items)
  }
}
```

&nbsp;

이렇게 클래스 선언 타입에 변성을 주는 것을 **선언 지점 변성**이라고 합니다. 앞서 언급한 바처럼 선언 지점 변성은 자바에서는 불가능하고 코틀린에서만 가능합니다. 

&nbsp;

# 3. 코틀린 표준 라이브러리로 살펴보는 선언지점 변성 및 @UnsafeVariance 어노테이션

코틀린 표준 라이브러리 중 데이터를 소비만 하는 인터페이스로 `Comparable` 인터페이스가 있습니다.

```kotlin
public interface Comparable<in T> {
  public operator fun compareTo(other: T): Int
}
```

클래스 선언 지점에 `in`키워드를 사용한 것을 확인할 수 있습니다.

&nbsp;

반대로 데이터를 생산만 하는 대표적인 인터페이스로는 `List`가 있습니다. 코틀린의 `List`는 불변컬렉션이기 때문에 자바의 `List`와는 다릅니다.

```kotlin
public interface List<out E> : Collection<E> {
  override val size: Int
  override fun isEmpty(): Boolean
  override fun contains(element: @UnsafeVariance E): Boolean
  override fun iterator(): Iterator<E>
  override fun containsAll(elements: Collection<@UnsafeVariance E>): Boolean
  
  public operator fun get(index: Int): E
   
  //... 생략
  
}
```

그런데 `List` 인터페이스는 100% 데이터를 생산만 하지 않습니다. 예시에 보여드린 `contains` 또는 `containsAll` 함수를 보면 데이터를 소비하는 역할을 하는 것을 알 수 있습니다. 

이런 경우를 위해 사용하는 어노테이션이 `@UnsafeVariance` 어노테이션입니다. 이 어노테이션은 안전하지 않은 변성을 감수하고 타입 어노테이션을 다른 곳에 사용할 수 있도록 도와주는 어노테이션입니다. 

&nbsp;



## 참고: 제네릭 함수 만드는 법!

제네릭 정리 첫번째 글과 지금 정리하는 글 모두 제네릭한 클래스를 만드는 것에 관하여 정리를 했는데요, 제네릭한 함수를 만들 수도 있습니다. 방법은 다음과 같습니다.

```kotlin
fun <T> getSize(array: Array<T>): Int {
  return array.size
}

fun main() {
  val intArray = arrayOf<Int>(1,2,3)
  val longArray = arraOf<Long>(1L, 2L, 3L)
  
  println(getSize(intArray))
  println(getSize(longArray))
}
```

제네릭한 함수를 만들기 위해서는 `fun` 키워드와 함수 이름 사이에 `<T>` 타입 파라미터를 작성해주면 됩니다. 

&nbsp;

# 4. 마치며

제네릭1, 2에 걸쳐 제네릭에 대한 기본 중의 기본에 대해서 정리를 해 보았습니다. 제네릭은 조금 어려운 개념이라 조금 더 공부를 많이 해야될 것 같다는 인상을 받았습니다. 관련된 내용은 추후에 다시 정리를 해서 글을 올려보도록 하겠습니다.