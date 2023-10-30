---
layout: post
title:  "Mono.error() 와 Throw 의 차이"
date:   2023-10-23 21:00:00 +0900
categories: Java WebFlux Reactive Exception
comments: true
---

# Mono.error() 와 Throw 는 언제 어떻게 써야할까?

WebFlux 로 비즈니스 로직 개발 시 error handling 은 어떻게 해야 할까? 로직 한 가운데에서 throw 해버릴까? 아님 잘 손질해서 return Mono.error() 를 할까? 이 둘의 차이는 뭘까?

결론은 ‘둘의 큰 차이는 없다’ 이다. 그런데 뭔가가 다르다, 뭔가가…

# Upstream 과 Downstream

Upstream(상류) 과 downstream(하류) 은 개발에서 다양한 개념으로 이용되는 표현이지만, reactive 에서 pipeline 의 흐름을 잘 설명해주는 표현이다. 그런데 단순히 상/하류로 표현하기에는 upstream 은 상류로 ‘올라가는’ 느낌이 있다. 실제로 사전에서도 ‘거슬러올라가다’라는 형용사적 표현이 있다. Pipeline 로직이 수행될 때 먼저 수행되는 부분, 또는 먼저 수행될 부분으로 호출되어 들어가는 것을 upstream 으로 보고, 반대로 나중에 반응하여(reacitve) 수행되는 부분을 downstream 으로 볼 수 있다.

아래와 같은 코드가 있다고 가정하자. Mono.just() 로 부터 시작하는 코드는 ②, ③ 으로 흘러(downstream)간다. 그렇지만 ① 은 pipeline 이 시작되기 전에 먼저(upstream) 수행된다.

```java
Mono.just(①).map(②).flatMap(③).block();
```

조금 다른 코드를 보자.

```java
public Mono<String> getUserById(String userId) {
  ...
  return userService.getUser(userId);
}
```

위 예시에서는 userService 가 user 정보를 반환하면서 stream 이 시작되므로 userService.getUser() 는 getUserById() 에게 있어서 upstream 이다.

```java
public Mono<String> makeNickname(Mono<String> name) {
  ...
  return name.map(this::decorate);
}
```

위 예시에서는 name 이란 stream 을 그대로 이어받아 흘려보내므로 this.decorate() 는 makeNickname() 에게 있어서 downstream 이다.

왜 갑자기 이러한 흐름의 방향을 이야기하냐면, throw 와 Mono.error() 는 이 stream 에서 다르게 동작하기 때문이다.

# Exception 이 없는 Mono/Flux

WebFlux 에서 Mono 와 Flux 는 펑터(functor)이다. 펑터는 제네릭(generic)과 유사한데, 데이터 타입을 감싸는 것으로 데이터 타입에 동일한 동작을 제공해준다. 

- Lifting 과 functor([https://overcurried.com/귀차니스트를 위한 펑터/](https://overcurried.com/%EA%B7%80%EC%B0%A8%EB%8B%88%EC%8A%A4%ED%8A%B8%EB%A5%BC%20%EC%9C%84%ED%95%9C%20%ED%8E%91%ED%84%B0/))
- 함수의 합성([https://evan-moon.github.io/2020/01/27/safety-function-composition/](https://evan-moon.github.io/2020/01/27/safety-function-composition/))

일반적인 Java 로직 내에서 throw 가 발생하면 호출한 곳으로 에러가 전파(bubbling) 되는데, WebFlux 의 downstream 에서는 Mono 펑터가 에러를 잡아 return 값으로 만든다. 즉, 에러나 실패도 pipeline 으로 흘려보내는 것이다. 그리고 이 pipeline 에서 doOnError(), onErrorXXX() 같은 연산자들이 에러를 맡아 처리하게 된다.

그러나 만약 pipeline 시작 전에 에러가 발생(보통 최상위 upstream 에서 에러가 발생)한다면, Mono 펑터로 에러를 집어넣지 못하고 에러를 전파시켜 버린다. 즉, 이 때는 runtime exception 이 발생하여 전파되게 된다.

## 예시

아래 코드는 upstream 에서 throw 가 발생했으므로 Mono 로 에러가 처리되지 않는다.

```java
public Mono<String> someMethod(String param) {
  ...
  throw RuntimeException();
}
...
someMethod("test")
  .doOnError(System.out::println); // 동작하지 않음
```

그러나 아래 코드는 downstream 에서 throw 가 발생했으므로 Mono 로 처리된다.

```java
public Mono<String> someMethod(Mono<String> param) {
  ...
	return param.map(p -> {
    throw RuntimeException();
  });
}
...
someMethod(Mono.just("test"))
  .doOnError(System.out::println); // 동작함
```

# Throw 를 Mono 로 감싸면 Mono.error()

결론적으로 말하자면 pipeline 로직 내에서 발생한 throw 는 Mono/Flux 펑터에 의해 감싸져 일반 return 값이 된다. 그것이 Mono.error() 인 것이다. 다만 어디서 감싸지느냐가 차이 - 내가 감싸서 반환하느냐 혹은 downstream 에서 감싸지느냐 - 에 따라 다른 것이다. 내가 만들고자 하는 pipeline 의 흐름에 대해 고민해보자.

에러 처리는 Mono 에 담아 pipeline 에서 처리하는 것이 좀 더 WebFlux 다운 구현이라고 생각된다. 기존 MVC 에서 쓰던 try ~ catch 는 WebFlux 에 어울리지 않는다. 

Mono.error() 를 쓰는 것이 Mono 로 반환해야 하는 메서드에서는 더 자연스럽다고 본다. 그러나 객체를 반환해야하는 메서드에서라면 throw 도 충분히 괜찮다. 단, 이 throw 가 Mono 로 감싸질 수 있도록 메서드를 downstream 에서 처리되도록 하자.

참고로 Mono.error() 의 함수 정의는 아래와 같이 “실패한 Mono” 다.

> Create a Mono that terminates with the specified error immediately after being subscribed to.
> 
> Params: error – the onError signal
> Returns: a failing Mono
> 

# 참조

- Throwing an exception vs Mono.error() in Spring webflux : [https://stackoverflow.com/a/69182985/8350542](https://stackoverflow.com/a/69182985/8350542)
- What is Upstream and Downstream in Software Development?[https://reflectoring.io/upstream-downstream/](https://reflectoring.io/upstream-downstream/)
- Explain about downstream and upstream in rxJava : [https://stackoverflow.com/a/53921697/8350542](https://stackoverflow.com/a/53921697/8350542)
- [https://itvillage.tistory.com/entry/Hello-Reactor-들여다-보기](https://itvillage.tistory.com/entry/Hello-Reactor-%EB%93%A4%EC%97%AC%EB%8B%A4-%EB%B3%B4%EA%B8%B0)