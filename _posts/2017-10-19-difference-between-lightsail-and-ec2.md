---
layout: post
title:  "Lightsail과 EC2의 차이"
date:   2017-10-19 14:00:00 +0900
categories: AWS EC2 Lightsail StackOverflow
comments: true
---
# Lightsail과 EC2의 차이
영어 공부겸 StackOverflow의 글 번역. 발번역 주의
[한국 공식 블로그](https://aws.amazon.com/ko/blogs/korea/category/amazon-lightsail/)에서 Lightsail소개 해줌

## [What is difference between Lightsail and EC2?][reference]

테스트에 의하면&sup1; Lightsail은 사실 버스트 기능이 있는 [`t2`](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/t2-instances.html) 계열의 EC2 instance와 동일하다.

물론, EC2는 t2이외의 다양한 instance 종류가 있고, 전반적으로 "파워풀"하(또는 "현재 상태에 적합한" 걸 적용하기쉽)고, 요금이 좀 더 비싸다.
그러나 의미있는 비교를 해보자면, 512 MiB의 Lightsail instance는 동급(비슷한 가격)인 t2.nano와 스펙상 완전히 동등하다. 1 GiB는 t2.micro, 2 GiB는 t2.small과 동급 등...

Lightsail은 가볍고, 단순화 된 제품이다. 하드디스크는 고정된 크기의 EBS SSD 볼륨을 제공하고, 인스턴스는 정지된 상태에서도 비용이 청구되며&sup3;, Security Group 규칙은 덜 유연한데다, EC2의 설정들 중 일부만 접근 가능하다.

그리고 극적으로 단순화된 콘솔을 제공한다. 실제 작업이 EC2 위에서 동작하더라도 이것을 AWS Console의 EC2 영역에서 확인할 수 없다.
이 instance는 특수한 VPC안에서 자동으로 provisioning되지만 Console에서 보이지 않는다.
Lightsail은 이 숨겨진 VPC와 사용자의 default VPC간(같은 region일 경우)의 VPC Peering을 지원하여, Lightsail instance가 default VPC상의 EC2나 RDS와 같은 자원에 접근 가능하다.&sup2;

Lightsail의 bandwidth(네트워크 대역폭)는 무제한(물론 무료는 아님)이지만, 한 달에 무료로 상당한 양의 outbound bandwidth를 제공&sup3;한다.
또한 Routh53의 제한된 기능과 단순한 인터페이스를 맺을 수 있다.

그러나 Lightsail의 요점은 *단순화* 이기 때문에, 이러한 여러 제약들을 단점으로만 볼 수 없다.
EC2의 유연함이 결국 복잡함 유발하기 때문이다.
Lightsail의 대상은 AWS의 EC2, EBS, VPC, 그리고 Route53 같은 무수한 옵션들을 고려하고 싶지 않은 "간단한 VPS를 원하는" 고객들이다.
만약 private key를 이용해 SSH에 접속해야 하는 것(Lightsail은 built-in SSH client를 제공하지만, 반드시 사용할 필요는 없음, 표준 SSH client로 접근할 수도 있음)과 같은 기술적인 요구가 없다면, 사실상 기술 습득 시간(학습곡선)이 없기 때문이다.

----

&sup1;Lightsail instance는 단지 (VPC와 Classic의) "regular"한 EC2와 같다. 모든 Instance는 [instance metadata service][instance-metadata-service]에 접근하여 instance type이나 가용영역(AZ)와 같은 자기 자신의 메타 정보들을 확인할 수 있다. Lightsail instance의 메타 정보에 의하면 t2 머신이라는 것이 확인되었다.

&sup2;Lightsail의 문서에는 *Default VPC* 만 peering 동작한다고 명시적으로 표시되어 있지는 않다. 그러나 이것은 사실인 것으로 보인다. 만약 2013년 이전에 생성된 AWS 계정이라면, Default VPC라는 것이 없을 수도 있다. 그러나 이것은 다음 설명 [Can't establish VPC peering connection from Amazon Lightsail(at Server Fault)][vpc-peering-lightsail]에서 처럼 고객 지원으로 해결 할 수 있을 것이다.

&sup3;번역자 주석: Lightsail은 월별 과금 ([공식사이트][lightsail-pricing]), Instance별로 outbound 기본 무료용량이 제공되고, 초과분은 EC2의 데이터전송 비용과 동일하게 과금

기타: 현재(2017년 10월) Northern Virginia, Oregon, Ohio, London, Frankfurt, Ireland, Mumbai, Tokyo, Singapore, Sydney만 지원


[instance-metadata-service]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html
[vpc-peering-lightsail]: https://serverfault.com/a/818281/153161
[lightsail-pricing]: https://www.amazonlightsail.com/pricing/
[reference]: https://stackoverflow.com/questions/40927189/what-is-difference-between-lightsail-and-ec2
