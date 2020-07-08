---
layout: post
title:  "Java에서 환경 변수 가져오는 3가지 방법"
date:   2020-07-08 23:00:00 +0900
categories: Java
comments: true
---

# Java에서 환경 변수 가져오는 3가지 방법
Spring에 쩔어 살다 보면 Spring에서 자연스럽게 뚝딱 처리하던 것들이 사실 꽤 복잡한 과정이 필요하다는 걸 잊게 된다.  
어느 날 문득 plain 하게 Java 프로그램을 만들던 중 JVM arguments(또는 options)나 application.properties를 가져오는 방법을 모른다는 걸 깨달았다.  
  
## 환경 변수
* System Environment
  + Windows의 C:\\> SET
  + Linux의 $ env
  ~~~ java
  // Get System Environment (no default value)
  String sysEnv = System.getenv(name);
  ~~~

## JVM arguments(-D로 넘어오는 값)
* System properties
  ~~~ java
  // Get System Property(-D..)
  String sysProp = System.getProperty(key, def);
  ~~~

## application.properties
* Properties (System.Properties와 같은 class)
  ~~~ java
    Properties properties = new Properties();
    ClassLoader loader = Thread.currentThread().getContextClassLoader();
    try (InputStream resourceStream = loader.getResourceAsStream("application.properties")) {
      properties.load(resourceStream);
    } catch (IOException e) {
      logger.error("Error: ", e);
    }

    // Get application.properties
    String appProp = properties.getProperty(key, def);
  ~~~