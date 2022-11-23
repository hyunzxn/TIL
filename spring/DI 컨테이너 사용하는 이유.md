# Dependency Injection(DI)를 사용하는 이유



## 들어가며

객체지향을 구성하는 다양한 개념이 있지만 그 중 제일 핵심이 되는 개념을 고르자면 **다형성**을 고를 수 있을 것이다. 다형성의 핵심 개념에 대해서 설명을 해보자면 **역할**과 **구현**을 분리해서 생각을 하자는 것이다.

<br>

추상화된 인터페이스로 대표되는 **역할**과 이를 구현한 구체 클래스로 대표되는 **구현**을 각각 자신이 맡은 책임을 기준으로 분리하여 생각을 해보는 것이 다형성의 핵심 개념이 된다. 

<br>

이러한 다형성의 역할과 구현을 분리하여 생각하는 것을 객체지향의 5가지 대표원칙인 SOLID 중 OCP와 DIP 원칙과 연관지어 생각해보면 Spring DI 컨테이너가 등장한 이유에 대해서 이해할 수 있다. 

<br>

객체지향의 5대 원칙 SOLID는 워낙 유명하기에 위에서 언급한 OCP와 DIP에 대해서 설명을 간략히 하면 다음과 같다.

> OCP : 개방폐쇄원칙. 확장에는 열려있고 변경에는 닫혀있어야 한다는 원칙
>
> DIP   : 의존관계역전원칙. 구현 클래스에 의존하지 말고 추상화된 인터페이스를 의존해야된다는 원칙

<br>

<br>

## 왜 DI를 사용하는가?

결론부터 말해보자면 다형성만으로는 OCP와 DIP를 지키며 올바른 객체지향적 설계를 할 수 없기 때문이다. 왜 이렇게 되는 것일까?

<br>

<img width="1279" alt="회원 서비스 인터페이스" src="https://user-images.githubusercontent.com/100478841/203460055-61f09d7a-d579-4e57-84c3-c775650a1014.png">

<br>

<img width="1285" alt="회원 서비스 구현체" src="https://user-images.githubusercontent.com/100478841/203460118-3021642d-8215-4659-bbca-f6dc011410c7.png">

<br>

회원 서비스 인터페이스가 있고 회원 서비스 구현체가 있을 때 회원 서비스 구현체 쪽의 코드에 표시한 부분에 주목해보자. MemberRepository는 생략되었지만 인터페이스이고 MemoryMemberRepository의 경우는 이를 구현한 구체 클래스이다. 회원 서비스 구현체의 코드를 보면 **구체 클래스를 의존하기 때문에 DIP를 위반하고 있으며 만약 MemoryMemberRepository가 아닌 MemberRepository를 구현한 다른 구체 클래스를 사용하기 위해선 해당 코드를 수정해야되기 때문에 OCP까지 위배하고 있는 것이다.** 

그렇다면 `new MemoryMemberRepository()` 부분을 지우게 된다면 어떻게 될까? 이렇게 하면 DIP를 지킬 순 있게 되겠지만 NullPointerException이 터지게 된다. MemberRepository를 구현한 구체 클래스가 특정되지 않았기 때문이다. 

<br>

즉, DIP를 지키기 위해서 구체 클래스를 지우게 되면 NullPointerException이 터지게 되고 이것을 막기 위해 구체 클래스를 바로 의존하게 만들면 OCP를 위배하게 된다. 두 가지 모두를 동시에 충족할 수 있는 방법이 없는 것이다. 이를 어떻게 극복할 수 있을까?

해결책은 의외로 간단한데, **구체 클래스를 외부에서 만들어서 주입(Injection) 해주면 된다.** 비즈니스 로직을 구현하고 있는 클래스에서는 인터페이스만 의존하게 하고 인터페이스를 구현한 클래스는 미리 클래스 외부에서 만들어놨다가 쏘옥 집어넣어주기만 하면 되는 것이다. 그리고 이런 개념을 **의존관계 주입(Dependency Injection, DI)** 라고 한다. 

의존관계 주입을 적용하기 위해서는 우선 **관심사의 분리**가 필요하다. 관심사의 분리란 비즈니스 로직을 담당하는 클래스는 오직 비즈니스 로직을 실행하는데에 중점을 두고 구체 클래스를 만들고 주입하는 것은 외부에서 담당하게 하는 것이다. 각자 자기의 역할에만 집중해서 자기의 역할만 수행한다는 개념이다. 

<br>

관심사의 분리를 적용하여 구체 클래스를 만들어 두는 클래스를 별도로 생성하면 다음과 같이 할 수 있다. 

<img width="1283" alt="AppConfig" src="https://user-images.githubusercontent.com/100478841/203461340-c615b4d6-db92-43a7-accc-cf8268eee2c1.png">

따로 구체 클래스를 만들어 주는 역할을 담당하는 AppConfig 클래스를 만들었다. 표시한 부분을 살펴보면 memberService라는 메소드는 MemberService 인터페이스를 구현하는 구체 클래스인 MemberServiceImpl을 생성하고 그 때 파라미터로 MemoryMemberRepository를 넘겨주는 것을 확인할 수 있다. MemberServiceImpl 클래스를 살펴보면 다음과 같다. 

<br>

<img width="1282" alt="MemberServiceImpl 생성자 주입" src="https://user-images.githubusercontent.com/100478841/203461729-8d96e35b-53f7-4256-846d-85083cd0b83b.png">

- 빨간색 표시 부분을 보면 이제 인터페이스에만 의존하고 있음을 확인할 수 있다. DIP를 잘 지키고 있다. 
- 주황색 표시 부분을 보면 생성자를 통해 AppConfig 클래스에서 넘겨받은 파라미터인 MemoryMemberRepository가 MemberRepository에 할당되면서 이제 MemberRepository를 구현하는 구체 클래스가 MemoryMemberRepository가 됨을 확인할 수 있다. 

<br>

이렇게 하게 되면 장점이 무엇일까? 

1. DIP를 지킬 수 있다. MemberServiceImpl은 오직 인터페이스인 MemberRepository만 의존하고 있기 때문이다. 
2. OCP를 지킬 수 있다. 이것이 가능한 이유는 다음과 같다. 이제 MemberRepository를 구현하는 구체 클래스는 오직 AppConfig에서 만들어서 주입을 해주기 때문에 MemberServiceImpl에서 코드가 수정될 일이 없다. 일단 변경에 닫혀 있음이 보장된다. 그리고 만약 MemberRepository를 구현하는 다른 구체 클래스를 만들고 그것을 사용하고 싶다면 AppConfig에서 생성자에 주입해주는 부분만 변경해주면 되는 것이다. MemberServiceImpl의 입장에서 확장에도 열려있게 된다. 따라서 OCP를 모두 지킬 수 있게 된다.  

<br>

## 마치며

스프링을 공부하게 되면 의존관계 주입을 필수로 거치게 되는 것 같다. 그냥 외워서 쓸 수도 있지만 좋은 객체지향적 설계를 지키기 위해서 이런 개념이 나왔다는 것을 알면서 공부하니 그 의미가 더 와 닿았던 것 같다. 

<br>

### 출처

- 인프런 김영한님 스프링 핵심 기본원리 강의 + 강의자료