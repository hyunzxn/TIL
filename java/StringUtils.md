# StringUtils

<br>

**StringUtils란?**

문자의 유효성 검사를 위해 자주 사용되는 클래스이다. org.springframework.util 패키지 안에 들어있고 스프링부트를 사용한다면 추가적인 의존성 추가는 하지 않아도 된다.

**StringUtils 사용이유?**

가령 문자열의 길이를 체크하고 싶을 때 String 클래스에 있는 isEmpty()를 사용하면 Null 체크가 되지 않아 NullPointerException이 발생할 수 있다는 단점이 있다. StringUtils 클래스는 Null 체크를 자동으로 해주기 때문에 NPE를 방지할 수 있다는 장점이 있다.

<br>

**주요 메소드**
- hasLength

null체크 후, String클래스의 isEmpty를 호출하여 길이가 0인지 판단해주는 메소드이다. Null 체크를 해주기 때문에 NPE를 막을 수 있다. 주의할 점으로는 공백만 있는 문자열 (예를 들어 “ ”) 도 true가 반환된다. 

- hasText

hasLength에다 추가적으로 공백이 아닌 문자열이 존재하는지까지 검증해주는 메소드이다. 결론적으로 hasText 메소드는 null 체크, 길이가 0보다 큰 지 체크, 공백이 아닌 문자열이 하나라도 포함되어 있는지를 체크해주는 메소드이다. 문자열 처리에 관련해서 쓰면 좋을 것 같다.