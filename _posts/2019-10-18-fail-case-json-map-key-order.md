---
layout: post
title:  "금주의 실패사례 - JSON의 map에서 key의 순서"
date:   2019-10-18 20:00:00 +0900
categories: JSON Java
comments: true
---
# JSON을 응답으로 받을 때, 값을 정렬해서 받고 싶다.
Front-end에서 REST API 서버로 부터 응답을 받을 때, 보통은 JSON 형식으로 받는다.  
원래 JSON에는 정렬(order)가 없다고는 하지만, 대체적으로 document로 쓰여있는 순서를 order로 그대로 유지한다.  
그런데, map을 back-end에서 미리 정렬해서 응답했더니 JavaScript가 객체로 읽어들이는 과정에서 자기 마음대로 다시 정렬하는 문제가 생겨버렸다.  

## Map을 key로 정렬하자
다들 여러가지 사연이 있듯이, 서버로 부터 데이터를 받을 때 key-value형식을 key로 정렬하여 받아야하는 상황이 생겼다. Front에서 정렬 할 수도 있겠지만, front의 부담을 줄이기 위해 서버에서 정렬하여 주기로 하였다.

* 증상
  + Browser에서 개발자모드로 debugging
    - 실제로 응답된 JSON document를 확인해 보면 map이 key 순으로 정렬이 되어 있음
  + 그러나 객체로 deserialize 하면 정렬이 깨짐
    - 정확히는 무조건 오름차순이 되는 문제
    - key 값이 [0, 1.3, 1] 일때, 이를 내림차순으로 정렬해도 오름차순 [0, 1, 1.3] 으로 다시 정렬됨

* 왜 안될까?
  + List는 정렬이 잘 유지됨
    - Map만 문제인가? 어떤 경우는 잘 유지되기도 함 (환장)
  + 나의 테스트 값을 다시 체크 해보자
    - 테스트할 때 썼던 key 값이 [0, 1.3, 1]
    - 이를 어떤 방향으로 정렬하더라도 오름차순 [0, 1, 1.3] 으로 재정렬됨
  + 테스트 숫자를 바꿔보자
    - [0, 1.3, 1, 0.2] 을 정렬하니 [0, 1, 0.2, 1.3] 으로 됨 (잉?!)
    - 즉, 오름차순으로 재정렬 하는 것이 아님
    - 0, 1이 항상 맨위로 갔기 때문에 재정렬이 된다고 착각했던 것일 뿐

* 확인 결과
  + JSON map에서 key가 숫자(정수)인 경우는 무조건 맨위에 오름차순으로 자동정렬
  + 소수점은 문자형으로 판정해서 정수 다음 차례로 내려감
    - 이 부분은 좀 더 확인해 볼 것
  + 그 후 **문자형 key는 전달 받은 map의 키 순서를 유지**
    - 즉, [0, 3, 2] 였으면 무조건 [0, 2, 3] 으로 정렬 됨
    - 그러나, [0.0, 0.3, 0.2] 였으면 [0.0, 0.3, 0.2] 이 유지
    - [c, b, a, 0] 이면 [0, c, b, a] 가 됨

## 결론
JSON의 map을 어떤 방식(내림차순 등)으로 정렬하더라도, deserialize했을 때 key가 숫자인 경우, 그 숫자가 전체 key 중 일부일지라도 숫자 key는 무조건 오름차순으로 자동정렬된다.  

## 참고
* Java에서 map을 정렬하는 방법
  + Map의 형식은 버전을 key로, 데이터를 value로 하고 key기준으로 정렬
    - Key순서로 정렬하는 방법: [LinkedHashMap 또는 TreeMap](https://stackoverflow.com/questions/683518/java-class-that-implements-map-and-keeps-insertion-order), [Gson과 LinkedHashMap의 조합으로 정렬을 유지](https://stackoverflow.com/questions/29491281/building-ordered-json-string-from-linkedhashmap)
    - 버전은 #.# 과 같은 형식의 key이기 때문에, 일반 String 비교가 아니라 별도의 comparator를 별도로 정의 해야 함
    - [comparator만드는 방법](https://stackoverflow.com/a/17680341), comparator를 treemap의 생성자에 넣으면 그걸로 정렬해줌
      > private Map<String, String> versionMap = new TreeMap<>(new VersionComparator());
    - [Collections.reverseOrder()를 하면 treemap에 역순정렬](https://www.geeksforgeeks.org/how-to-create-a-treemap-in-reverse-order-in-java/) - Collections.reverseOrder() 역시 comparator를 반환
  + 이를 JSON형식의 MIME(application/json)으로 browser에 반환하면
    - **정렬이 안됨**... 정렬이 깨짐. 왜?
    - 일단 JSON spec에 의하면, [json에서 order를 지키는 방법은 없다](https://stackoverflow.com/questions/4515676/keep-the-order-of-the-json-keys-during-json-conversion-to-csv)고 함.    
    - 그런데 경험상 지켜짐 (순서를 보장 못한다고 이해 하면 될듯)
  + 참고: Map 구현체 별 특징
    - TreeMap: 자동(정의된 comparator에 의거)으로 key정렬
    - HashMap: 그냥 랜덤
    - LinkedHashMap: 값이 입력된 순서를 유지
