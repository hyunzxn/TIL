# 1. Garbage Collection 이란?

자바의 메모리 관리법 중 하나이다. JVM의 힙 영역에 할당됐던 메모리 중 더 이상 필요 없어진 메모리 객체를 모아서 주기적으로 제거하는 프로세스이다. 이러한 GC 때문에 자바 개발자는 개발 시 별도로 메모리를 해제하는 작업을 하지 않고도 메모리 관리에서 비교적 자유롭다는 장점을 가진다.

GC 개념은 자바에만 있는 것은 아니고 파이썬, 자바스크립트, Go 등 다양한 언어에서 지원하는 개념이다. 

<br>

이렇게만 보면 GC 가 장점만 있는 것 같지만 그렇지 않다. 우선 메모리를 해제하는 주체가 JVM의 **Garbage Collector**이기 때문에 언제 GC가 실행되는 지 알 수 없어 제어하기가 힘들다는 문제가 있다. 

또한 GC가 일어나는 동안에는 다른 동작이 멈추는 `Stop-The-World` 현상이 발생하는 문제가 있다.

> **Stop-The-World**
>
> GC를 수행하기 위해 JVM이 프로그램 실행을 멈추는 것을 의미한다. GC가 작동하는 동안 GC 관련 스레드를 제외한 다른 모든 스레드가 동작을 멈춘다. 이 시간을 줄이는 것이 GC의 핵심이다.

&nbsp;

# 2. Garbage Collection 대상

Garbage Collector 는 GC의 대상이 되는 대상을 어떻게 판단하는 것일까? 이것을 판단하기 위해 `도달성(Reachability)` 이라는 기준이 있다. 객체를 참조(Reference)하고 있다면 도달성이 있다고 판단하고 없다면 도달성이 없다고 판단하여 메모리에서 제거해버린다.

<img src="https://github.com/hyunzxn/TIL/assets/100478841/a65395eb-2f7c-42b7-a295-42f611a2f711" width="1000" height="500"/>



분홍색 객체는 아무 곳에서도 참조되고 있지 않기 때문에 GC의 대상이 되는 것이다. 

&nbsp;

# 3. Garbage Collection 방식 - Mark & Sweep

Mark-Sweep 이란 다양한 GC에서 사용되는 알고리즘으로서 객체를 솎아내는 내부 알고리즘이다. 가장 기초적인 방식이라고 할 수 있다.

- Mark

  Root Space 로부터 그래프 순회를 통해 연결된 객체를 찾아가면서 객체 간의 참조관계를 파악한다.



- Sweep

  아무도 참조하고 있지 않은 객체를 제거한다.



- Compact

  스윕 후에 분산된 객체들을 힙의 시작 주소로 모아서 압축하는 과정이다.

<br>

> **Root Space**
>
> 힙 메모리 영역을 참조하는 method area, static 변수, stack, native method stack 등이 있다. Root 에서 대상 객체에 접근이 불가능 하다면 해제의 대상이 되는 것이다.

&nbsp;

# 4. Garbage Collection 동작 과정

JVM의 Heap 영역은 참조 데이터가 저장되는 공간으로서 GC의 대상이 되는 곳이다. 힙 영역은 처음에 설계될 때 다음 2가지를 고려하여 설계됐다.

1. 대부분의 객체는 금방 도달성을 상실한다.
2. 오래된 객체에서 새로운 객체로의 참조하는 경우는 매우 드물다.

<br>

즉, 객체는 일회성이기에 메모리에 오래 남아있는 경우가 드물다는 의미이다. 이를 기반으로 JVM 개발자들은 Heap 영역을 2가지로 나눴다. (원래는 Permanant 영역까지 세 곳으로 나눴지만 Java8 이후로 2가지 영역으로 구분하고 있다. 이 영역은 Native Method Stack 영역에 편입됐다.)

<br>

<img src="https://github.com/hyunzxn/TIL/assets/100478841/21e48788-eb34-4d63-8079-7d8aab683c0f" width="1000" height="500"/>

<br>

**Young 영역**

- 새롭게 생성된 객체가 할당되는 영역
- 대부분의 객체는 Young 영역에 생성되었다가 사라진다. → 많은 객체가 금방 도달성을 상실하기 때문
- Young 영역에 대한 GC 를 `Minor GC` 라고도 한다. 시간이 보통 0.5s ~ 1s 사이에 끝나기 때문에 애플리케이션에 큰 영향을 주지 않는다.

<br>

**Old 영역**

- Young 영역에서 도달성을 유지하여 살아남은 객체가 복사되는 영역
- Young 영역보다 크게 할당되며, 영역의 크기가 큰 만큼 Garbage는 적게 발생한다.
  - 더 크게 할당 되는 이유? Young 영역의 수명이 짧은 객체는 애초에 큰 공간이 필요하지 않고, 큰 공간이 필요한 객체는 바로 Old 영역에 할당되기 때문
- Old 영역에 대한 GC 를 `Major Gc` 또는 `Full GC` 라고도 한다. `Minor GC` 에 비해서 시간이 오래 걸린다(거의 10배 이상). 애플리케이션에 큰 영향을 준다. 여기서 `Stop-The-World` 문제가 발생한다.

<br>

그리고 Young 영역은 효율적인 GC 를 위해 다시 3가지 영역으로 나뉘어진다.

<br>

<img src="https://github.com/hyunzxn/TIL/assets/100478841/43007287-c8f9-4e9b-8a94-77473987833f" width="1000" height="500"/>

<br>

**Eden**

- new 키워드를 통해 새로 생성된 객체가 위치
- 정기적인 Garbage Collectiong 후 살아남은 객체들은 Survival 영역으로 보냄

<br>

**Survivor 0 / Survivor 1**

- 최소 1번의 GC 이상 살아남은 객체가 존재하는 영역
- Survivor 0 또는 Survivor 1 둘 중 하나는 꼭 비어 있어야 한다.

<br>

이렇게 Young 영역을 더 세부적으로 쪼갬으로써 객체의 생존 기간을 더 세심하게 제어하여 효율적인 GC를 할 수 있다. 

&nbsp;

## 4.1 Minor GC 과정

1. 새로운 객체가 Eden 영역에 생성
2. 시간이 지나 Eden 영역이 꽉 차고 `Minor GC` 실행
3. Mark 동작을 통해 도달 가능한 객체 탐색
4. 3번에서 살아남은 객체는 Survivor 0 또는 Survivor 1 영역 (둘 중 하나로) 이동
5. 아무 곳에서도 참조되지 않는 객체는 메모리가 해제됨
6. 살아남은 모든 객체는 age 값이 1씩 증가

> **age 값이란?**
>
> Survivor 영역에서 객체가 살아남은 횟수를 의미하는 값이며, Object 헤더에 기록된다. age 값이 임계값에 다다르면 Old 영역으로 이동 여부를 결정한다. 가장 일반적인 HotSpot JVM의 경우에는 임계값이 31 이다. 

7. 다시 Eden 영역에 신규 객체들로 꽉 차면 다시 한 번 `Minor GC` 실행하고 Mark 한다.
8. Marking 한 객체를 비어있는 Survivor 영역으로 이동시킨다. (기존에 Survivor 영역에 있던 애들도 새로운 Survivor 영역으로 보낸다.)
9. 살아남은 객체는 age 값이 1씩 증가한다. (첫 Minor GC에서 살아남은 애들은 age=2 일 것이고 두 번째 Minor GC에서 살아남은 애들은 age=1 일 것이다.)
10. 이걸 반복

&nbsp;

## 4.2 Major GC 과정

1. 객체의 age 값이 임계값에 다다르면 Old 영역으로 이동한다. → 이걸 `Promotion` 이라 부른다.
2. 위의 과정이 반복되어 Old 영역이 꽉 차면 `Major GC` 가 실행된다.
3. Old 영역에 할당된 메모리가 허용치를 넘게 되면 Old 영역에 있는 모든 객체를 검사하여 참조되고 있지 않은 객체는 한꺼번에 삭제된다.

&nbsp;

# 5. Garbage Collection 알고리즘 종류

## 5.1 Serial GC

- 서버의 CPU 코어가 1개일 때 사용하는 가장 단순한 방식
- GC를 처리하는 스레드가 1개이기 때문에 가장 Stop-The-World 시간이 길다.
- Minor GC 는 Mark-Sweep 을 사용하고 Major GC 는 Mark-Sweep-Compact 를 사용한다.
- 실무에서 잘 사용하지 않는다.

&nbsp;

## 5.2 Parallel GC

- Java 8의 디폴트 방식
- Serial GC와 기본적인 알고리즘은 같지만 Minor GC 를 멀티 스레드가 수행한다. Old 영역은 여전히 싱글 스레드가 수행한다.
- Serial GC에 비해서 Stop-The-World 시간 감소

&nbsp;

## 5.3 Parallel Old GC

- Parallel GC 개선한 버전
- Young 영역, Old 영역 모두 GC를 멀티 스레드가 수행
- 새로운 GC 방식인 Mark-Summary-Compact 방식 사용

&nbsp;

## 5.4 Concurrent Mark Sweep GC

- 애플리케이션의 스레드와 GC 스레드가 동시에 실행되어 Stop-The-World 시간을 최대한 줄이기 위해 고안된 방식
- GC 과정 매우 복잡
- CPU 사용량이 높음 → GC 대상 파악하는게 복잡한 과정 필요하기 때문
- 메모리 파편화 문제 발생
- Java 14 이후 사용 중지

&nbsp;

## 5.5 Garbage First GC

- Java 9+ 버전 디폴트 GC
- 4GB 이상의 힙 메모리, Stop-The-World 시간을 0.5s 정도로 목표로 할 때 사용 권장 (힙 메모리 크기가 너무 작으면 권장 X)
- 기존의 GC 알고리즘은 힙 영역을 Young / Old 로 물리적으로 고정하여 사용하였지만 Garbage First GC는 `Region` 이라는 새로운 개념을 도입
  - 체스판 같은 이미지 떠올리면 좋다
  - 각 Region 마다 역할이 동적으로 부여된다. (Eden, Survivor 등등)

&nbsp;

## 5.6 Shenandoah GC

- Java 12에 릴리즈  
- 강력한 Concurrency 와 가벼운 GC 로직으로 힙 사이즈에 영향을 받지 않고 일정한 pause 시간 소요가 특징

&nbsp;

## 5.7 ZGC

- Java 15에 릴리즈
- 대량의 메모리(8MB ~ 16TB)를 low-latency로 잘 처리하기 위해 디자인 된 GC
- ZPage 라는 영역 사용

&nbsp;

---

**[출처]**

[가비지 컬렉션 동작 원리 & GC 종류 총 정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)