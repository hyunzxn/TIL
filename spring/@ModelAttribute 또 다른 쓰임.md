# @ModelAttribute 의 또 다른 쓰임

@ModelAttribute를 컨트롤러 안의 별개의 메소드 차원에 붙여주면 어떻게 되는지를 기록하고자 한다.

<br>

## @ModelAttribute 를 메소드에 붙이면 어떻게 되는가?

<br>

```java
Map<String, String> regions = new LinkedHashMap<>();
regions.put("SEOUL", "서울");
regions.put("BUSAN", "부산");
regions.put("JEJU", "제주");
model.addAttribute("regions", regions);
```

<br>

이런 코드가 있다고 하자. 이 코드를 필요로 하는 컨트롤러 안의 모든 메소드 안에 반복해서 써주는 것은 중복된 코드가 너무 많아지기 때문에 좋지 않다. 이렇게 중복되는 코드가 있을 때 @ModelAttribute를 사용하면 중복을 피할 수 있다.

<br>

```java
@ModelAttribute("regions")
public Map<String, String> regions() {
  Map<String, String> regions = new LinkedHashMap<>();
  regions.put("SEOUL", "서울");
  regions.put("BUSAN", "부산");
  regions.put("JEJU", "제주");
  return regions;
}
```

<br>

@ModelAttribute를 메소드 차원에서 사용해주면 이 메소드가 리턴하는 값을 컨트롤러의 모든 메소드가 호출될 때 자동으로 model 담아준다. 이 경우엔 regions라는 Map이 될 것이다. 

<br>

`model.addAttribtue("regions", regions)`가 모든 메소드에서 자동으로 들어가게 되는 것이다. attributeName은 @ModelAttribute 옆에 있는 괄호에 적혀진 것으로 들어가게 된다. 이렇게 되면 중복을 피할 수 있고 바로 모델에서 데이터를 뽑아서 사용할 수도 있다.