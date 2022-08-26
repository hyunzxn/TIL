# Validation1

<br>

## 검증이 왜 필요한가?

컨트롤러를 통해 들어오는 HTTP 요청이 정상인지 검증하는 것은 중요하다. 이상한 데이터가 데이터베이스에 저장되는 것을 막아야 하기 때문이다. 검증은 클라이언트와 서버 두 곳 모두에서 할 수 있지만 클라이언트만으로 단독적으로 검증 작업을 하는 것은 위험하다. 자바스크립트를 조작할 수 있기 때문이다. 그러므로 반드시 서버에서 검증을 하는 작업을 해줘야 한다.

<br>

## Spring이 제공하는 검증 처리 방법 - BindingResult

### BindingResult란?

BindingResult는 스프링이 제공하는 검증 처리를 위한 인터페이스로서 Errors 라는 인터페이스를 상속하고 있다. BindingResult 객체를 생성하면 `addError()`라는 메소드를 통해 BindingResult 객체에 필드에러, 객체에러(글로벌에러)를 담을 수 있다. 

<br>

BindingResult 객체를 사용하기 위해서는 컨트롤러의 파라미터에 `BindingResult bindingResult`를 선언해주면 되는데 이 때 주의할 점이 있다. bindingResult에 **어떤 객체**에 관련된 잘못된 요청인지를 알려주기 위해 bindingResult는 반드시 `@ModelAttribute `로 선언된 객체 뒤에 위치해야한다. <u>순서가 중요한 파라미터라는 것이다.</u> 

<br>

Ex) 

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, BindingResult bindingResult, ...) {
  ....
}
```

<br>

### BindingResult를 활용해서 검증 처리하는 방법

BindingResult를 활용해서 검증 처리를 하는 방법은 다음과 같다.

<br>

Ex)

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, BindingResult bindingResult, ...){
  if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, new String[]{"required.item.itemName"}, null, null));
        }
  .... (이하 생략)
}
```

<br>

bindingResult.addError() 부분을 잘 봐야한다. FieldError 클래스를 이용해서 검증 처리하는 방식이다. FieldError 클래스는 생성자를 2개 가지고 있는데, 여기서 사용한 생성자는 두 번째 생성자이다. 두 번째 생성자가 가지는 파라미터는 다음과 같다.

<br>

```java
public FieldError(String objectName, String field, @Nullable Object rejectedValue, boolean bindingFailure,
			@Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage) {
  
  ... 이하 생략
}
```

<br>

- `ObjectName`은 HTTP 요청을 받아서 데이터를 담을 객체이다. (예시에 따르면 Item)
- `Field`는 해당 객체에서 어떤 부분(이상한 데이터가 입력된 특정 필드를 의미한다. 예시에 따르면 itemName)
- `rejectedValue`는 고객(사용자)이 잘못한 입력한 값을 의미한다. 이것을 써주는 이유는 잘못된 요청을 고객이 했을 때 고객이 어떤 잘못된 값을 입력했는지를 화면에 그대로 남겨주기 위함이다. (가령 숫자를 입력해야되는데 문자를 입력한 경우에 어떤 문자를 입력했는지 그대로 남겨두는 것이다)
- `bindingFailure`는 고객이 입력한 정보가 제대로 bindingResult에 바인딩 되지 않는 경우에 true 그렇지 않으면 false의 값을 가진다.
- `codes`는 검증 오류 메시지에 관련된 것이다. `codes=검증 오류메시지 내용` 이런 식으로 errors.properties에 미리 담아놓고 검증 오류 메시지를 화면에 출력할 수 있다. (메시지, 국제화와 완전 동일한 방식이다.) codes는 String[] 형식으로 담아주면 된다.

- `arguments`는 검증 오류 메시지에 들어가는 파라미터를 담는 곳이다. arguments는 Object[] 형식으로 담아주면 된다.
- `defaulMessage`는 codes를 찾지 못 했을 때 보여줄 기본 디폴트 문구이다. 

<br>

그런데 이렇게 하면 필드에러 관련 코드가 조금 지저분해질 수 있다. 이것을 보완하기 위한 방법은 다음과 같다. 

<br>

### rejectValue, reject 활용

rejectValue, reject 를 사용하면 앞서 작성한 코드보다 더 간단하게 검증 처리를 할 수 있다.

<br>

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, BindingResult bindingResult, ...){
   if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.rejectValue("itemName", "required");
      }
		... (이하 생략)
}
```

<br>

```properties
required.item.itemName=상품 이름은 필수입니다. 
range.item.price=가격은 {0} ~ {1} 까지 허용합니다. 
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
```

<br>

아래는 검증 오류 메시지를 모아놓은 errors.properties이다. 위의 코드를 보면 bindingResult에 이제 addError를 써주지 않고 바로 rejectValue를 써주는 것을 확인할 수 있다. 만약 객체오류(글로벌오류)라면 reject를 써주면 된다.

<br>

`bindingResult.rejectValue("itemName", "required");` 그런데 이 부분을 보면 검증 오류 메시지인 `required.item.itemName`을 다 써주지 않고 required만 써준 것을 확인할 수 있다. 이것이 어떻게 가능한 것인가?

<br>

이것을 가능하게 하는 것은 스프링이 제공하는 `MessageCodesResolver`라는 인터페이스 때문이다. 

<br>

### MessageCodesResolver

MessageCodesResolver를 알아보기 이전에 검증 오류 메시지를 쓰는 전략에 대해서 간단하게 보자면 검증 오류 메시지를 적는 방법으로는 구체적으로 적는 것과 범용성있게 포괄적으로 적는 방식 두 가지가 있다. 
<br>

전자의 경우는 예를 들어 '상품의 이름은 필수입니다.' 라는 식으로 고정적으로 상품의 이름을 명시하는 방식이고 후자의 경우는 '필수 값입니다.' 라는 식으로 경우를 특정하지 않고 범용적으로 사용할 수 있는 검증 오류 메시지이다. 경우에 따라서 사용할 메시지가 달라지기 때문에 둘 다 모두 장단점이 있다. 

<br>

스프링에서는 구체적으로 쓴 오류 메시지가 범용적으로 쓴 오류 메시지보다 우선순위가 앞선다. 따라서 errors.properties에 `required.item.itemName=상품 이름은 필수입니다. `과 `required=필수값입니다. `가 모두 있다면 `required.item.itemName=상품 이름은 필수입니다.`가 우선 적용되는 것이다. 그렇다면 이것이 어떻게 가능한 것일까? 

<br>

**MessageCodesResolver**

MessageCodesResolver 는 인터페이스고 기본 구현체는 DefaultMessageCodesResolver 이다. DefaultMessageCodesResolver 는 기본 오류코드 생성 규칙이 있다. 

<br>

1. 객체 오류의 경우

   - code + "." + object name
   - code

   Ex)

   > 오류 코드: required, object name: item 
   >
   > Level 1: required.item
   > Level 2: required

   

   <br>

2. 필드 오류의 경우

   - code + "." + object name + "." + field
   - code + "." + field
   - code + "." + field type
   - code

   Ex)

   > 오류 코드: typeMismatch, object name "user", field "age", field type: int
   >
   > Level 1: "typeMismatch.user.age"
   >
   > Leve 2: "typeMismatch.age"
   >
   > Level 3: "typeMismatch.int"
   >
   > Level 4: "typeMismatch"



<br>

앞서 필드 에러 생성자를 보면 codes가 String[] 형태라고 살펴본 바 있다. DefaultMessageCodesResolver에 의해 생성된 코드들이 이 배열 안에 담기게 되는 것이다. 우선순위 순서대로 말이다. 결론적으로 rejectValue & reject와 MessageCodesResolver 의 이런 특성을 이용하여 위의 예시 코드처럼 간단하게 사용할 수 있는 것이었던 것이다. 코드에 들어갈 키워드만 입력해주면 Level 1 ~ Level 4에 해당하는 것이 주루룩 만들어지고 그것이 배열에 자동으로 담기고 스프링은 그 배열의 인덱스대로 화면에 표시하기 때문에 코드에 들어갈 키워드만 넣어주면 됐던 것이다. 

<br>

## 검증 오류 메시지를 이렇게 처리하는 이유

검증 오류 메시지를 이렇게 처리하는 이유는 유지, 보수 측면에서 유리하기 때문이다. 가령 모든 코드에 검증 오류 코드, 메시지를 일일이 다 써준다면 오류 코드나 메시지를 변경해야될 일이 있다면 모든 코드를 찾아서 일일이 수정해줘야 하는 번거로움이 있기 때문이다. 그러나 이렇게 properties 파일을 만들고 한 곳에서 관리하면 수정 요청이 들어와도 손쉽게 수정이 가능하다는 이점을 가질 수 있는 것이다. 

<br>

## Validator 분리

검증 로직을 컨트롤러에 다 넣으면 컨트롤러 쪽 코드가 너무 복잡해지기 때문에 좀 더 깔끔하게 코드를 작성하기 위해 검증 로직만을 처리하는 별도의 클래스를 만들고 거거에 검증 로직을 몰아놓는 것이 깔끔하다. 

<br>

Validator를 만들 때 스프링이 제공하는 Validator 인터페이스를 상속할 수 있다. Validator 인터페이스는 supports와 validate 두 가지 메소드를 가지고 있다. supports는 해당 Validator를 지원하는 여부를 확인할 수 있고 validate는 검증 대상 객체와 bindingResult를 매개변수로 받는다.

```java
public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}

```

<br>

이 Validator를 구현하는 클래스를 만들면 된다. 이 때 그 클래스에 @Component 어노테이션을 붙여줌으로써 빈으로 만들고 컨트롤러에서 주입 받아서 사용할 수 있다. 

<br>

이 때 직접 검증하는 클래스를 호출하는 방식도 있는데 스프링에선 `@Validated`어노테이션을 이용하여 직접 호출하지 않아도 자동으로 호출해주는 기능을 제공해준다. 최종적으로 완성된 코드는 다음과 같다.

<br>

```java
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult, ...) {
  ... (이하 생략)
}
```



