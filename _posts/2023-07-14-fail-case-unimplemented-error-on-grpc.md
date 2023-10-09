---
layout: post
title:  "오늘의 실패사례 - gRPC 에러 해결"
date:   2023-07-14 21:00:00 +0900
categories: Java Protobuf gRPC
comments: true
---

# 갑자기 gRPC 에러를 만났다

Protobuf 를 이용해 다른 서비스와 잘 연동해서 개발하고 있었는데, 갑자기 Method not found 에러를 만났다.

> io.grpc.StatusRuntimeException: UNIMPLEMENTED: Method not found: ….
> 

아래와 같은 에러는 마치 내가 어떤 기능을 구현하지 않았거나, 구현한 것을 찾지 못했다는 것 처럼 보인다. 라이브러리 버전 불일치? bean 주입 실패? 뭐지?

```
io.grpc.StatusRuntimeException: UNIMPLEMENTED: Method not found: mys.service.user.GuestService/FindGuest
```

# 구현한 method 가 사라졌다고?

만약 REST 로 구현했다면 404 를 만났을 것이라 바로 눈치챌 수 있겠지만, gRPC 는 익숙지 않아서 한참을 헤맸다.

[https://stackoverflow.com/q/75949594/8350542](https://stackoverflow.com/q/75949594/8350542) 여기의 댓글이 힌트

> Are you sure that in the packaged case you are pointing your client at the right server? I once made the mistake of accidentally calling a server that didn't have the desired service and was equally confused by getting this response.
> 

결론을 말하자면 stub 을 통해 호출할 대상 서비스의 구현체를 찾지 못했다는 뜻이다. 즉, 대상 서버에 해당 method 가 없다는 뜻.

## 결론

이런 저런 이유로 상대 서비스의 package 명이 변경되었었는데, 기존 서버에는 구 버전의 package 로 배포되어 있고, 이를 신규 package 의 stub 으로 접근하려니 구현체를 찾지 못했다는 뜻이다.

상대 서비스를 재배포하여 해결!

Protobuf 는 내 서비스를 사용하는 클라이언트에게 stub 을 제공해준다. 그렇기 때문에 내 서비스를 변경하려면  stub 도 같이 업데이트 되어야 하는데, 클라이언트가 다른 조직이거나 불특정 다수라면 stub 변경 관리가 어려워진다. 

그래서 REST 보다는 조금 더 빡빡하게 compatibility 를 중시하는 경향이 있는 것 같고, 대신 더 쉽고 빠르게 개발할 수 있는 것 같다.

참고로 만약 내가 서버였다면 아직 구현하지 않았다는(구현체를 못찾았다는) 뜻일 수도?

[https://stackoverflow.com/a/56235482/8350542](https://stackoverflow.com/a/56235482/8350542)