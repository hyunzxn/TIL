# HTTP 메시지 컨버터

JSON ↔︎ 객체 컨버팅을 도와주는 HTTP 메시지 컨버터에 관한 기록입니다.

<br>

## @ResponseBody

@ResponseBody를 사용하면 HTTP Body에 문자내용을 그대로 반환할 수 있다. 

<br>

원래 @Controller에서 String을 반환하면 뷰 논리네임으로 인식하여 viewResolver를 호출해서 view를 반환하는데 @ResponseBody 혹은 @RestController를 사용하면 HTTP Body에 String 내용을 그대로 넣을 수 있다. 

<br>

- 기본 문자 처리를 위한 HTTP 메시지 컨버터: **StringHttpMessageConverter**
- 기본 객체 처리를 위한 HTTP 메시지 컨버터: **MappingJackson2HttpMessageConverter**

<br>

## Spring MVC에서 HTTP 메시지 컨버터 사용

- HTTP 요청: **@RequestBody, HttpEntity(RequestEntity)** 사용
- HTTP 응답: **@ResponseBody, HttpEntity(ResponseEntity) 사용**

<br>

HTTP 메시지 컨버터는 HTTP 요청과 HTTP 응답 두 가지 모두에 사용된다.

<br>

## 주요 HTTP 메시지 컨버터

- **ByteArrayHttpMessageConverter**: byte[] 데이터를 처리
- **StringHttpMessageConverter**: String 데이터 처리
- **MappingJackson2HttpMessageConverter**: application/json 처리

<br>

**StringHttpMessageConverter**

```java
content-type: application/json
 
@RequestMapping
void hello(@RequetsBody String data) {}

```

<br>

**MappingJackson2HttpMessageConverter**

```java
content-type: application/json

@RequestMapping
void hello(@RequetsBody HelloData data) {}
```

<br>