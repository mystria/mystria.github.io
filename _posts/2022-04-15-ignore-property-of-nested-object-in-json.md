---
layout: post
title:  "JSON 에서 nested object 의 property 제외하기"
date:   2022-04-15 17:00:00 +0900
categories: JSON Java Spring
comments: true
---

# 클래스를 serialize 할 때, property 를 제외해 보자

Spring 으로 REST(HTTP) API 를 구현할 때, 또는 JSON 으로 저장하는 DB 를 이용할 때, JSON serialize 시 특정 property 는 제외하고 싶은 경우가 있다.

Property 나 getter() 에 개별로 적용 할 때는 @JsonIgnore 를 사용하면 되고 클래스 단위에서 property 를 제외하고 싶다면 @JsonIgnoreProperties 를 사용할 수 있다.

그런데 만약 nested object 에서 특정 property 를 제외하고 싶다면? 

# 개요

아래와 같은 class 구조의 object 를 JSON 으로 serialize 할 때, 

```java
public class User {
  public String name;
  public Integer age;
  public Contact contact;
}

public class Contact {
  public String phoneNumber;
  public String email;
}
```

다음과 같이 serialize 가 된다.

```json
{
  "name": "Kim",
  "age": 99,
  "contact": {
    "email": "hello@gmail.xx",
    "phoneNumber" : "000-0000-0000"
  }
}
```

내부 object 의 일부 property 들(age, phoneNumber)을 아래 JSON 처럼 제외하고 표시하려면?

```json
{
  "name": "Kim",
  "contact": {
    "email": "hello@gmail.xx"
  }
}
```

제약사항을 하나 추가해서 우리는 User 클래스나 Contact 클래스를 수정할 수 없다고 가정한다. 그렇다면 “age” 같이 직접적인 property는 @JsonIgnoreProperties 를 이용하여 클래스 “밖”에서 해결 가능하다.

```java
@JsonIgnoreProperties({"age"})
public class User {
  public String name;
  public Integer age;
  public Contact contact;
}
```

그러나 아래처럼 nested 된, “contact.phoneNumber”는 제외가 불가능하다.

```java
@JsonIgnoreProperties({"contact.phoneNumber"}) // 불가능
public class User {
  public String name;
  public Integer age;
  public Contact contact;
}
```

즉, 클래스 밖에서는 @JsonIgnoreProperties 를 이용해 직접적인 영향이 닿는 해당 object 의 property 만 제외할 수 있다.

누군가 FasterXML 측에 해당 기능을 넣어달라고 요청했는데, 구조적인 이유로 불가능하다고 결론이 났다. (2020년 12월) - [https://github.com/FasterXML/jackson-databind/issues/2940](https://github.com/FasterXML/jackson-databind/issues/2940)

대신 add-on 을 추가한 후 필터로 적용할 방법을 안내하고 있다.  

- [https://github.com/FasterXML/jackson-databind/issues/690](https://github.com/FasterXML/jackson-databind/issues/690)
- [https://github.com/Antibrumm/jackson-antpathfilter](https://github.com/Antibrumm/jackson-antpathfilter)

# Add-on 없이 nested object 의 property 제외 방법

다양한 방법들이 존재하지만, 쉬운 길은 없었다. 

## 어떻게든 되게할 방법들

1. JsonSerialize를 이용해 custom serializer 를 적용하는 방법
    - 간단하게 annotation으로 해결하려다가 배보다 배꼽이 더 커짐 : [https://stackoverflow.com/a/51570918/8350542](https://stackoverflow.com/a/51570918/8350542)
    - Custom serializer에 대한 튜토리얼 : [https://www.baeldung.com/jackson-custom-serialization](https://www.baeldung.com/jackson-custom-serialization)
2. 안쓰는 값을 null로 바꿔버린 후 Include.NON_NULL 적용하는 방법
    - 간단하긴 하지만, 다소 극단적인 꼼수 : [https://stackoverflow.com/a/53412240/8350542](https://stackoverflow.com/a/53412240/8350542)

## 정석으로 해결할 방법들

1. JsonView를 이용하는 방법
    - [https://stackoverflow.com/a/43630545/8350542](https://stackoverflow.com/a/43630545/8350542)
2. Mix-in 을 이용해 property를 추가하는 방법
    - [https://stackoverflow.com/a/58670226/8350542](https://stackoverflow.com/a/58670226/8350542)

## 관점을 바꾸기

1. 접근을 반대로하여, 뺀 다음에 추가하는 방식
    - @JsonAppend를 사용해야 함 : [https://stackoverflow.com/a/42455398/8350542](https://stackoverflow.com/a/42455398/8350542)
    - 위의 custom serializer 만큼 손이 많이 감

# 또 다른 방법

Mix-in 과 관점 바꾸기 방법에서 힌트를 얻었는데, nested object 의 이름을 무시하는 방법. 분해 - 재조립 방법으로도 해결 가능하다.

- [https://stackoverflow.com/a/55762195/8350542](https://stackoverflow.com/a/55762195/8350542)

@JsonUnwrapped 를 이용하면 nested object 의 property 들을 한 단계 위로 꺼낼 수 있다. 즉, 새로운 object 를 만들 수 있다는 뜻이다. 아래의 예시에서 UserWrapper 는 User 와 동일한 결과를 반환한다.

```java
public class UserWrapper {
  @JsonUnwrapped
  public User user;
}
```

이를 응용하여 user 와 contact 를 외부로 꺼낸다.

```java
public class UserWrapper {
  @JsonUnwrapped
  @JsonIgnoreProperties({"age", "contact"})
  public User user;

  @JsonIgnoreProperties({"phoneNumber"})
  public Contact contact;  // 단, 여기에 user 의 contact 를 할당해야 함
}
```

위의 결과는 우리가 원했던 바로 그것!

```json
{
  "name": "Kim",
  "contact": {
    "email": "hello@gmail.xx"
  }
}
```

아니, 잠깐! 그런데 이렇게 하면 그냥 별도의 response 용 object 를 만들어 매핑한 것과 뭐가 다르냐는 비판이 있을 수 있다. 하지만 User 와 Contact 클래스를 우리가 수정할 수 없는 상황에서, 그에 대한 deep copy 를 만드는 것은 부담이 될 수 있다. User 나 Contact 클래스의 property 가 변경되면 wrapper 도 변경해야 하기 때문이다.

# 결론

그런데 왜 이렇게까지 해야하나? 그냥 처음부터 클래스의 property 에 @JsonIgnore 잘 적용하면 되지 않느냐? 싶겠지만... 살다보면 어쩔 수 없는 경우들이 종종있다. 그럴 때 사용해 볼 수 있는 변칙 아이디어라고 생각하자.

# 참조

- JsonIgnore와 JsonIgnoreProperties 설명
    - [https://www.codekru.com/java/how-to-use-jsonignoreproperties-and-jsonignore](https://www.codekru.com/java/how-to-use-jsonignoreproperties-and-jsonignore)
    - [https://www.logicbig.com/tutorials/misc/jackson/ignoring-properties.html](https://www.logicbig.com/tutorials/misc/jackson/ignoring-properties.html)