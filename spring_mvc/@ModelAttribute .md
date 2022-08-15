# @ModelAttribute 

Spring MVC에서 @ModelAttribute란 무엇이고 어떻게 사용하는지에 대해서 공부하고 기록을 남기려 합니다.

<br>

## @ModelAttribute가 없이 개발을 한다면?

실제 개발을 할 때는 요청 파라미터로 넘어온 값들을 필요한 객체에 담아줘야 하는데 @ModelAttribute 없이 개발을 한다면 다음과 같이 코드를 작성할 것이다.

<br>

```Java
@Data
public class HelloData {

	private String username;
	private int age;
}
```

<br>

```java
@ResponseBody
@RequestMapping("/model-attribute-v0")
public String modelAttributeV0(@RequestParam String username, @RequestParam int age) {

	HelloData helloData = new HelloData();

	helloData.setUsername(username);
	helloData.setAge(age);

	return "ok";
}
```

<br>

객체를 생성할 클래스를 하나 만들어 두고 요청 파라미터를 @RequestParam을 이용해서 받아준 다음 객체를 생성한 뒤에 setter를 이용해서 객체에 요청 파라미터 값을 담아줄 것이다. 

<br>

딱히 잘못되거나 틀린 방법은 아니지만 약간 번거로운 느낌이 있다. @ModelAttribute를 사용하면 이를 해결할 수 있다.

<br>

## @ModelAttribute 사용법

@ModelAttribtue 사용법은 몹시 간단하다.

<br>

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
	return "ok";
}
```

<br>

이렇게 그냥 컨트롤러에 만든 메소드의 파라미터 자리에 @ModelAttribute 어노테이션을 작성하고 객체를 넣어주면 끝이다. 이렇게 하면 요청 파라미터로 넘어온 값들이 알아서 helloData라는 객체 속에 담긴다. 

<br>

<br>

**어떻게 이것이 가능한 것인가?**

스프링 MVC는 @ModelAttribute가 있으면 다음과 같은 과정을 실행한다.



- 객체를 생성한다.  
- 요청 파라미터 이름으로 객체의 프로퍼티를 찾는다. (프로퍼티: 클래스의 필드 → 객체에 값을 넣을 자리)
- 찾은 프로퍼티에 값을 바인딩한다. 



<br>

<br>

[참고]

- 인프런_김영한 Spring MVC 1편
