---
layout: post
title:  "금주의 실패사례 - Java Lambda 표현에서 reversed가 안될 때"
date:   2021-02-10 21:00:00 +0900
categories: Java
comments: true
---

# Java Lambda 표현에서 reversed가 안될 때
Java Lambda 에서는 collection 을 효과적/직관적으로 map, filter, sort 등을 할 수 있다.  
목록을 정렬할 때는 `.sorted()` 를 적용하면 되는데, 이때 정렬할 요소(elements)가 comparable 하지 않다면 Comparator 로 별도의 정렬 기준을 정해줘야 (이 기준도 comparable 해야 함) 한다.  
그리고 이 Comparator 를 역순으로 만들고자 할 때는 간단하게 `.reversed()` 를 적용해주면 되는데...  
라고 생각했지만 약간의 이해와 추가작업이 더 필요하다.

## 주의
본 글이 작성되었을 시점에는 컴파일 에러가 발생하였으나, 현재는 IDE 단에서 메소드 참조로 바꾸라고 하는 식으로 제안하거나  
> Cannot resolve method 'getName' in 'Object'
>

와 같은 에러를 발생하여 처음부터 컴파일 되지 않을 것임을 안내해 준다.

## reversed() 안될 때
* 증상
  + sorted() 할 때 Comparator 에 reversed()를 적용하면 compile 에러 발생
  + 같은 문제를 겪은 사람
    + [StackOverflow: Comparator.reversed() does not compile using lambda](https://stackoverflow.com/questions/25172595/comparator-reversed-does-not-compile-using-lambda)
* 해결
  + comparing 할 때 타입을 잃어버림, 타입을 consumer에서 정의 해주면 해결
  + https://stackoverflow.com/a/25173599 답변을 요약
    + 컴파일러의 타입 추론 메커니즘의 약점, lambda에서 타입을 추론하기 위해서 대상 타입 설정 필요
    > userList.sort(Comparator.comparing(u -> u.getName()).reversed()); // Compiler error
    >
    + sorted()는 Comparator.comparing()에게 Comparator\<User\> 같은 타입을 기대
    + 즉, Comparator.comparing()에게 User타입을 주는 Function이 필요하다는 뜻
    + 그런데 reverse()에서 대상 타입 설정이 막혀버림(이유는 알 수 없음)
    + 이 때 **메소드 참조(method reference)** 를 이용해 타입을 전달해주면 해결 가능
    + 그러나 메소드 참조를 이용할 수 없을 땐(추가 파라메터가 있다거나), 다음과 같이 타입을 명시하여 해결해야 함
    > Comparator.comparing(**(User u)** -> u.getName()).reversed()
    >

* 샘플 코드
  ~~~ java
  users.stream().sorted(Comparator.comparing((User u) -> u.getName()).reversed());
  or
  users.stream().sorted(Comparator.comparing(User::getName).reversed());
  ~~~

* 참고
  + 메소드 참조: String::valueOf 같은 표현식, 함수를 호출 하는 것이 아닌 참조할 수 있도록 전달, https://www.baeldung.com/java-method-references
  + https://stackoverflow.com/questions/50916403/java-8-comparator-chaining-with-a-reverse-on-single-property

