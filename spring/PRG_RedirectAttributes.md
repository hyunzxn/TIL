# PRG

**PRG란**

POST Redirect GET 의 약자로 서버에 데이터를 생성하는 POST 요청을 보낸 뒤 새로고침을 했을 때 계속해서 데이터가 생성되는 현상을 해결하기 위한 방법이다. 

<br>

**왜 이런 현상이 발생하는 것일까?**

웹 브라우저의 새로고침은 서버에 전송한 마지막 데이터를 다시 서버에 전송한다. 즉 URL이 바뀌지 않은 상태에서 새로고침을 하면 계속해서 마지막에 보낸 데이터가 자동적으로 서버에 전송되면서 데이터가 추가가 되는 것이다. 

<br>

**해결방법**

데이터를 저장하는 POST 요청을 한 뒤에 리다이렉트를 해주는 것이다. 아예 새로운 URL 요청으로 리다이렉트를 해주게 되면 마지막으로 요청한 내용이 리다이렉트를 해준 URL이 되기 때문에 새로고침을 하더라도 의도하지 않은 데이터가 서버에 전송되는 것을 막을 수 있다. 예를 들어 아예 다른 화면으로 GET 요청을 해주게 된다면 마지막으로 서버에 요청한 내용은 GET 요청 결과로 반환 받은 결과가 되는 것이다. 

<br>

# RedirectAttributes

RedirectAttributes는 리다이렉트를 해줄 때 특정한 값을 담아서 그것들을 화면에 렌더링할 때 활용할 수 있게 도와주는 것이다. 

<br>

Ex) 

**Controller 코드**

```java
@PostMapping("/add")
public String addItem(Item item, RedirectAttributes redirectAttributes) {

		Item savedItem = itemRepository.save(item);
		redirectAttributes.addAttribute("itemId", savedItem.getId());
		redirectAttributes.addAttribute("status", true);

		return "redirect:/basic/items/{itemId}";
}
```

<br>

-  item 객체의 아이디를 redirectAttributes에 담는다. 담아줄 때 name을 설정해주면 `redirect:/basic/items/{itemId}` 이 부분에서 itemId 자리에 추가한 값이 치환되어 들어간다.
- 그렇다면 남아있는 다른 attribute 값인 ("status", true) 이것은 어떻게 처리되는 것일까? 이것은 리다이렉트 URL 뒤에 쿼리파라미터 형식으로 들어가지게 되는 것이다. 

<br>

결론적으로 완성된 URL의 모습은 다음과 같다. `/basic/items/{itemId}?status=true` (itemId 자리에는 itemId가 그 때 그 때 바뀌면서 들어감)

<br>

그렇다면 이렇게 넣어놓은 attribute 값들은 어떻게 사용할 수 있는 것인가?

<br>

**Thymeleaf 코드**

```html
<h2 th:if="${param.status}" th:text="'저장 완료'"></h2>
```

<br>

- `th:if="${param.status}"` param 이라는 것은 쿼리파라미터를 의미한다. 타임리프에서 자주 써서 아예 param 이라고 만들어 놓았다. 이 코드는 param.status가 true인 경우에 `th:text="'저장완료'"` 라는 코드를 텍스트로 보여주라는 의미이다. 