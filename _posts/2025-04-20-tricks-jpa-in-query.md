---
layout: post
title:  "IN Query 를 QueryDSL 로 구현할 때 주의사항"
date:   2025-04-24 00:00:00 +0900
categories: SQL, JPA, QueryDSL
comments: true
---

PK 등을 이용해 DB 에서 데이터를 대량으로 조회해야하는 경우가 있다.
예를 들면 이런저런 조건을 통해 상품 ID 1000 개를 알아냈고, 이제 이 상품 1000 가지의 목록을 가져와야 하는 경우이다. 가장 단순한 방법은 PK(상품 ID)를 IN query 를 이용해 조회하는 것인데, 이 ID 수량이 꽤 많을 때 우리는 어떤 문제를 만나게 될까?

# 문제 정의

DB 에서 SQL IN 절로 PK n 개를 조회한다면 어떻게 될까? 대략 아래와 같이 QueryDSL 이 작성될 것이다.

```kotlin
fun findById(ids: List<String): List<Product> =
  from(product)
  .where(id.`in`(ids))
  .fetch()
```

이는 아래와 같은 쿼리로 변환된다.

```sql
SELECT ID, NAME, CATEGORY FROM PRODUCT WHERE ID IN (...);
```

만약 저 IN 절이 100 개쯤 되면 어떨까? DB 종류에 따라 다르겠지만 일반적인 MySQL 에서의 경험으로는 아무 문제 없다. 속도도 충분히 빠르다.

그럼 1000 개 쯤이면? 10000 개, 100000 개 쯤 된다면? 경험상 수 천개가 넘어가면 쿼리 속도가 엄청 느려졌다.

참고로 이런 갯수별 성능을 정확히 지표로 표현할 수 없는 것은 동작하는 DB 의 종류, 시스템 자원 크기, 테이블 크기, 인덱스 등이 모두 변수가 되기 때문이다.

아무리 인덱싱이 되어 있는 PK 더라도 IN 절을 무분별하게 활용할 수 없다. 적당히 나누어 조회하도록 chunk 를 걸어주고 취합 하는게 좋다.

```kotlin
fun findById(ids: List<String>): List<Product> =
  ids
    .chunked(128) // <- 128 은 어디서 나온 걸까?
    .flatMap {
      from(product)
      .where(id.`in`(it))
      .fetch()
    }
```

그렇다면 우리는 이 “적당히 나누어” 줄 값을 어떻게 구해야 할까?

# 참고해야 할 것들

## 관련 설정

### 최대 쿼리 사이즈

`max_allowed_packet` 은 최대 쿼리 사이즈(보통 400MB)이다. IN 절에 들어갈 데이터가 짧은 숫자가 아니라면 고려해봐야 할 것이다.

```sql
SHOW VARIABLES LIKE 'max_allowed_packet';
```

### 인덱스 활용 조건

`eq_range_index_dive_limit` 는 range 시 index 를 탈지, 통계를 이용할지 판단하는 기준이다. range 가 될 경우 통계정보를 이용해 조회하게되어 속도가 예상과 달라질 수 있다.

보통 200개 (= 199개 까지만 index 를 태운다는 뜻)

통계를 판단하는게 무조건 나쁜건 아닌거 같긴한데, 예상 밖에 시간이 더 걸릴 수 있기 때문에 통계보다는 직접 판단하는걸 권장하는듯 하다. 그러나 수가 너무 늘어나면 성능이 떨어지기 때문에 성능 테스트를 하여 결정하라고 함

개인적인(= 부정확한🤑) 테스트 시 199개 이하는 항상 빠른데, 200개, 201개 까지는 빨랐다 느렸다하고, 202개 부터는 느렸다. (200, 201개는 캐싱의 영향일 수도 있음, 마지막 값을 바꾸는 것만으로도 느려짐)

```sql
SHOW VARIABLES LIKE 'eq_range_index_dive_limit';
```

### Prepare Statement 검토

`in_clause_parameter_padding` 은 JPA 설정으로 IN 쿼리의 preparedStatement 종류(크기)를 제한하는 방법이다.

간단히 말하자면 비슷한 쿼리는 미리 캐싱을 해두는 것인데, IN 절 이후 붙는 파라메터 개수에 따라 쿼리를 캐싱하지 못하는 것 보다 1, 2, 4, 8, 16 … 으로 10 가지만 캐싱하는게 낫다는 전략이다.

```sql
SELECT * FROM product WHERE id IN (?, ?, ?, ?, .... )
-- IN 절에서 ? 의 갯수별로 구문(statement)을 준비해둔다.
```

JPA 의 설정에 padding 옵션은 기본으로 꺼져있기 때문에 활성화 해줘야 한다.

```yaml
spring.jpa.properties.hibernate.query.in_clause_parameter_padding=true
```

단점, 해당 설정을 모르는 사람이 debug 시 IN 절이 예상보다 길어져 깜짝 놀랄 수 있다.

IN 3개 호출 하면 처럼 4개로 IN 생성(마지막 item 반복)됨

```sql
SELECT * FROM product WHERE id IN (1, 2, 3, 3);
```

### OR 에 의한 쿼리 사이즈 확인

IN 절은 경우에 따라 OR 의 연속으로 간주될 수 있다. 이 경우 쿼리를 메모리에 올릴 때 사이즈 제한이 걸릴 수 있다.

`range_optimizer_max_mem_size` 는 IN 절을 OR 로 바꿀 때 허용할 메모리 사이즈(보통 8MB)이다.

```sql
SHOW VARIABLES LIKE 'range_optimizer_max_mem_size';
```

# 결론

### IN 절 개수 제한

- MySQL에서 IN 절 개수 제한은 없지만, max_allowed_packet 크기에 영향을 받음 → 일반적으로 수 천 개도 가능하므로 크게 영향이 없음

### eq_range_index_dive_limit 적정 값

- DBA 의견 : 기본적으로 IN 절의 개수는 200~1000 사이가 적정(위 내용들에 대한 종합적 통찰)하며 500~1000 정도로 설정가능
- 기본값 200
- 그러나 기본값을 변경하는 것 보다는 IN 절을 분할하는 것이 좋음
    - 너무 많은 인자로 인해 쿼리 로그가 잘리는 문제가 발생 → IN 절을 작게 나누어 운영 가독성을 높이는 것이 권장됨
- 기본값 유지 권장
    - AWS 엔지니어들과 논의 결과, 과거에는 다양한 파라미터 조정이 일반적이었으나, 현재는 기본값 유지 + 애플리케이션 레벨의 최적화가 우선
    - 클라우드 환경에서는 무분별한 파라미터 변경이 추가 운영 비용 증가로 이어질 수 있음 → 조직의 SRE 에서 기본값 유지를 권고

### in_clause_parameter_padding 활성화

- 앞으로 in 절을 이용한 조회를 많이 한다면 prepared statement 를 효율화 시킬 필요가 있다. 그러나 다양한 이유(옵티마이저의 사이드이펙트)로 비추천
- MySQL 8.0 부터는 성능 향상 효과도 없어 padding 불필요

## 정리

200개 이하, 8MB 이하, 2의 거듭제곱, 적절한 로그 길이, 성능영향 없는 최대값 → `128` ?

> 주의! 각자의 DB 에서 테스트 해보고, 데이터의 형태 및 요구사항을 판단하여 결정해야 함!!
>

# 참고

- `eq_range_index_dive_limit` : https://pjh3749.tistory.com/288
- `in_clause_parameter_padding` : https://www.manty.co.kr/bbs/detail/develop?id=98
- `range_optimizer_max_mem_size` : https://jobc.tistory.com/216
- https://dev.mysql.com/doc/refman/8.4/en/range-optimization.html
- https://jojoldu.tistory.com/565