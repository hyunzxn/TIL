# 1. JVM 개념

Java Virtual Machine 의 줄임말이다. JVM이 있기 때문에 자바로 작성된 **프로그램은 어떤 OS에도 종속되지 않고** 실행될 수 있다. Java로 작성된 소스 코드(*.java)는 CPU가 인식을 하지 못하므로 기계어로 컴파일을 해줘야 한다. 

하지만 Java는 JVM을 거쳐서 OS에 도달하기 때문에 기계어로 바로 컴파일 되는 것이 아니라 JVM이 인식할 수 있는 Java Bytecode(*.class) 로 먼저 변환된다. 즉 자바 컴파일러는 *.java 파일을 → *.class 로 변환해주는 것이다.

이것을 그림으로 표현하면 다음과 같다.

![flow](https://github.com/hyunzxn/TIL/assets/100478841/05029d19-90a6-4f60-940a-d2fcee13aee7)

&nbsp;

> **바이트 코드란 무엇일까?** 
>
> 바이트 코드란 가상 컴퓨터에서 돌아가는 프로그램을 위한 이진 표현법이다. 자바 바이트 코드란 JVM이 이해할 수 있는 언어로 변환된 자바 소스 코드를 의미한다. 자바 바이트 코드는 다시 실시간 번역기 또는 JIT 컴파일러에 의해 바이너리 코드로 변환된다. 바이너리 코드는 컴퓨터가 이해할 수 있는 0 과 1 로만 구성되어 있는 이진 코드이다.

&nbsp;

# 2. JVM 동작 방식

<img src="https://github.com/hyunzxn/TIL/assets/100478841/bcb280dd-8c7d-40d4-9837-43a6aae9ce16" width="500" height="500"/>

이 그림을 순차적으로 분석해보면 다음과 같다.

1. 자바 프로그램을 실행하면 OS가 JVM에 메모리를 할당해준다.
2. 자바 컴파일러가 자바 소스코드를 자바 바이트코드로 컴파일한다.
3. Class Loader 가 동적 로딩을 통해 필요한 클래스들을 로딩 및 링크하여 Runtime Data Area 에 올린다.
4. Runtime Data Area 에 로딩 된 바이트코드는 Execution Engine 에 의해 해석된다.
5. 이 과정에서 Execution Engine 에 의해 GC(Garbage Collector)가 작동하고 Thread 동기화가 이뤄진다.

&nbsp;

이제 JVM 구조에 대해서 알아보면서 이것이 무슨 의미인지 알아볼 것이다.

&nbsp;

# 3. JVM 구조

JVM 구조는 다음과 같다.

- 클래스 로더(Class Loader)
- 실행 엔진(Execution Engine)
  - 인터프리터
  - JIT 컴파일러
  - 가비지 콜렉터
- 런타임 데이터 영역
  - 메소드 영역
  - 힙 영역
  - 스택 영역
  - PC Register
  - 네이티브 메서드 스택 영역
- JNI(Java Native Interface)
- 네이티브 메서드 라이브러리

&nbsp;

## 3.1 클래스 로더

클래스 로더는 JVM 내로 클래스 파일(.class) 들을 **동적으로 로드하고 링크를 통해 배치**하는 작업을 수행하는 모듈이다. 다시 말해서 바이트 코드(.class)들을 엮어서 JVM 메모리 영역인 런타임 데이터 영역에 배치한다. 

이 때 클래스 파일을 한 번에 메모리에 올리는 것이 아니라 애플리케이션에서 필요한 경우에 동적으로 메모리에 적재하는 방식이다.

클래스 파일의 로딩은 세 가지 순서로 진행된다.

1. **로딩** : 클래스 파일을 가져와 JVM 메모리에 올린다.
2. **링크**: 클래스 파일을 사용하기 위해 검증을 하는 과정이다.
   - 검증: 메모리에 올라온 클래스가 JVM 명세에 명시된 대로 구성되어있는지 검사한다.
   - 준비: 클래스가 필요로 하는 메모리를 할당한다.
   - 분석: 클래스의 상수 풀 내의 모든 심볼릭 레퍼런스를 다이렉트 레퍼런스로 변경한다.
3. **초기화**: 클래스 변수들을 적절한 값으로 초기화한다.

&nbsp;

클래스 로더의 종류는 세 가지가 있다.

1. **Bootstrap Class Loader**
   - 네이티브 코드로 작성되었으며 JVM에 내장되어있다.
   - JVM이 시작될 때 실행되며 JVM 실행에 필요한 클래스들을 로딩한다.
2. **Platform Class Loader**
   - 자바에서 기본적으로 제공해주는 클래스를 로딩할 때 사용한다.
   - Bootstrap Class Loader를 부모로 가지고 있다.
   - Java 8까지는 Extension Class Loader로 불렸지만, 모듈 시스템이 도입되면서 이름이 변경되었다.
3. **System Class Loader**
   - 유저가 작성한 클래스를 로딩할 때 사용된다.
   - ClassPath에 명시된 경로를 통해 클래스를 찾는다.
   - Platform Class Loader를 부모로 가지고 있다.
   - Java 8 까지는 Application Class Loader로 불렸지만, 모듈 시스템이 도입되면서 이름이 변경되었다.

&nbsp;

클래스 로더의 특징은 다음과 같다.

1. 위임 모델 사용

   ➡︎ 자신에게 클래스 로딩 요청이 들어오면 자신의 부모 클래스에게 로딩 요청을 보내고 부모가 클래스를 찾지 못 하면 자신이 탐색한다.

2. 계층 구조 

   ➡︎ 상위 클래스 로더의 클래스는 하위 클래스에서 볼 수 있지만 반대는 성립하지 않는다.

&nbsp;

## 3.2 실행 엔진

런타임 데이터 영역에 배치된 바이트 코드를 명령어 단위로 읽어 실행한다. 바이트 코드는 엄밀히 말하면 기계어가 아니라 JVM이 이해할 수 있는 언어이기 때문에 실행 엔진이 기계어로 변환해주는 것이다.

이 과정에서 실행 엔진은 **인터프리터**와 **JIT 컴파일러** 두 가지 방식을 혼합하여 바이트 코드를 실행한다. 

- 인터프리터 방식

  바이트 코드 명령어를 하나씩 읽고 해석하고 바로 실행한다. JVM 안에서 바이트 코드는 기본적으로 인터프리터 방식으로 실행된다. 반복되는 코드여도 매번 하나씩 읽고 바로 실행하기 때문에 전체적인 속도가 느리다는 단점이 있다.

- JIT 컴파일러

  반복되는 코드를 발견하여 바이트 코드 전체를 한 번에 컴파일하여 Native Code로 변경하여 이후에는 해당 코드를 더 이상 인터프리팅 하지 않고 캐싱해두었다가 네이티브 코드로 바로 실행하는 방식이다. 전체적인 속도가 인터프리팅 방식보다 빠르다. 

  그러나 네이티브 코드로 변환하는 것도 비용이 소모되므로 모든 코드를 JIT 컴파일 방식으로 실행하지 않고 인터프리팅 방식으로 실행하다가 일정 기준이 넘어가면 JIT 컴파일 방식으로 실행한다.

  여기서 기준을 핫스팟 이라고 부른다. 일반적으로 다음과 같다. 다만 핫스팟은 JVM 구현에 따라 달라질 수 있다.

  - 호출 빈도 수: 특정 메서드가 실행되는 횟수 → 자주 호출되는 메서드는 JIT 컴파일의 대상이 된다.
  - 루프 반복 횟수: 특정 루프가 반복되는 횟수가 많을수록 JIT 컴파일의 대상이 된다. 

  

> **네이티브 코드**
>
> Java 의 부모가 되는 C, C++, 어셈블리어로 구성된 코드를 의미한다.

&nbsp;

**가비지 콜렉터**

힙 메모리 영역에서 더는 사용하지 않는 메모리를 자동으로 회수하는 역할을 담당한다. C , C++ 같은 언어는 개발자가 사용하지 않는 메모리를 직접 해제해줘야 하지만 Java는 JVM 에 있는 가비지 콜렉터가 자동으로 사용되지 않는 메모리를 해제한다.

일반적으로 자동으로 실행되지만 GC가 실행되는 시간은 정해져 있지 않다. GC에 관해서는 다시 한 번 더 상세하게 정리해보도록 하겠다.

&nbsp;

## 3.3 런타임 데이터 영역

JVM의 메모리 영역으로서 애플리케이션을 실행할 때 사용되는 데이터들을 적재하는 영역이다. 세부 구조는 다음과 같다.

<img width="760" alt="runtime data area" src="https://github.com/hyunzxn/TIL/assets/100478841/ea94f895-b425-4c93-95d6-61d4747a905f">

- ### 메서드 영역(스태틱 영역)

  - JVM이 시작될 때 생성된다.

  - 바이트 코드를 처음 메모리 공간에 올릴 때 초기화되는 대상들을 저장하기 위한 공간이다.

  - JVM이 동작하고 클래스가 로드될 때 적재돼서 프로그램이 종료될 때까지 저장된다.

  - 모든 스레드가 공유하는 영역이다. 다음과 같은 정보들이 저장된다.

    - Field Info: 멤버 변수의 이름, 데이터 타입, 접근 제어자 정보
    - Method Info: 메소드 이름, return 타입, 매개 변수, 접근 제어자 정보
    - Type Info: Class 인지 Interface 인지 여부 저장, Type 의 속성과 이름, Super Class 이름

  - 메서드 영역 안에는 **Runtime Constant Pool** 을 별도로 존재한다.

    - 각 클래스 / 인터페이스 마다 별도의 constant pool 이 존재하는데, 클래스를 생성할 때 참조해야 할 정보들을 상수로 가지고 있는 영역이다. 
    - JVM은 이 constant pool 을 통해 메소드나 필드의 실제 메모리상 주소를 찾아 참조한다. 
    - 상수 자료형을 저장하여 참조하고 중복을 막는 역할을 수행한다.

    

- ### 힙 영역

  - JVM이 관리하는 프로그램 상에서 데이터를 저장하기 위해 런타임 시에 동적으로 할당하여 사용하는 영역이다.
  - 모든 스레드가 공유하는 영역이다.
  - new 키워드로 생성되는 객체, 인스턴스 변수, 배열 타입 등 참조 타입이 저장되는 곳이다.
  - 객체가 더 이상 사용되지 않거나 명시적으로 객체를 null 로 선언할 때 까지 사용한다.
  - **GC의 대상**이 되는 영역이다.
  - 힙 영역에 생성되는 객체와 배열은 참조 타입(Reference Type)으로서 JVM 스택 영역의 변수나 다른 객체의 필드에서 참조된다. 즉, 주소는 스택 영역에 있고 실제 데이터는 힙 영역에 있는 것이다. 만약 힙 영역에 있는 값을 참조하는 변수나 필드가 없다면 GC 에 의해서 힙 영역에서 제거된다.

  

- ### 스택 영역

  - 각 스레드마다 하나씩 존재한다. 스레드가 시작될 때 할당된다.
  - 프로세스가 메모리에 로드될 때 스택 사이즈가 고정되어 있기 때문에 런타임 시에 스택 사이즈를 변경할 수 없다. 만약 프로그램 실행 중 메모리 크기가 충분하지 않으면 스택오버플로우 에러가 발생한다.
  - 스레드가 종료되면 런타임 스택도 사라진다.
  - 기본 타입 변수를 생성할 때 저장하는 공간이다. 스택 영역에 직접 값을 가진다.
  - 참조 타입 변수는 힙 영역이나 메서드 영역의 **주소**를 가진다. 값을 직접 가지는 것이 아니다. 
  - 임시적으로 사용되는 변수나 정보들이 저장되는 영역이다.
  - 자료구조의 Stack 처럼 LIFO 구조이다. 
  - 메서드를 호출 할 때마다 각각의 스택 프레임이 생성되고 메서드 안에서 사용되는 값들을 저장하고, 호출된 메서드의 매개변수, 지역변수, 리턴 값, 연산 시 발생하는 값들 임시로 저장한다.

  > **스택 프레임**
  >
  > 메서드가 호출될 때마다 만들어진다. 현재 실행 중인 메서드 상태 정보를 저장하는 공간이다. 
  >
  > 메서드 호출 범위가 종료되면 스택에서 제거된다.



- ### PC Register

  - 스레드가 시작될 때 생성된다.

  - 현재 수행 중인 JVM 명령어 주소를 저장하는 영역이다.

  - JVM 명령어 주소는 스레드가 어떤 부분을 무슨 명령으로 실행해야 할 지에 대한 기록을 가지고 있다.

  - 프로그램을 실행하는 궁극적인 주체는 CPU이다. CPU가 명령어를 수행해야 프로그램이 실행될 수 있다. 이 때 CPU는 **레지스터** 라고 하는 CPU 내의 기억장치를 사용하여 연산을 진행한다. 예를 들어 A 와 B를 더하라는 연산이 있다면, A를 받고 B를 받고 더하라는 작업을 받게 되는데 각각의 값을 기억해 둘 공간이 필요한데 그 곳이 바로 레지스터이다.

  - 그러나 자바의 PC Register는 이런 CPU Register와 다르다. 자바는 OS 나 CPU 입장에서는 하나의 프로세스이기 때문에 JVM의 리소스를 사용해야한다. 따라서 자바가 CPU에 바로 연산을 수행하도록 하는 것이 아니라 작업 하는 내용을 CPU에게 연산으로 제공해야하고 이를 위한 버퍼 공간으로서 PC Register 라는 메모리 영역을 사용한다.

  - 스레드가 자바 메서드를 수행하고 있으면 JVM 명령어 주소를 PC Register에 저장한다. 만약 자바 메서드가 아닌 다른 언어의 메서드를 수행하고 있다면 undefined 상태가 된다.



- ### 네이티브 메서드 스택 영역

  - 자바 소스 코드가 컴파일 되어 생성된 자바 바이트 코드가 아닌 실제로 실행할 수 있는 기계어로 작성된 프로그램을 실행시키는 영역이다.
  - 자바 이외의 언어(C, C++, 어셈블리)로 작성된 네이티브 코드를 실행하는 공간이다.
  - JIT 컴파일러에 의해 변환된 네이티브 코드가 여기서 실행되는 것이다.
  - 일반적인 자바 메서드를 실행하면 스택 영역에 쌓이다가, 해당 메서드 내부적으로 네이티브 방식을 사용하는 메서드가 있다면 해당 메서드는 네이티브 메서드 스택 영역에 쌓인다. 그리고 네이티브 메서드 수행이 끝나면 다시 그냥 스택 영역으로 돌아와 작업을 이어서 수행한다.

&nbsp;

## 3.4 JNI(Java Native Interface)

자바가 다른 언어로 만들어진 애플리케이션과 상호작용 할 수 있도록 인터페이스를 제공하는 프로그램이다. 앞서 살펴본 네이티브 메서드 스택 영역에 적재하고 수행되도록 하는 것이 JNI 이다.

&nbsp;

## 3.5 네이티브 메서드 라이브러리

C, C++로 작성된 라이브러리를 의미한다.

&nbsp;

---

**<출처>**

- **[Inpa Dev](https://inpa.tistory.com/entry/JAVA-%E2%98%95-JVM-%EB%82%B4%EB%B6%80-%EA%B5%AC%EC%A1%B0-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%98%81%EC%97%AD-%EC%8B%AC%ED%99%94%ED%8E%B8#thankYou)**
- **[It is Whatit is](https://velog.io/@sgwon1996/JAVA%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC%EC%99%80-JVM-%EA%B5%AC%EC%A1%B0)**
