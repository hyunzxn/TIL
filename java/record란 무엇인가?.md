# Record란 무엇인가?

**참고**

[곰민 님 블로그](https://colevelup.tistory.com/28)

[Baeldung](https://www.baeldung.com/java-record-keyword)

&nbsp;

# 1. 들어가며
Record는 Java 14에서 프리뷰로 등장하고 Java 16에서 정식으로 출시된 기능입니다. Record는 불변 데이터를 전달할 때 사용됩니다. Record 이전에는 불변 데이터를 전달하기 위해서는 여러 보일러 플레이트가 포함된 클래스를 만들어야 했습니다. 그러나 이 과정이 복잡하고 그로 인한 실수할 여지가 많았습니다. 이번 글에서는 이런 수고를 덜어준 Record 클래스에 대해서 정리해보도록 하겠습니다.

&nbsp;

# 2. Record가 필요한 이유

```java
public class Person {

    private final String name;
    private final String address;

    public Person(String name, String address) {
        this.name = name;
        this.address = address;
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, address);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        } else if (!(obj instanceof Person)) {
            return false;
        } else {
            Person other = (Person) obj;
            return Objects.equals(name, other.name)
              && Objects.equals(address, other.address);
        }
    }

    @Override
    public String toString() {
        return "Person [name=" + name + ", address=" + address + "]";
    }

    // standard getters
}
```

불변 데이터를 담는 예시 코드입니다. 이 코드의 문제점을 몇 가지 살펴보면 다음과 같습니다.

1. 보일러 플레이트 코드가 너무 많습니다. 
   - hasCode()
   - equals()
   - toString()
2. 그로 인해 클래스의 목적이 모호해집니다. 실제로는 클래스가 Person 객체의 이름과 주소라는 상태를 가지고 있기만 하면 되는데 보일러 플레이트 코드 때문에 본연의 목적이 잘 나타나지 않습니다.

&nbsp;

Record를 사용하면 이 문제를 해결할 수 있습니다.

```java
public record Person (String name, String address) {}
```

&nbsp;

# 3. Record 살펴보기

## 3.1 생성자

record를 만들면 각 필드가 argument로 들어간 public 생성자가 자동으로 생성됩니다. 그래서 이걸 활용해서 객체를 생성할 수 있습니다. 

```java
Person person = new Person("John Doe", "100 Linda Ln.");
```

## 3.2 Getter

record를 만들면 각 필드에 접근하는 public Getter 역시 만들어 줍니다. 간단하게 예시를 보면 다음과 같습니다.

```java
Person person = new Person("John Doe", "100 Linda Ln.");

System.out.println(person.name());
System.out.println(person.address());

// John Doe
// 100 Linda Ln.
```

## 3.3 equals

equals 메서드 또한 자동으로 생성해줍니다. 이 메서드는 제공된 객체의 유형이 동일하고 모든 필드의 값이 일치할 때 `true`를 반환합니다.

```java
String name = "Daniel";
String address "Seoul";

Person person1 = new Person(name, address);
Person person2 = new Person(name, address);

System.out.prinln(person1.equals(person2));

// true
```

## 3.4 hashCode

hashCode 메서드 또한 자동으로 생성해줍니다. 두 객체의 필드 값이 모두 일치하는 경우에 두 객체에 대한 동일한 값을 반환합니다.

## 3.5 toString

record는 toString() 메서드도 자동으로 생성해줍니다. 

```java
String name = "Daniel";
String address = "Seoul";

Person person = new Person(name, address); 

System.out.println(person.toString());

// Person[name=Daniel, address=Seoul]
```

&nbsp;

# 4. 제약 사항

record는 제약 사항이 있습니다.

- record는 암묵적으로 final 클래스이기 때문에 상속이 불가하고, abstract 선언이 불가합니다.
- 다른 클래스를 상속할 수 없습니다. 인터페이스 구현은 가능합니다.

&nbsp;

# 5. 마치며

record의 사용법을 좀 살펴보면 DTO를 만들 때 많이 사용하는 것 같습니다. DTO의 값은 바뀔 일이 없고 데이터를 전달하는 책임만 잘 수행하면 되기 때문에 record를 잘 사용하면 좋을 것 같습니다.