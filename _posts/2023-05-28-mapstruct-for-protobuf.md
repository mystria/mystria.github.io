---
layout: post
title:  "MapStruct 를 이용해 Protobuf 로 생성된 Java Class 매핑하기"
date:   2023-05-28 01:00:00 +0900
categories: Java MapStruct Protobuf
comments: true
---

# 개요
평소 프로젝트 진행 시 MapStruct 를 이용해 DTO 변환을 편하게 하고 있었다. 그런데 gRPC API 를 이용하는 서비스를 개발하는 과정에서 우리 DTO 와 Protobuf 로 자동생성된 Java Class 사이를 매핑하는게 간단하진 않았다.

## MapStruct Library 란?

MapStruct 라이브러리[[공식](https://mapstruct.org/)]는 Java Bean 매핑 라이브러리로 타입과 변수명을 인식해 DTO 와 Entity 를 변환하는 매퍼(Mapper) 코드를 자동생성해 준다. 반복적인 코드 작성 없이 간단하게 매핑해주기 때문에 번거로운 작업과 코드량을 줄일 수 있다.

여기[[mapstruct-examples](https://github.com/mapstruct/mapstruct-examples/tree/main)]에서 다양한 예제를 확인할 수 있으며 관련 글도 많이 찾아 볼 수 있다.

## Protobuf 란?

Protobuf(Protocol Buffers)[[공식](https://protobuf.dev/)]는 언어와 플랫폼 중립적인 데이터 직렬화 메커니즘이다. 보통 gRPC 에서 인터페이스 정의를 위해 IDL(Interface Definition Language) 로 사용한다.

Java 개발 시 Protobuf 를 기반으로 Java Class 를 생성할 수 있는데, 이를 통해 쉽게 gRPC 서비스를 구현 또는 호출할 수 있다.

# DTO 와 Protobuf 생성 Class 의 매핑

MapStruct 를 사용하는 방법은 인터넷에 많이 설명되어 있기 때문에 생략한다.

- 참조 : [https://www.baeldung.com/mapstruct](https://www.baeldung.com/mapstruct)

원할한 설명을 위해 앞으로 “Protobuf 생성 Class” 를 그냥 Protobuf Class 로 부르겠다. 그리고 아래 예제를 통해 설명하고자 한다.

### Protobuf 예제

```protobuf
syntax = "proto3";
option java_multiple_files = true;
...

message Restaurant {
  string name = 1;
  int32 id = 2;
  repeated Seat seats = 4;
  Owner owner = 5;
}

message Owner {
  string name = 1;
}

message Seat {
  string seat_id = 1;
  Type type = 2;
}

enum Type {
  UNKNOWN = 0;
  SINGLE = 1;
  COUPLE = 2;
  FAMILY = 3;
}
```

### Java DTO 예제

```java
@Getter
@Builder
public class RestaurantDto {
    private String name;
    private Integer id;
    private List<SeatDto> seats;
    private OwnerDto owner;

    @Getter
    @Builder
    public static class SeatDto {
        private String seatId;
        private TableType tableType;
    }

    @Getter
    @Builder
    public static class OwnerDto {
        private String name;
    }

    public enum TableType {
        COUPLE,
        FAMILY
    }
}
```

## DTO 를 Protobuf Class 로 매핑하기

DTO 를 Protobuf Class 로 변환할 때 (개인적으로 생각하는)가장 큰 문제점은 List → repeated 이다. Protobuf 에서 repeated 타입은 List 와는 살짝 다른 느낌인데 `addSeat()` 처럼 add 형식으로 목록을 추가한다. 그래서 MapStruct 적용 시 단순히 List to List 라고 생각하면 안된다.

아래와 같이 매퍼를 정의할 때 collection 매핑 전략으로 ADDER_PREFERRED 를 반드시 정의해줘야 한다.

```java
@Mapper(collectionMappingStrategy = CollectionMappingStrategy.ADDER_PREFERRED,
    nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS)
public interface RestaurantMapper {
    ...
}
```

Null 값 확인 전략은 Protobuf Class 가 Null 을 허용하지 않기 때문에 추가한다. 이 두 가지 전략 추가는 앞서 말한 [MapStruct 예제](https://github.com/mapstruct/mapstruct-examples/blob/main/mapstruct-protobuf3/usage/src/main/java/org/mapstruct/example/mapper/UserMapper.java)에서 제안하는 내용이다.

## Protobuf Class 를 DTO 로 매핑하기

Protobuf Class 를 DTO 로 변환하는건 평소와 비슷하게 사용하면 된다. 단, 아래의 List, Enum 매핑을 참고할 것!

## List 매핑하기

앞서 설명한 DTO 와 Protobuf Class 간 매핑에서 List 처리에 또 하나의 문제가 있다. 

예제 코드에서 `repeated Seat seats = 4;` 로 정의된 collection 은 엄밀히 List 타입이 아니기 때문에 seats 와 seats 간의 매핑이 자동으로 되지 않는다. Protobuf Class 에는 seats 의 list 는 seatsList 로 따로 정의되어 있으므로 이 부분을 수동으로 매핑해줘야 한다.

```java
@Mapping(source = "seatsList", target = "seats")
RestaurantDto toDto(Restaurant proto);

@Mapping(source = "seats", target = "seatsList")
Restaurant toProto(RestaurantDto dto);
```

DTO 와 Protobuf Class 양방향으로 source / target 을 지정해줘야 한다.

## Enum 매핑하기

Enum 간의 매핑이 아주 완벽히 1:1 이면 문제가 없지만 한 쪽이 적거나 많으면 문제가 발생한다.

적은 종류의 enum 에서 많은 종류의 enum 으로의 매핑은 별 문제가 없다. 그러나 반대인 많은 종류에서 적은 종류로의 매핑은 매핑되지 않고 남는 것들에 대한 문제가 생긴다. 

Protobuf Class 에 정의된 enum 에는 숨겨진 타입이 하나 더 존재하는데 바로 `UNRECOGNIZED` 이다. 이는 -1번에 지정되어 있다. 이 부분은 보통 DTO 의 enum 에서는 없을 수 있기 때문에 매핑이 필요하다.

예제에서는 대충 남는 것들을 UNKNOWN(String 으로 지정) 이나 Null 로 매핑하였다.

```java
@ValueMapping(source = MappingConstants.ANY_REMAINING, target = "UNKNOWN")
Type map(TableType type);

@ValueMapping(source = MappingConstants.ANY_REMAINING, target = MappingConstants.NULL)
TableType map(Type type);
```

MappingConstants 에는 다양한 옵션이 있는데 source 에서 사용 불가능한 것들과 target 에서 사용 불가능한 것들이 있기 때문에 주석 또는 build 단계의 에러메시지를 주의해서 살펴보자.

```
The following constants from the source enum have no corresponding constant in the target enum and must be be mapped via adding additional mappings: UNRECOGNIZED.
```

## 전체 Mapper 코드

```java
...
import org.mapstruct.CollectionMappingStrategy;
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.MappingConstants;
import org.mapstruct.NullValueCheckStrategy;
import org.mapstruct.ValueMapping;
import org.mapstruct.factory.Mappers;

@Mapper(collectionMappingStrategy = CollectionMappingStrategy.ADDER_PREFERRED,
    nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS)
public interface RestaurantMapper {

    RestaurantMapper INSTANCE = Mappers.getMapper(RestaurantMapper.class);

    @Mapping(source = "seatsList", target = "seats")
    RestaurantDto toDto(Restaurant proto);

    @Mapping(source = "seats", target = "seatsList")
    Restaurant toProto(RestaurantDto dto);

    @ValueMapping(source = MappingConstants.ANY_REMAINING, target = "UNKNOWN")
    Type map(TableType type);

    @ValueMapping(source = MappingConstants.ANY_REMAINING, target = MappingConstants.NULL)
    TableType map(Type type);
}
```

# 그냥 MapStruct 팁

- `@Mapping(target = "fieldName", ignore = true)` 를 이용하면 매핑 대상 필드를 무시할 수 있다.
- `@Mapping(target = "fieldName", defaultValue = "defaultValue")` 를 이용하면 대상 필드의 기본값을 설정할 수 있다.
- `@Mapping(source = "dto", target = "fieldName", qualifiedByName = "mapDto")` 를 이용하면 매핑 로직(`qualifiedByName`)을 별도로 지정할 수 있다.
- Java 8 이상에서는 interface 에 default 로 method 구현이 가능하다. 이를 이용해 복잡한 매핑 로직은 매퍼 클래스에 직접 작성도 가능하다.
- `@Mapping(target = "isOpen", expression = "java(dto.getStatus().equals('open'))")` 처럼 간단한 로직은 inline 으로 구현 가능하다.
- `@Mapper(componentModel = "spring")` 를 이용하면 매퍼를 Spring 에서 주입할 수 있다.
- 매퍼는 annotation processing 단계에 생성되므로 `RestaurantMapper INSTANCE = Mappers.getMapper(RestaurantMapper.class);` 처럼 주입하지 않고 static 으로도 사용 가능하다.
- Lombok 사용 시 Lombok annotation processing 을 반드시 MapStruct 보다 먼저 실행해야 MapStruct 코드가 제대로 생성된다. Lombok 이 생성한 constructor 나 builder 를 이용해 매퍼를 만들기 때문이다. - [https://stackoverflow.com/a/65955495/8350542](https://stackoverflow.com/a/65955495/8350542)

# 결론

간혹 속성이 수십 개에 달하는 DTO 를 구현하는 경우도 있다. 어쩌면 IDE 의 코드 생성이나 Copilot 의 자동 완성을 이용하여 DTO 매핑을 할 수도 있을 것이다. 그러나 DTO 의 속성들은 변한다. MapStruct 를 이용하자. Build 단계에서 새로운 매퍼를 만들어줄 것이다.

MapStruct 팀은 개발자의 의견을 매우 빠르게 수렴하고 도와주고 있다. 실제로 StackOverflow 를 보면 MapStruct 개발팀이 직접 답변 및 설명을 해주는 것을 쉽게 찾아 볼 수 있다. 어느 개발자가 요청한 Protobuf 3 를 지원하는 과정도 git issue[[링크](https://github.com/mapstruct/mapstruct/issues/743)] 에서 볼 수 있었다. 충분한 샘플 코드도 제공한다.

매핑 노가다에 지친 개발자라면 꼭 사용해 보자!