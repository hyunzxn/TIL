# 로깅(Logging)

실무에서는 `System.out.println()`을 사용하지 않고 log를 이용해서 필요한 정보를 출력한다. 성능면에서도 `System.out.println()`보다 로깅을 이용하는 것이 더 좋고 필요한 정보도 많이 얻을 수 있기 때문에 로깅을 이용하는 것이 바람직하다.

<br>

## 로깅 라이브러리

로깅 라이브러리는 종류가 아주 많지만 스프링부트는 기본적으로 스프링부트 로깅 라이브러리인  `spring-boot-starter-logging`를 포함하고 있다.  

<br>

스프링부트 로깅 라이브러리는 두 개의 라이브러리를 기본으로 사용한다.

- SLF4J
- Logback

<br>

SLF4J는 수많은 로그라이브러를 통합해서 제공하는 인터페이스이고 Logback은 SLF4J 인터페이스를 구현한 여러 구현체 중 하나이다. 실무에서는 Logback을 주로 사용한다.

<br>

## 로깅 하는 법

로깅을 하기 위해선 우선 로그 선언을 해야한다. 방법은 세 가지가 있다.

<br>

**로그 선언**

- `private Logger log = LoggerFactory.getLogger(getClass());`
- `private static final Logger log = LoggerFactory.getLogger(Xxx.class)`
- `@Slf4j` (Lombok 사용) → 실무에서 가장 많이 사용!

<br>

**로그 호출**

`log.info("hello")` 

<br>

**로그 레벨**

로그에는 총 5단계의 레벨이 있다.

<br>

- trace
- debug
- info
- warn
- error

<br>

사용하고자 하는 레벨의 로그를 `log.` + 로그레벨 이런 식으로 써주면 된다.

<br>

로그 레벨이 나뉘어져 있는 이유는 개발자가 개발을 하면서 각 단계에 맞는 로그를 남겨서 문제 상황 혹은 정보를 빠르게 파악하기 위함이다.

<br>

**로그 레벨 설정**

로그 레벨을 설정할 수 있다. 로그 레벨을 설정하게 되면 설정한 로그레벨 이하의 로그레벨부터 로그가 콘솔에 보인다. 

ex) 로그레벨을 debug로 설정 시 debug, info, warn, error에 해당하는 로그만 콘솔에 보임.

<br>

설정은 application.properties 파일에서 할 수 있다.

`logging.level.hello.springmvc=debug`

<br>

이렇게 하면 hello.springmvc 패키지에 있는 모든 로그는 debug 레벨부터 보이게 되는 것이다. 

<br>

(참고로 전체 로그 설정은 default가 info 로 되어있다. 패키지 로그 레벨을 info랑 다르게 설정하면 패키지에 적용된 로그레벨 설정이 우선 적용된다.)

<br>

**올바른 로그 사용법** ⭐️

로그를 출력할 때 두 가지 방법으로 할 수 있다.

<br>

1. `log.debug("data="+data)`  ❌
2. `log.debug("data={}", data)` ⭕️

<br>

두 가지 중 어느 것이 옳은 방법일까? 2번 방식이 정답이다. 

<br>

이유는 다음과 같다. 일단 로그 레벨을 info 로 설정했다고 가정하자. 그러면 debug에 해당하는 로그는 출력되지 않을 것이다.

그런데 1번 방식은 자바 언어의 특성상 "data="+data 라는 연산이 실행이 된다. 연산이 실행이 되면 어쩔 수 없이 리소스를 사용하게 된다. 출력도 하지 않을 로그를 위해 불필요한 리소스가 사용되는 것이다. 반면 2번 방식은 로그 레벨이 info 이기 때문에 아무런 연산을 발생시키지 않는다. (본인이 debug 이기 때문에) 따라서 불필요한 리소스 낭비를 막을 수 있다.

<br>

실무에서는 꼭 2번 방식으로 사용하는걸 명심하자!

