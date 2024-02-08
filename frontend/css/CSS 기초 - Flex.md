# Intro

**div 의 display 속성의 종류**

1. block

2. inline

3. inline-block

- div의 display 속성의 default value는 `block` 이다.
- `block` 은 부모 요소의 `width` 를 100% 전부 다 차지하기 때문에 다른 요소가 옆에 올 수 없다.
- `inline` 은 다른 요소를 옆에 둘 수 있다. 그러나 `width`, `height` 를 가질 수 없다.
- `inline-block` 은 `block` 처럼 `width`, `height` 다 가질 수 있고 `inline` 처럼 다른 요소를 옆에 둘 수도 있다.

<br>

**flexbox 등장 이유**

`flexbox` 가 없이 요소를 정렬하려면 일일이 margin을 사람이 계산해서 배치해야 할 뿐 아니라(이런 방식을 `명령형 방식` 이라고 한다) 사용자의 디바이스 화면 크기에 따라 요소가 다시 정렬될 때 개발자가 원하는 대로 배치되지 않을 수 있다.

반면에 `flexbox` 를 사용하면 컴퓨터에게 요소를 배치할 위치만 알려주면 컴퓨터가 알아서 배치를 하게 만들 수 있다. 이런 방식을 `선언형 방식` 이라고 한다.

&nbsp;

# Flexbox

**대전제**

-  `flexbox` 를 사용할 때는 배치할 개별 요소가 아니라 배치할 개별 요소의 **부모 요소(Container)** 에 관심을 둔다.
- 모든 선언은 자식 요소가 아닌 컨테이너(부모)에게 한다. (단 드물게 자식 요소에 선언하는 경우가 있다.)

<br>

**Flexbox 에서의 axis**

- 컨테이너에서 자식 요소의 위치를 이동시키는 방향을 결정하는 속성은 `flex-direction` 이다. `flex-direction` 은 **컨테이너(부모)의 속성**이다. `flex-direction` 의 default value 는 `row` 이다. (수평 방향)
- `Flexbox` 에는 `main axis` 와 `cross axis` 두 가지 축이 있다. `main axis` 는 `flex-direction` 의 방향을 의미한다.  `cross axis` 는 `main axis` 와 교차되는 방향을 의미한다. 
  - `main axis` 와 `cross axis` 는 고정된 것이 아니다. `flex-direction` 에 의해 변동하는 것이다.
  - `flex-direction` 이 `row` 인 경우에는 `main axis` 는 수평, `cross axis` 는 수직이 된다.
    - 일반적인 `flex-direction: row` 인 경우에는 수평 방향 배치의 순서가 ⇨ 이렇게 왼쪽에서 오른쪽을 향하게 된다.
    - 그러나 `flex-direction: row-reverse` 인 경우에는 수평 방향 배치의 순서가 ⇦ 이렇게 오른쪽에서 왼쪽을 향하게 된다.
  - `flex-direction` 이 `column` 인 경우에는 `main axis` 는 수직, `cross axis` 는 수평이 된다.
    - 일반적인 `flex-direction: column` 인 경우에는 수직 방향의 배치의 순서가 ⇩ 이렇게 위에서 아래를 향하게 된다.
    - 일반적인 `flex-direction: column-reverse` 인 경우에는 수직 방향의 배치의 순서가 ⇧ 이렇게 아래에서 위를 향하게 된다.

<br>

**Flexbox 에서의 자식 요소 배치**

- 컨테이너 안에 있는 자식 요소를 `main axis` 방향으로 이동(배치)시키기 위해 사용하는 속성은 `justify-content` 이다.
- 컨테이너 안에 있는 자식 요소를 `cross axis` 방향으로 이동(배치)시키기 위해 사용하는 속성은  `align-items` 이다.
- 그렇다면 자식 요소를 수직 방향으로 이동(배치)시킬 때 기억해야 될 중요한 포인트가 무엇일까?
  - 바로 **컨테이너(부모)의 높이**다. `Flexbox` 는 컨테이너(부모) 안에서 자식 요소를 움직이는 것이기 때문에 수직 방향의 이동은 컨테이너(부모)의 높이가 중요하다.
  - 수직 방향으로의 이동은 항상 높이를 생각하고 있어야 한다.
- 컨테이너(부모) 안에 있는 자식 요소는 자식 요소 내부에 있는 또 다른 요소의 컨테이너(부모)가 될 수 있다.

<br>

**flex-wrap 속성**

- 만약에 컨테이너(부모)의 `width` 를 초과하는 자식 요소를 만들면 어떻게 될까? 예를 들어 컨테이너의 `width` 는 1000px인데 `width` 가 200px인 자식 요소를 10개를 만드는 식으로 말이다.
- 이런 경우에는 자식 요소의 `width` 가 자동적으로 줄어들면서 결과적으로는 컨테이너(부모) 안에 모두 10개가 들어가 있게 된다. 이것은 `Flexbox` 의 기본적인 속성 때문인데, `Flexbox` 는 기본적으로 컨테이너(부모) 안의 자식 요소들을 한 줄로 표시하려고 한다. 이 속성 때문에 자식 요소의 크기를 줄여가면서까지 한 줄로 표시하려고 하는 것이다.
- 이런 `Flexbox` 의 속성을 조절하기 위해서는 `flex-wrap` 이라는 속성을 사용하면 된다. `flex-wrap` 의 defatul value는 `no-wrap` 이다. 만약 이 속성을 `wrap` 으로 바꿔주게 되면 자식 요소의 크기가 정상적으로 돌아온다. 그리고 컨테이너의 `width` 를 초과하는 경우에는 **자동으로 줄바꿈**이 되면서 자식 요소가 배치되는 것을 확인할 수 있다.
- `flex-flow` 속성을 사용하게 되면 `flex-direction` 과 `flex-wrap` 을 한 번에 선언할 수 있다. ex) `flex-flow: row wrap;`

<br>

**align-content 속성**  

- `flex-wrap: wrap;` 속성이 적용돼서 다중 라인으로 자식 요소가 배치될 때 사용할 수 있는 특별한 속성이 있는데 바로 `align-content` 이다.
- 기본적으로 `Flexbox` 를 사용해서 자식 요소를 배치하면 컨테이너(부모) 단위로 움직이게 된다. 그래서 컨테이너(부모) 안에 다중 라인의 자식 요소가 있는 경우에 라인과 라인 사이의 간격을 조정할 수 없게 된다. 그런데 `align-content` 속성을 사용하게 되면 다중 라인의 경우에도 라인과 라인 사이의 간격을 조정할 수 있다. `align-content` 속성은 `flex-wrap` 속성을 `wrap` 으로 줘서 다중 라인으로 자식 요소가 배치될 때만 사용되는 속성이라는 것을 기억해두면 좋다.

<br>

**자식 요소에 바로 속성을 선언하는 경우**

- 일반적으로 `Flexbox` 의 대부분의 속성은 컨테이너(부모)에게 선언하지만 드물게 **자식 요소에게 바로 선언**하는 경우가 있다. 
  - **order**: 자식 요소의 배치 순서를 변경할 때 사용하는 속성이다.
    - `order` 속성을 주면 HTML을 수정하지 않더라도 배치 순서를 바꿀 수 있다. 그런데 `order` 속성을 준 자식 요소들은 맨 뒤로 가게 된다. 이것은 `order` 속성을 주지 않은 자식 요소들은 기본적으로 `order` 가 0 이기 때문이다. `order` 에는 음수를 줄 수도 있다.
  - **align-self**: 교차축 정렬을 정할 때 사용하는 속성이다.
    - `align-self` 속성은 컨테이너(부모) 안에 있는 자식 요소의 `cross axis` 방향으로 개별 정렬하게 해준다. 

<br>

**Flexbox 에서 Width를 지정하지 않고 넓이를 주는 법**

- 사용자가 사용하는 디바이스가 다양하기 때문에 요소의 크기를 px로 고정해서 준다면 어떤 곳에서는 너무 크고, 어떤 곳에서는 너무 작은 문제가 발생할 수 있다. 
- 이런 것을 방지하기 위해 px로 크기를 고정해서 주지 않아도 크기를 다르게 세팅할 수 있는 법이 있다. 바로 `flex-grow` 속성을 사용하면 된다.
- `flex-grow` 속성은 컨테이너(부모) 내부에 있는 자식 요소들에게 얼마만큼의 공간을 차지할 수 있는지를 알려주는 것이다. px이나 %를 사용하지 않고 비율만을 사용해서 표현한다.
  - `flex-grow` 의 속성의 defaul value는 0이다. 자식 요소 내부에 가지고 있는 컨텐츠에 딱 필요한 만큼의 공간만을 차지하는 것을 의미한다.
  - 예를 들어 A요소에 `flex-grow:1` 을 주고 B 요소에 `flex-grow:2` 를 주게 되면 B요소는 화면 크기에 상관없이 항상 A 요소의 2배의 `width` 를 차지하게 되는 것이다.
  - 만약 동일한 `width` 를 차지하게 하고 싶다면 동일 비율로 주면 될 것이다.
  - `flex-grow` 속성은 단독으로도 사용하지만 `flex-shrink` 라는 속성과 함께 사용하기도 한다.
  - `flex-shrink` 옵션은 아이템이 얼마나 줄어들지를 정할 수 있다. 비율, 순서 등을 조절하는 것도 가능하다. `flex-shrink` 의 default value는 1이다.

<br>

**Flex basis**

- `flex-basis` 속성은 아이템의 초기 `width` 를 결정한다.
- 그리고 만약 item의 크기에 변동이 생기면 초기 `width` 에서부터 변동이 된다. 다시 말해서 `flex-basis` 로 설정해둔 초기 크기가 다르다면 `flex-grow: 1;` 로 동일하게 설정을 해줘도 최종적인 크기는 다를 수 있다는 것이다. 
- 만약 `flex-grow: 0` , `flex-shrink: {value}` 속성을 주면 `flex-basis` 속성이 max-width 처럼 작용한다. `flex-grow` 가 0이기 때문에 아이템 내의 컨텐츠 크기 이상으로 더 이상 자라지는 않을 것이고 `flex-basis` 값이 있기 때문에 초기 크기는 가지고 있기 때문이다. 따라서 아무리 키워도 `flex-basis` 로 설정해 둔 크기 이상으로는 커지지 않을 것이기 때문에 max-width 를 설정해준 효과를 누릴 수 있는 것이다. 그러나 `flex-shrink` 속성의 값이 있기 때문에 축소 시에는 `flex-basis` 로 설정해 둔 크기보다 작아질 수는 있을 것이다.
- 그렇다면 반대로 `flex-grow: {value}` , `flex-shrink: 0` 로 설정을 해준다면 크기는 `flex-basis` 로부터 시작해서 계속해서 커지지만 크기는 아무리 줄여도 `flex-basis` 로 설정해 둔 초기 크기 미만으로는 작아지지 않을 것이다. 이 경우에는 `flex-basis` 가 min-width 처럼 작용하게 되는 것이다.
- 그리고 `flex-grow` 와 `flex-shrink` 와 `flex-basis` 를 한 번에 다 설정할 수 있는 속성이 `flex` 속성이다. 
- `flex-basis` 는 한 가지 또 특징을 가지는데 바로 `axis` 를 가질 수 있게 되는 것이다. 만약 `flex-direction` 이 `row` 라면 `flex-basis` 의 값은 `width` 가 되겠지만, `flex-direction` 이 `column` 이라면 `flex-basis` 의 값은 `height` 가 되는 것이다.