# Spring MVC 구조

스프링 MVC 는 FrontController 패턴으로 만들어져 있다. 

<br>

<u>주요 개념</u>

- 프론트 컨트롤러의 역할: **DispatcherServlet**
- 핸들러 매핑: **HandlerMapping**  → 여러 개의 핸들러(컨트롤러)를 찾아주는 역할 수행
- 핸들러 어댑터: **HandlerAdapter** → 다양한 종류의 핸들러를 호출하는 역할 수행
- **ModelAndView**: 모델과 뷰 담고 있는 객체
- **viewResolver**: String 형태로 넘어온 viewName을 받아서 VIew 객체 생성
- **View**: 모델과 viewName 받아서 화면에 그려주는 역할 수행

<br>


# HTTP 요청에 따른 Spring MVC 처리 흐름

- 클라이언트가 브라우저를 통해 HTTP 요청을 보냄
- 요청이 DispatcherServlet에 들어옴
- DispatcherServlet에서 HandlerMapping을 통해 요청을 처리할 수 있는 Handler를 조회함
- DispatcherServlet에서 조회한 Handler를 처리할 수 있는 HandlerAdapter 목록에 있는 HandlerAdapter를 조회함
- 조회한 Handler를 HandlerAdapter에 넘겨서 Handler를 호출함
- HandlerAdapter는 ModelAndView 객체를 DispatcherServlet 에 반환함
- DispatcherServlet는 ModelAndView 를 이용해서 viewName을 viewResolver에 보냄
- viewResolver는 View 객체를 DispatcherServlet 한테 반환해줌
- DispatcherServlet는 반환받은 View 객체에 있는 render 메소드를 통해 화면 그려서 클라이언트에 보여줌. (모델이 여기에 담겨져 있어서 화면 바로 그리기 가능)


<br>


(인프런 김영한 - Spring MVC 1편 강의 참고)

<br>

<u>잘못된 내용이 있을 수 있습니다. 지적해주시면 더 공부하여 수정하겠습니다!</u>
