---
layout: post
title:  "Spring Boot에서 json을 위한 XSS Filter 적용하기"
date:   2019-03-28 15:00:00 +0900
categories: Java Spring Boot XSS
comments: true
---

# Spring Boot에서 json을 위한 XSS Filter 적용하기
옛날 Spring을 처음 접하면서 엄청 놀랐었다. "와.. 간단한 설정이면 웹앱 개발 뚝딱이네.."  
최근 Spring Boot를 이용하여 개발을 하면서 많은 것을 느꼈다. "이제 개발할 건 비즈니스 로직 뿐이네.."  
그러나 최근 XSS 를 방어하기 위한 설정을 하다가 또 한 번 놀랐다.  
"Spring에 XSS 방어할 설정이 없다고?"

## Filter와 Interceptor
  * XSS를 방어하기 위해 제일 처음 떠오른 것은 역시 Filter와 Interceptor
    + 간단할 줄 알았는데 다음과 같은 문제에 직면
      - Filter든 Interceptor든 request의 InputStream을 한 번 읽으면 끝 ([참고](https://stackoverflow.com/questions/21193380/get-requestbody-and-responsebody-at-handlerinterceptor))
      - InputStream을 다시 전달하기엔 너무 많은 공수가 필요... 이렇게 까지 해야하나? ([Spring Interceptor(혹은 Servlet Filter)에서 POST 방식으로 전달된 JSON 데이터 처리하기](https://meetup.toast.com/posts/44))
  * 오픈 소스가 있지 않을까?
    + Lucy XSS Filter ([링크](http://naver.github.io/lucy-xss-filter/kr/))
      - Naver에서 blog나 cafe 서비스를 하면서 겪었을 고충들을 느낄 수 있음
      - 잘 만들었지만, 한글 임에도 불구하고 설명이 좀 복잡
      - Request param 또는 form-data로 전달 된 경우만 filtering
      - RequestBody를 통해 raw data로 보내는 것은 잡지 못함 (2019년 3월 기준)
  * 결국 모두가 비슷한 생각을 함
    + http://homoefficio.github.io/2016/11/21/Spring%EC%97%90%EC%84%9C-JSON%EC%97%90-XSS-%EB%B0%A9%EC%A7%80-%EC%B2%98%EB%A6%AC-%ED%95%98%EA%B8%B0/
      - 위 블로그의 작성자님의 방법(Message Conveter)이 가장 현실적으로 다가옴

## Message Conveter
  * XSS관련 검색 시 StackOverflow에서도 제안함
    + [Spring Boot escape characters at Request Body for XSS protection](https://stackoverflow.com/a/55292262/8350542)
    + [@InitBinder with @RequestBody escaping XSS in Spring 3.2.4](https://stackoverflow.com/a/25405385/8350542)
    + 위 HomoEfficio님 블로그에서도 해당 StackOverflow를 참조
      - 좀 더 확장하여 Mapper의 deserializer를 활용한 방법 [@Circlee7](https://medium.com/@circlee7/spring-boot-jackson-json-xss-%EC%B2%98%EB%A6%AC-fdc85a18e9f2)님의 글
  * Message Converter 사용법
    + Baeldung [[Http Message Converters with the Spring Framework](https://www.baeldung.com/spring-httpmessageconverter-rest)] 참조
    + Spring에서는 10여가지의 default converter를 제공
    + Request의 Content-Type에 대응하여 converter를 선택
      - application/json의 경우 대게 MappingJackson2HttpMessageConverter에서 처리
  * HTTP Escape 라이브러리 [StringEscapeUtils.escapeHtml4()](https://howtodoinjava.com/java/string/escape-html-encode-string/)/[Doc](https://commons.apache.org/proper/commons-text/javadocs/api-release/org/apache/commons/text/StringEscapeUtils.html) 를 활용하여 심플하게 구현할 예정

## Message Conveter를 적용
  * 위에서 제안된 Message Conveter를 WebMvcConfigurer에 적용
    + WebMvcConfigurerAdapter는 Spring Boot 2.0 이후 deprecated 됨
      - WebMvcConfigurer에 configureMessageConverters를 override 해서 사용
    + Adapter를 사용하지 않기 때문인지 configureMessageConverters에 기본적으로 default converters가 전달 됨
      - 참고: configureMessageConverters는 default converters를 사용하지 않고, 사용자가 입력한 것만 쓰게만드는 method
      - 이지만, WebMvcConfigurer에서 받을 땐, 이미 default converters가 생성되어 있음
      - 즉, 우리가 override할 MappingJackson2HttpMessageConverter가 이미 포함되어 있음
  * (주의!) MappingJackson2HttpMessageConverter 이 여러개 일 경우 내가 추가한 것이 선택되지 않음
    + 그러므로 application/json으로 선택되는 converter를 덮어 써야 함
    + 기존것을 수정할 순 없고, 찾아서 지우고, 새로운 것을 add해야 함
  * 수정된 코드
  ~~~ java
  @Override
  public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    // Replace MessageConverter from default WebMvcConfigurer
    Iterator<HttpMessageConverter<?>> converterIterator = converters.iterator();
    while(converterIterator.hasNext()) {
      // Do NOT add new one, MUST replace
      HttpMessageConverter converter = converterIterator.next();
      if (converter.getSupportedMediaTypes().contains(MediaType.APPLICATION_JSON))
        converterIterator.remove();
    }
    converters.add(createCustomHtmlEscapingConveter());
  }
  ~~~
  * ObjectMapper 추가 설정
    + Default Converter 는 기본적으로 존재하지 않는 property는 무시하고 mapping을 해줌
    + 그런데 그냥 new ObjectMapper()로 생성해버리면, 존재하지 않는 property 전달 시 에러 발생
      - https://stackoverflow.com/a/12730655/8350542
      - FAIL_ON_UNKNOWN_PROPERTIES 를 false로 설정

## 결과
  * RequestBody annotation엔 적용이 안됨(확인 필요)
  * ResponseBody annotation엔 적용 성공!
  ~~~ json
  // Request
  { 
	  "data": "hello <script>alert('xss attack');</script>" 
  }
  
  // Response
  {
    "result": "hello &lt;script&gt;alert('xss attack');&lt;/script&gt;"
  }
  ~~~

## 기타 참고 자료
  * https://stackoverflow.com/questions/22461663/convert-inputstream-to-jsonobject
  * https://stackoverflow.com/questions/6511880/how-to-parse-a-json-input-stream/13267720
  * https://www.baeldung.com/spring-boot-add-filter
  * https://supawer0728.github.io/2018/04/04/spring-filter-interceptor/
