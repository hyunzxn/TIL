# 1. 들어가며

평소 자바와 스프링을 사용하며 람다식을 조금 더 간단하게 표현하는 방법인 **메서드 레퍼런스**를 사용할 일이 많았습니다. 그렇지만 제대로 알고 사용한다기 보다는 IDE에서 추천해주는걸 그대로 사용했던 것 같습니다.

이번 기회에 메서드 레퍼런스에 대해서 확실하게 공부해보고 그걸 정리하고자 합니다.  

# 2. 메서드 레퍼런스(Method Reference) 사용법

## 2.1 메서드 레퍼런스란?

메서드 레퍼런스의 사용법을 살펴보기 앞서 메서드 레퍼런스가 무엇인지에 대해 알아보고자 합니다.

메서드 레퍼런스란 자바8부터 추가된 람다식을 좀 더 간단하게 표현하는 방식을 의미합니다. 람다식이 `() -> { }` 이런 모양을 하고 있는 반면에 메서드 레퍼런스는 `A::B` 이런 식으로 콜론을 두 개 연달아 쓰는 모양을 띄고 있습니다.

예시 코드를 보면 다음과 같습니다.

```java
Consumer<String> fun = str -> System.out.println(str);
fun.accept("Hello");

//결과: Hello
```

위에 나온 람다식을 메서드 레퍼런스로 표현하면 다음과 같습니다.

```java
Consumer<String> fun = System.out::println;
fun.accept("Hello")

//결과: Hello
```

예시에서도 알 수 있듯이 메서드 레퍼런스는 `Classname::MethodName` 형태로 사용합니다. 메서드 레퍼런스는 소괄호를 전혀 사용하지 않기 때문에 들어가는 파라미터와 리턴 타입 등에 대해서 잘 알고 사용하는 것이 중요합니다.

## 2.2 메서드 레퍼런스 사용 패턴

메서드 레퍼런스를 사용하는 패턴은 세 가지가 있습니다.

1. Static 메서드 레퍼런스
2. Instance 메서드 레퍼런스
3. Constructor 메서드 레퍼런스

이제 각각에 대해서 정리해보고자 합니다.



**Static 메서드 레퍼런스**

Static 메서드 레퍼런스는 이름처럼 Static 메서드를 메서드 레퍼런스 방식으로 사용하는 것입니다. 예시 코드는 다음과 같습니다.

```java
public class StaticMethodReferenceExample {

  public static void main(String[] args) {
    Movable move1 = distance -> Car.moving(distance);
    Movable move2 = Car::moving; // 메서드 레퍼런스 사용!
    move1.move(10);
    move2.move(20);
  }

  interface Movable {
		void move(int distance);
  }

  public static class Car {
  	static void moving(int distance) {
    	System.out.println("움직인 거리는 " + distance + " 입니다.");
  	}
	}

}

// 결과
// 움직인 거리는 10 입니다.
// 움직인 거리는 20 입니다.
```

이건 예시를 위해 직접적으로 `Movable` 인터페이스를 만들었지만 자바가 제공하는 함수형 인터페이스인 `Consumer`를 이용할 수도 있습니다.

```java
public class StaticMethodReferenceExample {

  public static void main(String[] args) {
    Consumer<Integer> move1 = distance -> Car.moving(distance);
    Consumer<Integer> move2 = Car::moving // 메서드 레퍼런스 적용!
    move1.move(10);
    move2.move(20);
  }

  public static class Car {
  	static void moving(int distance) {
      System.out.println("움직인 거리는 " + distance + " 입니다.");
    }
	}

}

// 결과
// 움직인 거리는 10 입니다.
// 움직인 거리는 20 입니다.
```

결과는 동일하게 나옵니다.



**Instance 메서드 레퍼런스**

Instance 메서드 레퍼런스는 객체의 멤버 메서드를 메서드 레퍼런스로 사용하는 것을 의미합니다. 예시 코드는 다음과 같습니다.

```java
public class InstanceMethodReferenceExample {

  public static void main(String[] args) {
    List<Country> countries = Arrays.asList(
    	new Country("한국"),
      new Country("미국"),
      new Country("일본")
    );
    countries.stream()
      .forEach(country -> country.printName());
  }

  public static class Country {
    String name;

    public Country(String name) {
      this.name = name;
    }

    public void printName() {
      System.out.println(name);
    }
  }
}

// 결과
// 한국
// 미국
// 일본
```

이걸 메서드 레퍼런스를 사용하면 다음과 같이 바꿀 수 있습니다.

```java
public class InstanceMethodReferenceExample {

  public static void main(String[] args) {
    List<Country> countries = Arrays.asList(
    	new Country("한국"),
      new Country("미국"),
      new Country("일본")
    );
    countries.stream()
      .forEach(Country::printName); // 메서드 레퍼런스 적용!
  }

  public static class Country {
    String name;

    public Country(String name) {
      this.name = name;
    }

    public void printName() {
      System.out.println(name);
    }
  }
}

// 결과
// 한국
// 미국
// 일본
```



**Constructor 메서드 레퍼런스**

Constructor 메서드 레퍼런스는 생성자를 생성해주는 메서 레퍼런스 방식입니다. 예시 코드는 다음과 같습니다.

```java
public class ConstructorMethodReferenceExample {
  
  public static void main(String[] args) {
    List<String> countries = Arrays.asList("한국", "미국", "일본");
    countries.stream()
      .map(name -> new Country(name))
      .forEach(country -> country.printName());
  }
  
  public static class Country {
    String name;

    public Country(String name) {
      this.name = name;
    }

    public void printName() {
      System.out.println(name);
    }
  }
}

// 결과
// 한국
// 미국
// 일본
```

이 코드를 메서드 레퍼런스를 사용하면 아래와 같이 변경할 수 있습니다.

```java
public class ConstructorMethodReferenceExample {
  
  public static void main(String[] args) {
    List<String> countries = Arrays.asList("한국", "미국", "일본");
    countries.stream()
      .map(Country::new) // 메서드 레퍼런스 적용!
      .forEach(Country::printName); // 메서드 레퍼런스 적용!
  }
  
  public static class Country {
    String name;

    public Country(String name) {
      this.name = name;
    }

    public void printName() {
      System.out.println(name);
    }
  }
}

// 결과
// 한국
// 미국
// 일본
```


## 3. 마치며

좋은 IDE를 사용하면 IDE가 코드를 자동적으로 개선시켜주는 편리함이 상당합니다. 특히 자바나 코틀린을 사용하는 개발자들이라면 인텔리제이의 편리함을 모두 알고 계시리라 생각합니다. IDE가 추천해주는 걸 편리하게 사용하는 것도 좋지만, 왜 그렇게 추천해주는지를 알고 쓰는 것이야 말로 정말 중요한 것이 아닐까요? 

아직 함수형 프로그래밍에 익숙치 않아서 람다, 함수형 인터페이스, 메서드 레퍼런스 등을 사용하는 것이 익숙하지는 않습니다. 파라미터가 뭐지? 하면서 다시 한 번 코드를 천천히 봐야 하는 미숙함이 있지만 점차 익숙해질 것을 기대해봅니다.



**<참고>**

제가 참고한 블로그는 다음과 같습니다.

[Java - 메소드 레퍼런스(Method Reference) 이해하기](https://codechacha.com/ko/java8-method-reference/)
