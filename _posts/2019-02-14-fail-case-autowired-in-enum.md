---
layout: post
title:  "금주의 실패사례 - Java Spring Boot에서 Enum안에 @Autowired넣기"
date:   2019-02-14 16:00:00 +0900
categories: Spring Boot Java Enum IoC Failure
comments: true
---

# Enum type을 생성할 때, Spring Boot의 service이용 하기
Enum은 최초로 호출 될 때 생성이 된다.  
이때 생성될 때 현재 DB 들어있는 정보로 동적으로 생성하고자 하는데,  
DB에서 정보를 가져오는 service를 @Autowired 시켜서 사용하고 싶다.  
어떻게?

## 참고
  * Bean 생성 타이밍에 대해서 잘 정리한 글 https://jeong-pro.tistory.com/167
  * 실질적인 해결 책 https://stackoverflow.com/questions/16318454/inject-bean-into-enum
  * 성공은 못시켰지만, 검토해볼만한 해결 책 https://stackoverflow.com/questions/3818026/force-initialization-of-an-enumerated-type-in-java
    + static {} 또는 {} 사용
  * @Autowired가 동작하지 않을 때 참고하기 좋은 방법들 https://technology.amis.nl/2018/02/22/java-how-to-fix-spring-autowired-annotation-not-working-issues/

## 요약
Enum 안의 static class가 생성될 때 @PostConstruct로 생성되길 기다렸다가 값을 삽입하므로 Injection 문제가 해결 되는 듯

## Sample
~~~ java
import javax.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

public enum Fruit {
  APPLE("red"),
  BANANA("yellow");

  private String color;
  private String seller;

  Fruit(String color) {
    this.color = color;
  }

  @Component
  public static class FruitEnumInjector {
    @Autowired
    private SellerService sellerService;
    @PostConstruct
    public void postConstruct() {
      try {
        APPLE.seller = sellerService.getSeller("apple");
        BANANA.seller = sellerService.getSeller("banana");
      }
      catch(Exception e) {
        throw e;
      }
    }
  }
  public String getColor() {
    return color;
  }
  public String getSeller() {
    return seller;
  }
}
~~~

## But
https://stackoverflow.com/questions/45192373/how-to-assign-a-value-from-application-properties-to-a-static-variable