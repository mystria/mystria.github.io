---
layout: post
title:  "Guava로 Cache 적용하기"
date:   2021-01-30 24:00:00 +0900
categories: Java Guava
comments: true
---

# Guava로 Cache 적용하기
Java application 개발 중 간혹 cache를 적용해야 할 때가 있다. 어떻게 할까... 순간 많은 로직이 뇌리를 스치고 지나간다.  
Static 한 map을 만들고, hit 하면 반환 - 못하면 DB 조회, 시간이 오래되면 갱신 등등... 비즈니스 로직을 구현하기도 바쁜데 제대로 동작할지 모르는 cache를 직접 구현하려니 막막하다.  
이럴 때 우리들의 든든한 형, Google 형님이 계신다. Google 형님은 우리들의 하찮은 시간을 보전하라고 [Guava](https://github.com/google/guava)라는 Java 용 라이브러리를 만들어 주셨다.  
Guava, 아마 Google + Java가 아닐까?  

## Guava의 Caches
[Guava의 Caches](https://github.com/google/guava/wiki/CachesExplained)는 대략 CacheBuilder와 CacheLoader로 구분됨  
* CacheBuilder: Cache를 만들어 주는 Builder class, cache의 크기, life-cycle 등(eviction)과 세부 커스텀 동작을 정의
* CacheLoader: Cache를 거쳐 제공할 데이터 조회(population)를 정의

## 구현
* 예제는 Guava GitHub에 자세하게 설명되어 있음
  + [https://github.com/google/guava/wiki/CachesExplained](https://github.com/google/guava/wiki/CachesExplained)  
* 정의 예제
  ~~~ java
  LoadingCache<String, User> users = CacheBuilder.newBuilder()
    .maximumSize(10)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build(
      new CacheLoader<String, User>() {
        @Override
        public Map<String, User> loadAll(Iterable<? extends String> keys) throws Exception {
          ...
          return userDaoService.getMap(keys);
        }
        @Override
        public User load(String key) throws Exception {
          ...
          return userDaoService.get(key);
        }
      });
  ~~~
  + Inline으로 작성되어 있지만, CacheLoader는 별도의 class로 상속받아 구현해두면 향후 유지보수가 쉬울 것
  + CacheBuilder는 LoadingCache 객체를 생성하는데, LoadingCache는 get(), getAll(), refresh()등 필요한 기능을 다 갖추고 있음
  + Cache 값 반환은 Guava의 ImmutableMap(불변, 변경할 수 없는 map) 타입으로 함
  + CacheBuilder에서 직접 cache값을 입력하거나, cache에 없을 때 수행할 메소드 설정(callable), refresh 설정, 삭제된 데이터 반영(removal listner) 등 다양한 튜닝도 가능
* 사용
  ~~~ java
  @Autowired
  private LoadingCache<String, User> userCache;
  ...
  Set<String> userIdSet = new HashSet<>(Arrays.asList("111", "222"));
  ImmutableMap<String, User> userMap = userCache.getAll(userIdSet);
  ~~~
  + getAll() 수행 시, 없는건 loadAll로 찾아서 함께 반환

## 결론
어쨌든 좋은 도구들은 미리 만들어져 있으니 우리는 비즈니스 로직에 집중하면 된다.

## 기타
* 참고: [Guava를 써야하는 5가지 이유](https://blog.outsider.ne.kr/710)