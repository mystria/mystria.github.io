---
layout: post
title:  "Kotlin 매개변수의 기본 값의 재미있는 점"
date:   2025-11-16 10:00:00 +0900
categories: Kotlin
comments: true
---

너무 사소해서 그냥 지나칠 수 있지만 생각해보니 재미있는 Kotlin 의 특징

# 매개변수의 기본 값

아래와 같이 메서드 정의 시 매개변수(parameter)에 기본 값(default value)을 지정해 두면, 메서드 호출 시 별도로 매개변수를 전달하지 않으면 기본 값으로 호출 됨
~~~ kotlin
fun method1(a: Int = 0, b: Int = 0) {
  println("a = $a, b = $b")
}

fun main() {
  method1()
}
// a = 0, b = 0
~~~

## 재미있는 부분

여러개의 매개변수가 있을 때, 일부만 넣으면 순서대로 할당됨
~~~ kotlin
fun method1(a: Int = 0, b: Int = 0) {
  println("a = $a, b = $b")
}

fun main() {
  method1(3) // 3 은 a 에 할당됨
}
// a = 3, b = 3
~~~

두 번째 이 후의 매개변수의 기본 값은 이전 매개변수를 사용할 수 있음
~~~ kotlin
fun method1(a: Int = 0, b: Int = a) { // b 의 기본 값은 a
  println("a = $a, b = $b")
}

fun main() {
  method1(3)
}
// a = 3, b = 3
~~~

역순은 안됨
~~~ kotlin
fun method1(a: Int = b, b: Int = 0) {
  println("a = $a, b = $b")
}

fun main() {
  method1(b = 3)
}
// Parameter 'b' is uninitialized here.
~~~

## 이유

Java 로 디컴파일한 method1 의 코드를 보면 다음과 같음 (읽기 쉽게 수정)
~~~ java
public static void method1(int a, int b, int v) {
  // 인자 전달 여부에 따라 v 값을 조정
  if ((v & 1) != 0) { // a 값을 전달하지 않을 경우
    a = 0;
  }
  if ((v & 2) != 0) { // b 값을 전달하지 않을 경우
    b = a;
  }
  System.out.println("a = " + a + ", b = " + b);
}
~~~

Kotlin 을 Java 로 바꿀 때 `v` 값 처럼 인자 전달 여부를 정의하는 로직이 존재

여기에서 매개변수 순서를 중요하게 여기는 것으로 추정되며, 실제로 실패하는(`a = b`) 코드는 바이트코드로 생성이 안됨
~~~
// Backend Errors: 
// ================
// Error at Sample.kt(-,-): Parameter 'b' is uninitialized here.
// ================
~~~

전체가 아닌 매개변수 일부만 전달했을 때 순서대로 처리하기 때문에, 순서를 중요하게 여기는게 아닐까 싶음

# 참조
- https://kotlinlang.org/docs/functions.html#parameters-with-default-values