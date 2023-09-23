# 1. 들어가며

제네릭은 자바와 코틀린 뿐 아니라 많은 프로그래밍 언어에 있는 개념입니다. 하나의 제네릭 클래스를 만들어 두면 다양한 타입에 대해서 유연하게 사용할 수 있기 때문에 확장성 있는 코드를 설계할 때 많이 사용되는 개념인 것 같습니다.

사실 기초적인 프로젝트를 할 때는 제네릭을 사실 사용할 일이 많지 않은 것 같습니다. 저도 부트캠프에서 했던 프로젝트나 인턴 생활을 하면서 했던 프로젝트에서 제가 직접 제네릭 클래스를 설계하고 제네릭을 많이 사용한 경험은 없습니다. 

이번에 제가 제네릭에 대해서 공부해본 것은 제네릭을 당장은 사용할 일은 없지만 의외로 제네릭 개념이 들어간 코드를 읽을 일이 많기 때문입니다. 특히 자바나 코틀린에서 제공하는 공식 라이브러리 코드에는 제네릭이 굉장히 많이 사용됩니다. 제네릭에 대해서 잘 모르다보니 이런 코드를 읽을 때 이해하는 것이 힘들었습니다. 그래서 한 번은 제네릭에 대해서 공부를 해보자는 생각을 하게 됐습니다. 

이번에 공부를 해보니 제네릭을 마냥 어렵게 생각할 것은 아니라는 인상을 받기도 하였지만 아직 완전히 다 이해했다고는 할 수 없을 것 같습니다. 이번 글은 '나 제네릭 완전히 다 마스터했다!!' 라는 글이 아닙니다. 지금까지 공부한 것을 잊지 않기 위해 그리고 나중에 다시 공부할 때 참고하면 좋을 것 같아서 정리를 해보고자 하는 글입니다.

부족하거나 이상한 부분에 대해서 말씀해주시면 더 공부하고 보완해보도록 하겠습니다.

&nbsp;

# 2. 제네릭(Generic)이란?

#### 제네릭의 개념

제네릭을 가장 이해하기 쉽게 접근을 해보자면 **나중에 지정할 타입에 대해서도 동작하는 코드**라고 할 수 있습니다. 우선 제네릭에 대해서 살펴보기 전에 제네릭을 사용하지 않으면 어떤 불편을 겪는지에 대해 살펴보면 좋을 것 같습니다.

&nbsp;

#### 제네릭을 사용하지 않을 때 어떤 어려움이 있는가?

어떠한 물건이든 넣었다 뺄 수 있는 `Box` 클래스가 있다고 해보겠습니다.

```kotlin
class Box {
  private val items: MutableList<Item> = mutableListOf()
  
  fun getFirst(): Item {
    return items.first()
  }
  
  fun put(item: Item) {
    this.items.add(item)
  }
  
  fun getFrom(otherBox: Box) {
    this.items.addAll(otherBox.items)
  }
}
```

```kotlin
abstract class Item {
  val name: String
}

abstract class Phone(name: String) : Item(name)

// 아이폰
class Iphone(name: String) : Phone(name)

// 갤럭시
class Galaxy(name: String) : Phone(name)
```

&nbsp;

지금까지 클래스 관계를 도식화 해보면 다음과 같습니다.

<img width="830" alt="클래스 관계" src="https://github.com/hyunzxn/TIL/assets/100478841/0aea9906-fe88-4999-a59c-28e3e032416a">

&nbsp;

이제 `Box`에 아이폰을 넣어보겠습니다.

```kotlin
fun main() {
  val box = Box()
  box.put(Iphone("아이폰"))
  val iphone: Iphone = box.getFirst() // Error: Type Mismatch
}
```

박스를 만들고 거기에 아이폰을 넣은 후에 아이폰을 꺼내려고 하니까 에러가 발생합니다. 그런데 이것은 당연합니다. 지금 만들어 놓은 `Box`의 코드를 보면 `getFirst` 의 리턴타입은 Item 이기 때문에 바로 Iphone 타입의 객체를 반환할 수 없기 때문입니다. 

이것을 해결하기 위한 방법이 두 가지가 있습니다. 

1. 타입 캐스팅

```kotlin
fun main() {
  val box = Box()
  box.put(Iphone("아이폰"))
  val iphone: Iphone = box.getFirst() as Iphone
}
```

코틀린에서는 `as` 키워드로 타입 캐스팅을 할 수 있는데요, 이 코드는 얼핏 보면 간단하게 해결을 할 수 있는 것처럼 보이지만 굉장히 위험한 코드입니다. 가령 누가 이렇게 코드를 바꿔버려도 에러가 나지 않기 때문입니다. 이 코드는 런타임에서야 에러가 비로소 발견되기 때문에 좋지 못 한 코드입니다.

```kotlin
fun main() {
  val box = Box()
  box.put(Galaxy("갤럭시지롱"))
  val iphone: Iphone = box.getFirst() as Iphone
}
```

&nbsp;

2. 코틀린 Safe Type Casting & 엘비스 연산자

```kotlin
fun main() {
  val box = Box()
  box.put(Iphone("아이폰"))
  val iphone: Iphone = box.getFirst() as? Iphone ?: throw IllegalArgumentException()
}
```

타입 캐스팅을 할 때  `as?` 이렇게 해주게 되면 잘못된 타입 캐스팅을 하게 되면 null이 되면서 엘비스 연산자 뒤의 에러가 발생하게 됩니다. 이것은 런타임 전에 에러를 내지 않는다는 장점이 있긴 하지만 결국 에러가 발생한다는 점에서 궁극적인 해결책이 될 수 없습니다. 

&nbsp;

이런 문제를 해결하기 위해 제네릭을 활용할 수 있습니다. 제네릭을 맨 처음 설명할 때 **나중에 지정할 타입에 대해서도 동작하는 코드**라고 한 바 있는데요. 지금 예시에 대입해보면 Iphone이 올 지 Galaxy가 올 지 알 수 없는 상황에서 아이폰인지 갤럭시인지는 나중에 지정하고 싶은데 일단 지금 두 가지 모두에 활용할 수 있는 코드를 작성하고 싶을 때 사용하면 되는 것입니다.

&nbsp;

그렇다면 이제 제네릭 클래스를 한 번 만들어 보겠습니다.

```kotlin
class GenericBox<T> {
  private val items: MutableList<T> = mutableListOf()
  
  fun getFirst(): T {
    return items.first()
  }
  
  fun put(item: T) {
    this.items.add(item)
  }
  
  fun getFrom(otherBox: GenericBox<T>) {
    this.items.addAll(otherBox.items)
  }
}
```

코드에서 보이는 것처럼 타입 파라미터를 사용한 클래스를 제네릭 클래스라고 합니다. `<>` 안에 들어간 `T` 를 타입 파라미터라고 부릅니다. 제네릭 클래스를 만들며 기존 코드와 가장 차이가 나는 지점은 클래스의 함수들도 타입 T를 가지게 됐다는 것입니다. 

이렇게 함으로써 GenericBox를 인스턴스화 할 때 타입 정보를 넘겨주게 된다면 그 타입 정보가 모든 함수에게도 동일하게 적용이 된다는 장점을 가지게 됩니다. 제네릭 클래스를 활용해서 다시 `Box` 에 아이폰을 넣고 아이폰을 꺼내보겠습니다.

```kotlin
fun main() {
  val box = GenericBox<Iphone>()
  box.put(Iphone("아이폰"))
  val iphone: Iphone = box.getFist() // GenericBox<Iphone>에는 Iphone만 들어있기 때문에 타입 캐스팅을 해주지 않아도 OK!
}
```

&nbsp;

# 3. 공변성

이제 우리는 제네릭 클래스를 만들 수 있게 되었습니다. 이제 새로운 요구사항을 처리해보도록 하겠습니다. 

> 새로운 요구사항: 아이폰 박스에 아이폰을 하나 넣고 Phone 박스에 getFrom 함수를 사용해 아이폰을 옮겨야 한다.

코드를 짜보면 다음과 같습니다.

```kotlin
fun main() {
  val iphoneBox = GenericBox<Iphone>()
  iphoneBox.put(Iphone("아이폰"))
  
  val phoneBox = GenericBox<Phone>()
  phoneBox.getFrom(iphoneBox) // Error: Type Mismatch
}
```

iphoneBox에 들어있는 아이폰들은 모두 Phone을 상속하는 객체들이기 때문에 phoneBox에 들어가는 것이 크게 부자연스럽지 않습니다. 그런데 실제로는 Type Mismatch 에러가 나면서 컴파일이 되지 않습니다. 이렇게 되는 이유를 이해하기 위해서는 공변성에 대해서 알아야 합니다.

&nbsp;

#### 변성

변성에 대해 알아보기 전에 **상속관계**에 대해 잠깐 언급하고 넘어가고자 합니다. 상속관계는 객체지향적 프로그래밍에서 많이 사용하는 개념인데요. 예를 들어서 알아보면 좋을 것 같습니다.

```kotlin
fun doSomething(num: Number) {
  // ...
}

val a: Int = 5
doSomething(a) // Number 타입인 파라미터에 Int 타입이 들어갈 수 있다!
```

Int는 Number의 하위 타입이기 때문에 이 코드는 에러 없이 정상적으로 동작합니다. 상속관계를 풀어서 말하면 **상위 타입의 자리에 하위 타입이 들어갈 수 있다!** 라고 정리할 수 있을 것 같습니다.

(김영한 강사님께서는 상속에 대해서 설명해주실 때 상위타입을 부모, 하위타입을 자녀라고 설명해주시면서 부모는 자녀를 품을 수 있지만 자녀는 부모를 품을 수 없다라고 설명해주신 바가 있는데 상속관계에 대한 가장 이해하기 쉬운 비유인 것 같습니다.)

이제 다시 새로운 요구 사항을 구현한 코드를 살펴보도록 하겠습니다,

```kotlin
fun main() {
  val iphoneBox = GenericBox<Iphone>()
  iphoneBox.put(Iphone("아이폰"))
  
  val phoneBox = GenericBox<Phone>()
  phoneBox.getFrom(iphoneBox) // Error: Type Mismatch
}
```

현재 phoneBox의 타입은 `GenericBox<Phone>` 이기 때문에 getFrom 함수도 `getFrom(otherBox: GenericBox<Phone>)` 이렇게 될 것입니다. 반면에 지금 파라미터로 넣어주려고 하는 iphoneBox는 `GenericBox<Iphone>` 타입을 가지고 있습니다.

따라서 Phone과 Iphone은 상속관계이지만 **`Generic<Phone>`과 `Generic<Iphone>`은 아무런 관계도 아니라는 것**을 유추할 수 있습니다. 이것을 가리켜 **GenericBox는 무공변**하다 라고 합니다. 이것에 대해서 좀 더 자세히 살펴보도록 하겠습니다.

&nbsp;

#### 공변이란?

이것을 좀 더 자세히 보기 위해 자바의 배열(Array)과 리스트(List)를 비교해보도록 하겠습니다. 둘의 가장 큰 차이점은 배열은 공변하지만 리스트는 무공변하다는 것입니다.

가령 자바의 배열에서는 A 객체가 B 객체의 하위 타입일 때 A 배열 역시 B 배열의 하위 타입으로 인식됩니다. `Array<String>`은 `Array<Object>`의 하위타입이 되는 것입니다.

따라서 이런 코드가 가능합니다.

```java
public static void main(String[] args) throws IOException {
  String[] strs = new String[]{"A", "B", "C"};
  // Object가 String의 상위타입이므로 Object[]가 String[]의 상위타입으로 간주된다.
  // 따라서 objs에 strs을 대입할 수 있다.
  Object[] objs = strs;
  objs[0] = 1; // java.lang.ArrayStoreException: java.lang.Integer
}
```

코드를 보면 문제가 하나 있습니다. `objs`는 `strs`이기 때문에 실제 타입은 `String[]`인데 type이 `Object[]`이기 때문에 숫자를 넣을 수 있고 이것은 런타임 때 오류를 뱉어내기 때문에 애플리케이션에 치명적인 문제가 될 수 있습니다. 반면에 리스트는 좀 다릅니다.

```java
public static void main(String[] args) throws IOException {
  List<String> strs = List.of("A", "B", "C");
  List<Object> objs = strs; // Type Mismatch
}
```

리스트는 불공변하기 때문에 `List<String>`과 `List<Object>`는 아무런 관계가 없기 때문에 `strs`가 `objs`에 들어갈 수 없습니다. 이러한 점 때문에 배열을 사용하는 것보다 리스트를 사용하는 것이 타입 safe한 측면에서 좋습니다.

&nbsp;

제네릭 클래스도 무공변하기 때문에 `Generic<Phone>`과 `Generic<Iphone>` 은 아무런 관계가 아니게 되고 따라서 지금 처리해야될 요구사항이 정상적으로 수행되지 않는다는 것을 이제는 이해할 수 있습니다. 결과적으로 그러면 `GenericBox<T>` 를 공변하게 만들어주면 모든 문제가 해결될 수 있습니다.

&nbsp;

#### 제네릭 클래스를 공변하게 만드는 법

제네릭 클래스를 공변하게 만드는 것은 생각보다 쉽습니다. 함수에 있는 타입 파라미터에 `out` 키워드를 붙여주면 됩니다. 이 때 `out`  키워드를 **변성 어노테이션**이라고 합니다.

```kotlin
fun getFrom(otherBox: GenericBox<out T>) {
  this.items.addAll(otherBox.items)
}
```

이렇게 바꾸고 나서 다시 코드를 작성해보면 이제는 에러가 나지 않습니다.

```kotlin
fun main() {
  val iphoneBox = GenericBox<Iphone>()
  iphoneBox.put(Iphone("아이폰"))
  
  val phoneBox = GenericBox<Phone>()
  phoneBox.getFrom(iphoneBox) // 클래스 간의 상속관계가 제네릭 클래스 간의 관계로까지 확장됨(상위 타입에 하위 타입 들어가게 됨!)
}
```



&nbsp;

#### 변성과 데이터의 생산•소비 상관관계

`out` 키워드의 의미를 좀 더 자세히 살펴보면 다음과 같습니다. `out` 키워드가 붙은 파라미터는 오직 생산자의 역할만을 할 수 있습니다. 여기서 생산자의 역할만을 할 수 있다는 것은 데이터를 꺼낼 수만 있다는 것입니다. 좀 더 쉽게 말하자면 **데이터를 return 할 수만 있다는 것**입니다.  만약 데이터를 소비하게 하려면(데이터를 추가적으로 넣을 수 있게 한다면) 에러가 발생하게 됩니다. 코드로 좀 더 살펴보도록 하겠습니다.

```kotlin
fun getFrom(otherBox: GenericBox<out T) {
  otherBox.put(this.getFirst()) // 에러 발생!!!
  this.items.addAll(otherBox.items)
}
```

왜 데이터를 집어넣으려고 하면 에러가 발생할까요? 그 이유는 타입 안정성에 있습니다. 다음과 같이 한 번 가정을 해보겠습니다.

```kotlin
fun getFrom(otherBox: GenericBox<out T) {
  otherBox.put(this.getFirst()) // 에러가 나지 않는다고 가정!!
  this.items.addAll(otherBox.items)
}
```

```kotlin
fun main() {
  val iphoneBox = GenericBox<Iphone>()
  iphoneBox.put(Iphone("아이폰"))
  
  val phoneBox = GenericBox<Phone>()
  phoneBox.put(Galaxy("갤럭시"))
  phoneBox.getFrom(iphoneBox)
}
```

이렇게 되면 `getFrom` 함수의 파라미터인 `otherBox`의 타입이 `GenericBox<Iphone>`인 상황이 됩니다. 근데 `getFrom` 함수에 의해 Galaxy를 넣어야 합니다. 이것은 타입 안정성에 위배가 됩니다. 다시 단계적으로 살펴보면 이렇습니다.

1. 아이폰 박스를 만들고 아이폰 박스에 아이폰을 넣는다.
2. 폰박스를 만들고 폰박스에 갤럭시를 넣는다.
3. getFrom 함수 실행 (이 때 otherBox의 타입은 `GenericBox<Iphone>`)
4. getFrom 함수를 실행할 때 `otherBox.put(this.getFirst())` 코드가 실행이 되어야 한다.
5. 여기서의 `this`는 PhoneBox를 의미한다. 그리고 2번에서 갤럭시를 넣었기 때문에 `this.getFirst()` 는 `Galaxy()` 를 의미한다. 
6. 그러면 otherBox(=> 아이폰 박스)에 갤럭시를 넣어야 하는데 이건 말이 되지 않는다.

이런 흐름으로 인해서 공변한 제네릭 클래스는 오직 데이터를 생산하는 것만 가능하게 되는 것입니다.

&nbsp;

그러면 이제 반대의 경우도 생각해볼 수 있습니다. 새로운 코드를 추가해보도록 하겠습니다.

```kotlin
fun moveTo(otherBox: GenericBox<T>) {
  otherBox.items.addAll(this.items)
}
```

새로운 moveTo 함수는 현재 자신이 가지고 있는 items를 다른 박스로 옮기는 역할을 수행합니다. 이것을 구현해보도록 하겠습니다.

```kotlin
fun main() {
  val phoneBox = GenericBox<Phone>()
  
  val iphoneBox = GenericBox<Iphone>()
  iphoneBox.put(Iphone("아이폰"))
  iphoneBox.moveTo(phoneBox) // Type Mismatch
}
```

이 코드도 얼핏 보기에는 문제가 없어 보입니다. 아이폰 박스에서 아이폰을 꺼내서 폰박스에 담는 것이기 때문입니다. 그러나 공변 관계에 대해서 살펴볼 때 언급한 것처럼 `GenericBox<Phone>` 과 `GenericBox<Iphone>`은 무공변하기 때문에 에러가 발생하게 됩니다.

따라서 두 제네릭 클래스 간의 관계를 만들어 주면 해결이 될 것 같습니다. 그런데 이번에는 `out` 키워드로는 해결이 되지 않습니다. 왜냐하면 `GenericBox<Iphone>` 자리에 `GenericBox<Phone>`이 들어가야 하기 때문입니다.

`Iphone`클래스는 `Phone`클래스를 상속합니다. 그리고 `out` 키워드는 이런 상속관계를 제네릭 클래스로 그대로 동일하게 적용하게 해줍니다. 그러나 지금 `moveTo` 함수를 보면 `GenericBox<Iphone>` 자리에 `GenericBox<Phone>`이 들어가야하는데 **이건 클래스 간 상속관계가 제네릭 클래스 간의 관계에선 역전되어야 함을 의미합니다**.

이럴 때 사용할 수 있는 키워드가 `in` 키워드 입니다. `in` 키워드를 사용하게 되면 클래스 간의 상속관계가 제네릭 클래스에서는 반대로 역전됩니다. 이런 관계를 **반공변**하다고 합니다. `in` 키워드가 붙은 함수의 파라미터는 공변한 파라미터와 반대로 데이터를 생산하는 것은 불가능하고 소비하는 것만 가능하게 됩니다.

&nbsp;

지금까지의 내용을 정리해보면 다음과 같습니다.

- 클래스 간의 상속관계가 제네릭 클래스 간의 관계로 확장되지 않는다. (무공변)
- 따라서 클래스 간의 상속관계를 확장해서 제네릭 클래스에서 사용하고 싶다면 변성을 만들어 줘야 한다.
- 클래스 간의 상속관계를 그대로 이어서 사용하려면 `out` 키워드를 사용해서 공변하게 만들어줘야 한다. `out` 키워드가 붙은 파라미터는 오직 생산자의 역할만 할 수 있다.
- 클래스 간의 상속관계를 역전해서 사용하려면 `in`  키워드를 사용해서 반공변하게 만들어줘야 한다. `in` 키워드가 붙은 파라미터는 오직 소비자의 역할만 할 수 있다. 

&nbsp;

# 4. 마치며

제네릭에 관해서 가장 기본적인 것만 정리를 해보았습니다. 사실 지금 정리한 건 함수의 파라미터에 제네릭 타입을 넣어준 경우에 한정적입니다. 함수 자체를 제네릭하게 만드는 법, 클래스 자체를 제네릭하게 만드는 법 등에 대해서는 다루지 않았습니다. 이런 부분에 대해서는 다음 글에 이어서 작성을 해보도록 하겠습니다.
