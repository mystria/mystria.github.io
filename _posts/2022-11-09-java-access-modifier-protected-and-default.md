---
layout: post
title:  "Java 접근 제한자 protected 와 default"
date:   2022-11-09 21:00:00 +0900
categories: Java
comments: true
---

# Access Modifiers

Java 의 접근 제한자(access modifier)에 대한 내용은 기본적인 것이며 여러 웹에서 충분히 다루어지고 있을 것이므로 자세한 설명은 생략한다.

### 참고 문서

- [https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html)
- [https://www.w3schools.com/java/java_modifiers.asp](https://www.w3schools.com/java/java_modifiers.asp)
- [https://www.baeldung.com/java-access-modifiers](https://www.baeldung.com/java-access-modifiers)

접근 제한자는 public / protected / default / private 4가지 있는데, 이들의 접근 가능 범위를 요약하면 아래 표와 같다.

### 접근 가능 범위

| Modifier | Class | Package | Subclass<br/>(same pkg) | Subclass<br/>(diff pkg) | World |
| --- | --- | --- | --- | --- | --- |
| public | Y | Y | Y | Y | Y |
| protected | Y | Y | Y | Y |  |
| default | Y | Y | Y |  |  |
| private | Y |  |  |  |  |

여기서 private 와 public 은 용도가 명확할 뿐 아니라 널리 사용되기 때문에 별다른 의문이 들지 않지만, protected 와 default 는 왜 존재하는지, 언제 어떤 경우에 사용하는지 궁금하여 내용을 찾아봤다.

# Protected 와 Default

## 전제

우리가 기본적으로 공감해야 하는 부분은 **클래스나 맴버의 접근을 최소한으로 제한해야 한다**는 것이다. 이러한 전제에 대해서는 수 많은 석학과 선배들이 논하고 있으니 따로 찾아보자. 가끔 모든 것을 public 으로 열어놓고 개발하는 극단적인 경우를 볼 수 있는데, 이러한 경우 처럼 ‘왜 굳이 빙빙 둘러가야 하느냐’고 생각한다면 이 글은 아무 의미가 없을 것이다.

웹에서 찾은 대다수의 문서들은 각 접근 제한자의 접근 가능 범위만 설명하고 있을 뿐 정답은 알 수 없었다. 본문 역시 단순한 의문에서 비롯하여 다른 사람들의 생각을 정리한 것이니 정답은 아니므로 참고만 하였으면 한다.

## 요약

- Protected : 상속 관계에서 활용
- Default : Package 단위로 캡슐화

## Protected 의 용도

Protected 는 애매한 것 같은데, 왜 쓰는 것일까? 명확한 답을 찾을 수 없었지만 여러가지 글들의 맥락상 캡슐화를 위해 “최대한 숨기고 조금 씩 공개하라”는 원칙을 위해 존재하는 것이었다.

1. Private 부터 시작
2. 필요한 만큼 최소한으로 공개 범위 확대

즉, protected 를 사용하면 package 내부나 subclass 에서만 접근할 수 있으므로, package 안에서만 사용될 맴버나 상속해서 사용할 때는 가급적 public 으로 열지 말라는 것이다. (이 때 package 내부용으로 default 도 가능하지만 이는 뒤에 다시 정리한다.)

아래 글에서 나름 정리된 답변을 얻을 수 있었는데, protected 는 상속에서 활용한다는 것이다.

> 외부에는 노출되지 않고 상속한 클래스에서만 접근할 수 있는 기능을 제공하기 위해, 또는 상속된 클래스가 부모 클래스에 기능을 제공하기 위해(= template pattern) 사용할 수 있다.
> 
- [https://stackoverflow.com/questions/17595224/when-i-need-to-use-a-protected-access-modifier](https://stackoverflow.com/questions/17595224/when-i-need-to-use-a-protected-access-modifier)
- [https://www.digitalocean.com/community/tutorials/java-access-modifiers](https://www.digitalocean.com/community/tutorials/java-access-modifiers)

그리고 아래 글에서 상속의 override 를 위한 것이다라는 의견도 볼 수 있었다.

> protected 는 잠재적으로 자식 클래스가 override 해서 바꾸어야 할 경우를 고려한 modifier이다. 즉 완성되지 못한, 혹은 완성될 수 없는 클래스 멤버를 의미한다.
> 
- [https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=2feelus&logNo=220576845725](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=2feelus&logNo=220576845725)

정리하자면 protected 의 개성은 package 범위 보다 subclass 범위에서 의미가 있으며, 상속을 통해 부모-자식 간의 통신이 필요한 경우에는 protected 를 사용한다는 것이다. 

## Default 의 용도

그렇다면 default 는 (실수로 빠뜨리고 작성하는 경우를 제외하면) 언제 사용하는가? Default 는 package-private 또는 no-modifier 라고도 한다. 이름처럼 package 범위에서 사용할 수 있는 private 으로 볼 수 있다. 앞서 언급한 protected 처럼, “최대한 숨기고 조금 씩 공개하라” 정책에 따라 package 내에서만 통신할 수 있다면 public 보다 default 를 고려해 볼 수 있을 것이다.

아래 글에서도 default 는 package 내의 클래스 간의 통신을 할 수 있도록 사용되기도 하지만, 그런 점에서 잘 사용 되지 않기도 한다. 그래서 사실상 private 보다 조금 넓은 범위에서 쓰는 용도 밖에 없어 보인다고 말한다.

> 약간 더 넓은 범위의 private 이다.
> 
- [https://stackoverflow.com/questions/6470556/pros-and-cons-of-package-private-classes-in-java](https://stackoverflow.com/questions/6470556/pros-and-cons-of-package-private-classes-in-java)

> 외부에 공개할 필요 없이, 같은 패키지에 있는 클래스 간의 밀접한 동작을 위해 서로의 내부를 보고자 할 때 이용될 수 있다.
> 
- [https://softwareengineering.stackexchange.com/questions/247118/when-to-use-default-access-modifier](https://softwareengineering.stackexchange.com/questions/247118/when-to-use-default-access-modifier)

정리하자면 package 단위로 제공되는 기능(ex: 라이브러리)에서 외부 API 는 public 으로 개방하지만 내부에서는 default 로 클래스간의 통신을 하게끔 활용한다는 것이다.

### 변칙 활용

관련하여 여러 글을 살펴보면, 캡슐화를 유지하면서도 쉽게 단위 테스트 하기 위해 default(또는 protected) 를 선언한다고 한다. 테스트 코드를 같은 package 에 배치하면 public 처럼 단위 테스트 할 수 있지만 외부에서는 사용하지 못하게 막을 수 있다. 언뜻 캡슐 속 메소드를 단위 테스트 하기에 적합해 보이기도 한다. 

- [https://softwareengineering.stackexchange.com/questions/220053/why-did-java-make-package-access-default](https://softwareengineering.stackexchange.com/questions/220053/why-did-java-make-package-access-default)

하지만 당연하게도 그러면 안된다. (이유는 처음 전제했던 내용을 참고)

- private 을 직접 테스트 하지마라 :
[https://softwareengineering.stackexchange.com/questions/100959/how-do-you-unit-test-private-methods](https://softwareengineering.stackexchange.com/questions/100959/how-do-you-unit-test-private-methods)

## 정리

Protected 와 default 의 활용 사례를 찾다보면 느껴지는 맥락은 “적절한 접근 제한자를 사용하라” 이다. 코드가 스파게티가 되는 이유 중 하나는 public 의 남용일 것이다. 이 같이 적절하지 못한 접근 제한자를 사용할 수 밖에 없는 상황이 온다면, 아마 StackOverflow 의 금언 처럼 **내가 설계를 잘못해서** 그럴 것이다.  

# 참고

- Default(no modifier) 는 상황에 따라 접근 가능 범위가 조금씩 다를 수 있다.
    - [https://stackoverflow.com/a/3530161/8350542](https://stackoverflow.com/a/3530161/8350542)