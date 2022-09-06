# 로그인 처리 1 - 쿠키, 세션

로그인 처리에 꼭 필요한 쿠키와 세션의 개념에 대해 정리해보고자 한다.

<br>

## 쿠키(Cookie)

쿠키는 HTTP 의 stateless(무상태성) 성질과 관련이 있다. 유저가 로그인을 하고 서버에 어떤 요청을 보내고 서버가 그 요청에 대한 응답을 하고 나면 HTTP 연결이 끊긴다. 그리고 유저가 다른 요청을 보낸다면 서버는 그 유저가 이전에 요청을 한 동일한 유저인지 식별할 수 있는 방법이 없다. 

<br>

이를 해결하기 위해 요청 URL에 쿼리파라미터 형식으로 매번 유저 정보를 넣을 수도 있지만 이렇게 하는 것은 너무 번거롭다. 이런 점을 해결하기 위한 대안으로 **쿠키**가 있다.

<br>

**동작원리**

클라이언트에서 서버에 로그인 요청을 보내고 서버가 로그인 요청을 성공적으로 수행하면 서버는 클라이언트에 쿠키를 보내준다. 그러면 클라이언트는 이 쿠키를 자체 쿠키 저장소에 저장한 뒤 이후에 클라이언트가 서버에 요청하는 모든 요청에 쿠키를 같이 담아서 보낸다. 그러면 서버는 이 쿠키를 확인함으로써 동일한 유저의 요청인 것을 확인할 수 있는 것이다.

<br>

**쿠키의 종류**

- 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지가 되는 쿠키
- 세션 쿠키: 만료 날짜 입력을 생략하면 브라우저 종료 시 사라지는 쿠키

<br>

**쿠키의 보안문제**

쿠키는 간편한 방법으로 유저를 식별할 수 있다는 장점이 있지만 여러가지 보안문제가 있다.

- 쿠키의 값 임의변경이 가능
  - 클라이언트 측에서 쿠키의 값을 임의로 변경하는 것이 얼마든지 가능하다.
  - 이렇게 변경이 되면 완전히 다른 사용자로 인식이 되는 보안 문제가 발생한다.
- 쿠키에 보관된 정보가 털릴 수 있다.
  - 쿠키의 정보가 나의 로컬 PC를 통해서든 네트워크를 통해서든 털릴 수 있다는 위험이 존재한다.
- 해커가 쿠키를 한 번 훔쳐가면 계속해서 그 쿠키를 사용할 수 있다. 
  - 만료 되기 전까지 계속해서 쿠키를 사용하면서 악의적인 요청을 할 수 있다.

<br>

이러한 쿠키의 보안적 한계를 극복하기 위해서 사용하는 것이 Session(세션)이다.

<br>

## 세션(Session)

세션은 쿠키의 한계를 극복하기 위해 나온 개념으로서 서버에 중요한 정보를 보관하면서 연결을 유지하는 개념이다. 

<br>

**세션의 장점**

- 예상 불가능한 세션 Id를 사용함으로써 임의로 변경이 불가능하게 만든다.
- 세션Id가 실제로 털린다고 하더라도 중요한 정보가 담겨있지 않기 때문에 보안상 위협이 쿠키보다 덜하다.
- 세션 만료시간을 짧게 설정해놓으면 해커가 털어가도 오래 사용하지 못 한다.

<br>

**세션의 작동원리**

클라이언트에서 서버로 로그인 요청을 보내면 서버에서 로그인 요청을 성공적으로 수행하고나면 바로 쿠키에 유저 정보를 담는 것이 아니라 세션 저장소라는 곳에 sessionId와 정보를 같이 담는다. 이 때 세션id는 추정이 불가능한 무작위 아이디 형식으로 만들어진다. 
<br>

<img src="/Users/moonhyunjun/Desktop/세션아이디 쿠키로 보냄.png" alt="세션아이디 쿠키로 보냄" style="zoom:50%;" />

<br>

세션저장소에 정보를 담은 후에 서버는 클라이언트에 쿠키를 보내는데 이 때 쿠키에는 세션id만 담겨있다. 클라이언트는 세션id만 담겨있는 쿠키를 쿠키 저장소에 저장한다. 그리고 이후에 클라이언트가 보내는 모든 요청에 이 쿠키를 담아서 같이 보낸다. 그러면 서버는 이 쿠키를 보고 세션아이디를 확인한 후에 세션저장소에서 세션id를 조회하고 해당 세션id에 해당하는 정보를 클라이언트에 반환해서 돌려준다. 
<br>

<img src="/Users/moonhyunjun/Desktop/세션아이디로 유저정보 반환.png" alt="세션아이디로 유저정보 반환" style="zoom:50%;" />

<br>

## 세션 사용하는 방법

세션은 크게 세 가지의 기능을 담당한다.

- 세션 생성
  - 세션 Id 생성
  - 세션 저장소에 세션id와 정보 저장
  - 세션 아이디를 쿠키에 담아서 클라이언트에 전달
- 세션 조회
  - 클라이언트가 요청한 쿠키에 담겨져 있는 세션 Id로 세션 저장소에서 정보 조회
- 세션 만료
  - 클라이언트가 요청한 쿠키에 담겨져 있는 세션 Id에 해당하는 세션을 세션 저장소에서 삭제

<br>

서블릿에선 다행히 이런 세션의 기능을 기본적으로 제공하고 있다. 서블릿이 제공하는 세션은 `HttpSession`을 통해서 이용할 수 있다.

<br>

```java
public interface SessionConst {
	String LOGIN_MEMBER = "loginMember";
}
```

HttpSession 에 데이터를 보관하고 조회할 때, 같은 이름이 중복 되어 사용되므로, 상수를 하나 정의했다.

<br>

```java
@PostMapping("/login")
public String loginV3(@Valid @ModelAttribute("loginForm") LoginForm form, BindingResult bindingResult, HttpServletRequest request) {
        
	// 검증 통과하지 못 해서 에러가 있을 때
        if (bindingResult.hasErrors()) {
        return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        if (loginMember == null) {
        bindingResult.reject("loginFail", "아이디 또는 비밀번호가 틀렸습니다.");
        return "login/loginForm";
        }

        // 로그인 성공처리

        // 세션이 있으면 있는 세션을 반환하고 세션이 없으면 신규 세션을 생성해서 반환
        HttpSession session = request.getSession();
        // 세션에 로그인 회원 정보 보관
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

        return "redirect:/";
        }
```

<br>

- `HttpSession session = request.getSession();`으로 세션을 생성한다.
- `session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);`으로 회원정보를 저장한다.

<br>

```java
@GetMapping("/")
public String homeLogin(
        @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember, Model model) {

        // 세션에 회원 데이터가 없으면 home
        if (loginMember == null) {
            return "home";
        }

        // 세션이 유지되면 로그인으로 이동
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
```

세션에 있는 정보를 화면에서 이용하는 코드이다. 

- `@SessionAttribute` 어노테이션을 이용해서 쉽게 세션에 있는 정보를 조회해서 사용할 수 있다. 