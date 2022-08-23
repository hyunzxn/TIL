# Spring Message & Internatioanlization

스프링이 제공하는 메시지와 국제화 기능에 대해서 기록해보고자 한다.

<br>

## Message란

화면에 렌더링 되는 글자, 문구 등

<br>

스프링이 제공하는 메시지 기능이란 화면에 보이는 문자들을 한 군데에 모아놓을 수 있는 기능이다. 이렇게 글자들을 한 군데에 모아놓으면 일괄적인 수정에 용이하기 때문에 요구사항이 변경되었을 때 대응이 쉽다는 장점을 가지고 있다. 

<br>

## 국제화(Internatioanlization)란?

화면에 표시되는 글자, 문구 등이 클라이언트의 언어에 맞게 최적화가 되어 표시되게끔 지원하는 기능이다. 가령 동일한 웹 사이트여도 우리나라에서 접속하면 한글로 표시되고 미국에서 접속하면 영어로 표시되게끔 하는 것이다. 

<br>

## 메시지 & 국제화 기능 사용법

메시지 관리기능을 사용하려면 스프링이 제공하는 `MessageSource` 인터페이스를 스프링 빈으로 등록하면 되는데 스프링 부트를 사용한다면 스프링 부트가 자동으로 `MessageSource`를 스프링 빈으로 등록하기 때문에 별도로 등록하지 않고 `messages.properties`만 만들어서 메시지를 등록해놓고 사용할 수 있다. 

<br>

`application.properties`에 `spring.messages.basename=messages`설정을 추가해주면 메시지 기능을 사용할 수 있다. 그런데 이것을 추가해주지 않더라도 사용할 수 있다. 별도의 설정을 하지 않으면 스프링 부트가 디폴트 설정으로 `messages`라는 이름으로 등록하기 때문이다. `basename`이라는 것은 스프링 부트가 읽을 파일의 이름이라고 생각하면 좋다. 

<br>

**메시지 파일 만들기**

<u>/resources/messages.properties</u> 경로로 파일을 만들어준다. (이 때 파일명이 **messages** 복수형이라는 것에 주의)

<u>/resources/messages_en.properties</u> 경로로 국제화를 위한 파일도 만들어준다.

<br>

각 파일에 이제 표시될 메시지들을 적어놓으면 된다. 

Ex) messages.properties

```prop
label.item=상품 
label.item.id=상품 ID 
label.item.itemName=상품명
```

<br>

`messages.en` 같은 경우에는 HTTP 헤더의 `Accept-Language` 값이 en인 경우에 표시된다.

<br>

## 화면에서 메시지 호출해서 사용하기

타임리프가 제공하는 스프링 통합 기능 중 메시지 기능을 손쉽게 사용할 수 있는 기능이 있다. 

<br>

`th:text="#{메시지 키}"`

Ex) th:text="#{label.item.itemName}" 

<br>

`#{}`포맷에 중괄호 안에 메시지 파일에 등록해놓은 메시지의 key를 넣어주면 메시지의 value가 화면에 출력된다.