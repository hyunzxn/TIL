# 인텔리제이 application.properties 한글 깨짐 문제 해결

application.properties에 한글을 쓰려고 하니까 한글이 깨지면서 ??? 이렇게 보였다. 이것에 대한 해결법을 적고자 한다.



## 해결법

- **Preferences > Editor > File Encodings > Properties Files** 항목으로 이동
- Default encoding for properties files을 **UTF-8**로 변경
- **Transparent vative-to-ascii conversion** 도 체크

<br>

<img width="985" alt="세팅" src="https://user-images.githubusercontent.com/100478841/184290380-9c57b50e-a5b3-431d-8de0-ea73be3327c7.png">





