---
layout: post
title:  "Coding Test 준비하기 with Java"
date:   2023-12-06 21:00:00 +0900
categories: Java
comments: true
---

# Java 로 코딩 테스트 준비
요즘은 많이들 Python으로 코테(coding test)를 준비하지만, Java가 더 편하거나 Java로 쳐야하는 경우도 있다. 사실 for-loop 와 if 구문만 사용해서 바닥부터 한땀 한땀 쌓아 올릴 수 있지만, 진짜 중요한 알고리즘 구현을 위해 간단한 로직은 이미 있는걸 쓰자! (Don't reinvent the wheel!)  
본문에서는 알고리즘 풀이법이 아닌, 미리 알아두면 좋은 간단한 Java 용법을 정리해둔다. 놀랍게도 평소 개발할 때는 잘 안쓰이는 용법도 많다.

# 기본

일단 java.util 을 import 해야 한다. 자동으로 되어 있는 경우도 있다. 문제를 살펴보고 List, Map 같은게 필요해 보이면 일단 추가하고 보자.

```java
import java.util.*;
```

IDE 에서 자동으로 만들어주는 sysout 코드, 미리 준비 안하면 기억이 안날 수 있다.

```java
System.out.println("Hello World!");
```

Stream() 이나 Lambda 식은 아직 for-loop 보다 약간 느리다. 그리고 복잡한 로직에서는 가독성이 떨어질 수 있다. 간단한 부분에서만 사용하자.

# 자료 구조

## 배열과 리스트

대부분의 알고리즘은 배열로 시작해서 배열로 끝난다.

### Assign

Integer 처럼 class type이 아니라 int 같은 primitive type을 쓰면 기본 0이 assign된다(boolean이라면 false). 이 점을 이용하면 array 생성 시 값을 굳이 초기화 하지 않아도 된다. 만약 하고 싶다면 Arrays.fill() 을 사용하자. 안그러면 for-loop로 값을 넣어줘야 한다.

```java
int[] a = new int[3]; // a[0], a[1], a[2] 모두 0
Arrays.fill(a, 1);    // a[0], a[1], a[2] 모두 1
```

고정된 값이라면 배열을 생성하면서 즉시 값 assign 할 수 있다.

```java
int[] a = new int[] {1, 2, 3};
int[][] b = new int[][] { {1, 2, 3}, {4, 5, 6}, {7, 8, 9} };
```

그리고 Arrays.copyOf() 를 사용하여 배열의 크기를 늘이거나 줄일 수 있다. 또한 sub-string 을 구하고자 할 때는 Arrays.copyOfRange() 를 사용할 수 있다. 이 방법을 쓰지 않으면 for-loop로 한땀 한땀 늘이고 줄이고 복사해야 한다. 

Arrays.copyOf() 를 사용하면 deep copy 가 되는 점도 활용할 수 있다.

```java
int[] a = new int[3];                    // a[0] ... a[2]
int[] a = Arrays.copyOf(a, 6);           // a[0] ... a[5]
int[] sub = Arrays.copyOfRange(a, 2, 4); // a[2] ... a[4]

// Stream 으로 특정 항목 제거
String[] newArray = Arrays.stream(originalArray)
    .filter(item -> !item.equals(itemToExclude))
    .toArray(String[]::new);
```

### 2차원 배열

가장 귀찮고 헷갈리는 요소 중 하나이다. 배열이 배열로 구성된 구조인데, 어디가 안이고 바깥 배열인지 헷갈릴 수 있다. 보통 미로 찾기 같은 문제에서 활용되는데, 이를 x, y 좌표로 이해하려 하면 y 축이 반대가되어 헷갈리므로 Excel sheet 를 상상하는 것이 조금 더 이해하기 쉽다.

```java
int[][] arrays = new int[10][10];
String[][] sheet = {
		{"a1", "b1", "c1"},
		{"a2", "b2", "c2"},
		{"a3", "b3", "c3"}};
// sheet[row][column] 
System.out.println(sheet[0][1]); // b1
System.out.println(sheet[1][0]); // a2
System.out.println(Arrays.toString(sheet[0])); // [a1, b1, c1]

for (int row = 0; row < sheet.length; row ++) {
    for (int col = 0; col < sheet[row].length; col ++) {
        System.out.println(sheet[row][col]);
    }
}
// 2차원 배열 복사
String[][] copied = new String[sheet.length][sheet[0].length];
for (int row = 0; row < sheet.length; row ++) {
    copied[row] = Arrays.copyOf(sheet[row], sheet[row].length);
}
```

아래와 같이 특정 배열을 제외할 수도 있다. 만약 배열에서 소모시켜야 하는 알고리즘 문제라면 이 방법은 배열을 매번 재계산되므로 매우 느리니 고려하지 말자. (Memoization 활용 추천)

```java
int arr[][] = new int[][]{{80, 20}, {50, 40}, {30, 10}};

int[][] result = Arrays.stream(arr)
        .filter(item -> !Arrays.equals(item, new int[]{50, 40}))
        .toArray(int[][]::new);
System.out.println("result = " + Arrays.deepToString(result));
// [[80, 20], [30, 10]]
```

### 문자열

문자열도 일종의 배열이다.

```java
String string = "abc";
char[] charArray = string.toCharArray();

String to = new String(charArray);

// String 을 char[] 말고 String[] 으로 바꾸면 String 의 풍부한 기능 사용 가능
String[] sp = s.split("");
```

### 배열 출력

디버깅을 위해 배열을 출력해봐야 할 때가 있다.

```java
System.out.println(Arrays.toString(arr));
System.out.println(Arrays.deepToString(arr)); // 2차원 배열
```

## 리스트

아니면 ArrayList 를 사용하여 List 의 기능들을 쓰자, Arrays.asList() 로 배열을 쉽게 List 로 바꿀 수 있지만, int(primitive)는 Integer(wrapper)로 상호 변환되지 않기 때문에 for-loop 또는 stream() 이 필요할 수 있다.

```java
// List<Integer> list = Arrays.asList(array); // not working
List<Integer> list = new ArrayList<>();
for (int i=0; i<array.length;i++) {
  list.add(array[i]);
}
// int[] intArray = list.toArray(); // not working
int[] intArray = list.stream().mapToInt(Integer::intValue).toArray();

Integer[] intArray = list.toArray(new Integer[0]); // 배열 타입 전달
Integer[] intArray = list.stream().toArray(Integer[]::new);
Integer[] intArray = Arrays.copyOf(list.toArray(), list.size(), Integer[].class);

// Iterator 를 array 로
StreamSupport.stream(arr.spliterator(), false).toArray(String[]::new);
```

## 큐와 스택, 맵

Queue 나 Stack 을 써서 풀어야하는 알고리즘들이 있는데, (Queue/Heap 나 Stack 을 구현하는 문제가 아니라면) 그냥 java.util 에서 제공하는 것을 활용해도 된다. 단순한 경우라면 List 나 Map 을 활용하는 것이 나을 때도 있다. 

- 참고: [https://real-012.tistory.com/152](https://real-012.tistory.com/152)

```java
import java.util.*;
...
Stack<Integer> stack = new Stack<>();
stack.push();
stack.pop();
stack.peek();
PriorityQueue<Integer> queue = new PriorityQueue<>(); // 우선순위 큐
queue.add();
queue.poll();
queue.peek();
```

HashMap 은 hash 로 활용하기 딱 좋다(key 가 unique 한 점을 활용하기도 좋다).

```java
// 사전 정의
Map<String, String> map = new HashMap<String, String>() {{
    put("a", "b");
    put("c", "d");
}};
String v1 = map.get("a");
String v2 = map.getOrDefault("e", "f");

// Map 을 이용한 key 카운트
Map<String, Integer> count = new HashMap<>();
count.compute(c, (key, value) -> (value == null) ? 1 : value + 1);
// or
count.merge(c, 1, Integer::sum);
```

# 알고리즘

## 정렬

정렬 자체를 구현해야 하는 경우도 있지만, Java 의 기본 라이브러리에 들어있는 sort() 를 추천한다. 

Arrays 는 O(nlogn) ~ O(n^2) 정도, Dual-Pivot QuickSort 을 쓴다고 하며, Collection 은 O(nlogn)정도, Tim sort(Insertion 과 Merge 의 조합)을 쓴다고 한다.

Array 와 List 의 정렬 방법이 다른 이유는 Array 는 지역 참조성(메모리에 뭉쳐있어 cache 에 적재됨)이 강하고, Collection 은 분산되어 있기 때문에 이를 고려한 차이라고 한다

- 참고: [https://sabarada.tistory.com/138](https://sabarada.tistory.com/138)

### Arrays 의 sort() 사용

```java
int[] numbers = new int[30];
Arrays.sort(numbers); // numbers 가 정렬됨
```

### ArrayList 의 sort() 사용

```java
List<Integer> list = new ArrayList<>(array);
list.sort(Comparator.naturalOrder());
```

### Map 으로 만든 것을 sort()

```java
Map<Integer, Integer> map = new HashMap<>();
map.put(1, 1);
...
List<Entry<Integer, Integer>> list = new ArrayList<>(map.entrySet());
list.sort(Entry.comparingByValue());

// TreeMap 을 사용하면 자동으로 key 가 정렬된다.
Map<Integer, Integer> map = new TreeMap<>();
for (int number : card) {
    map.merge(number, 1, Integer::sum);
}
```

## 검색

검색 속도를 개선하는 문제가 아니라면 그냥 contains() 를 사용해서 검색하자. List 는 O(n), Set 는 O(1) (순서가 없고 중복 미허용) 의 성능을 보여준다. 순서가 중요하지 않다면 Map (O(log n)) 도 활용하기 좋다.

- 참고 : [https://codechacha.com/ko/java-difference-between-list-and-set/](https://codechacha.com/ko/java-difference-between-list-and-set/)

경우에 따라 Regular Expression 으로 검색할 수도 있다.

```java
String regex = "(\\d{1,9})";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(targetString);
if (matcher.find()) {
    String paymentId = matcher.group(1);
}
```

문자열에서 특정 문자열을 찾아 지우기 같은 검색 알고리즘 문제라면 거기에 맞게 구현하는게 성능을 높일 수 있다.  replaceAll() 같은 걸로는 time complexity 가 발생하는 문제일 가능성이 높고, 이를 노린 케이스가 반드시 존재한다.

# 코딩 스타일

## 간단한 산수

이 부분도 if 문으로 간단하게 해결할 수 있지만 알고리즘 테스트에서 허용한다면 java.lang.Math 를 이용하면 쉽다. 어차피 이 정도는 기본적으로 알고있는 기능이므로 굳이 다시 구현해야 하는건 불합리 할 것 같다.

```java
import java.lang.Math;
...
Math.ceil()      // 올림
int ceil = (int) Math.ceil((double)A / (double)K);
Math.round()     // 반올림
Math.floor()     // 버림, int를 int로 나누면 자동으로 버려진다.
Math.abs()       // 절대값
Math.max(a, b)   // 최대값
Math.min(a, b)   // 최소값
Math.pow(x, 2)   // 제곱, double 반환. Java 에는 ^ 같은 제곱 연산자가 없음
Math.sqrt()      // 제곱근
```

## 이진법

Bit 단위 연산으로 빠르게 해결할 수 있는 문제도 있다. 또한 bit 를 flag 처럼 사용해서 극단적인 효율을 추구할 수 있다. (&, |, ^, ~, <<, >>)

XOR (같으면 false, 다르면 true) 을 이용하여 pair 를 제거 할 수 있다.

```java
// 홀수개인 숫자 찾기
int[] array = new int[] {2, 2, 3, 3, 4};
int odd = array[0] ^ array[1] ^ array[2] ^ array[3] ^ array[4];

int binary = Integer.parseInt(binaryString, 2); // 이진문자열을 숫자로 변환
String binary = Integer.toBinaryString(n);      // 숫자를 이진문자열로 변환
int count = Integer.bitCount(n);                // 숫자의 이진 1 개수 반환
```

참고로 ^ 는 제곱이 아니다. Java 에는 제곱 연산자가 없다.

또한 이진법을 ”10011” 같이 String 으로 표현하는 방식으로 알고리즘을 물어보는 경우도 있는데, bit 연산을 이해하고 있으면 “곱하기/나누기 2는 shift 1칸” 같은 알고리즘을 만들기 편리하다.

## 문자와 숫자

숫자를 문자열로 바꾸거나 문자열을 숫자로 바꾸는 경우도 있다.

```java
int num = 100;
String number = String.valueOf(num); // "100"

char[] numbers = number.toCharArray(); // ['1','0','0']
int n = numbers[0] - '0'; // char 를 int 로 변환

String number2 = new String(numbers);
int number2 = Integer.parseInt(number2);
```

## 변수명

코딩 테스트를 위한 수학 용어

알고리즘을 작성하다 보면 변수명 또는 함수명을 작성해야 하는데, 이때 기본적인 수학용어가 필요할 때가 있다.  

예를 들면 A값을 2로 나누어 반올림을 하는 함수가 있다고 하면 대충 짓지 말고 실제 수학용어를 쓰자.

```java
round(divideBy2(number)) 
```

인터넷에 검색하면 관련 글이 많다. "수학 용어 영어로"

- 참고 : [https://dcmru.tistory.com/152](https://dcmru.tistory.com/152)

## 배열 즉시 반환

배열을 return 값으로 받는 문제에서, 즉시 값을 반환해야 할 때 활용하면 좋다.

```java
return new int[]{0, 0};
```

## 배열 양쪽에서 만나는 반복문

배열의 앞 뒤에서 동시에 까면서 내려오면서 반복한다. 보통 palindrome 문제에 활용할 수 있다.

```java
int i = 0, j = arr.length - 1;
for (; j > i; j--) {
    if (...) { } // arr[i] 와 arr[j] 로 뭔가 하기
    i++;
}
```

## 테스트

테스트 케이스를 많이 만들어두자. 미리 문제를 읽으면서 만들어두는게 삽질을 줄일 수 있고, 나중에 코드를 변경해야 하는 사태를 줄여준다. 

- 문제의 제약사항에서 범위가 0 부터(0 ≤ N)이면 0을 반드시 테스트 하자.
- 대상의 개수가 1000,000,000 개 이상인 경우에는 time complexity 문제가 반드시 있다.
- 대상의 개수가 몇 만개 내외라면, 특히 숫자의 범위가 Integer.MIN_VALUE ~ Integer.MAX_VALUE 라면 덧셈과 뺄셈에서 overflow 를 발생시키는 문제가 반드시 있다. long 타입을 쓰거나 덧셈/뺄셈의 위치를 조절하여 해결하자.
- 대상에 양수만 존재하는게 아니라 음수가 포함된다면 최소값/최대값 계산에 대해 한 번 더 고민해보자. 음수를 빼게되는 상황이 발생할 수 있다.
- 값을 1234567 으로 나누는 문제가 있는데, 이는 소수로 나누는 것이므로 최대값을 줄이려는 목적으로 이해할 수 있다. (A + B) % C == ((A % C) + (B % C)) % C

위와 같은 경우들을 테스트 케이스로 미리 만들어 두어야 한다. 다만 timeout 은 입력 가능한 테스트 케이스로는 발생시키기 쉽지 않으니 주의.

참고로 ,

- Integer 의 MAX_VALUE 는 2147483647 이고, MIN_VALUE 는 -2147483648 이다. 
  → 즉, 제약조건에서 10자리 이상의 숫자는 overflow 가 발생할 수 있으니 Long 으로 할 것
- Long 의 MAX_VALUE 는 9223372036854775807, MIN_VALUE 는 -9223372036854775808 이다. 
  → 제약조건에서 18자리 숫자는 무리없이 수용, 그 이상은 문자열로 해결하는 문제일 것으로 추정

# 마무리

개인적으로 알고리즘 코딩 테스트는 개발업계의 적폐라고 생각하는 입장이다. 이 글을 작성하면서 지원한 회사의 코테들을 다 떨어졌기 때문에 그렇게 생각하는건 아니다… 🥲

- 코테는 연습한 사람이 잘 푼다. 물론 머리가 엄청 좋아서 그냥 잘 푸는 사람도 있겠지만, 보통은 연습을 하지 않으면 버벅이다 실수하게 된다. 약간 토익과 결이 같다고 본다.
- for 문과 if 문이 떡칠된 코드가 얼마가 문제가 많은데, for 와 if 를 잘 사용하는 방법만 테스트 한다는게 넌센스라고 보기 때문이다.
- 코딩 스타일을 코테로 판단한다는 것도 무리수다. 회사마다 부서마다 스타일이 다르고, 그걸 표준화 하기 위해 컨벤션을 정의하는데, 지원자가 출제자의 컨벤션을 어떻게 알 수 있겠는가?
- 현업에서는 자료 정리와 보고서 작성도 일이고, 옆 부서와 회의하는 것도 일이고, 좋은 도구와 라이브러리를 사용하는 것도 일이다. 하물며 출퇴근도 일이고, 회식 장소 섭외하는 것도 일이다.
- 알고리즘은 이런 업무와 관계성이 거의 없다. 즉, 알고리즘을 잘 푸는데 일도 잘할 순 있겠지만, 알고리즘 잘 푼다고 일을 잘하는건 아니라는 것이다.

물론 코딩 능력이 기본은 되어야 한다는 전제가 있고, 지원자가 너무 많으니 걸러내려는 상황도 이해하지만, 알고리즘은 개발 업무에서 아주 적은 부분일 뿐이므로 기업들이 다른 방법을 찾았으면 좋겠다.

알고리즘 공부하는 것도 일이고 비용이다. 공부 해야할 다른 것도 너무 많은데…
