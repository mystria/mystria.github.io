---
layout: post
title:  "EKS에서 동작하는 K8s의 Pod에 Elastic IP 할당하기"
date:   2018-11-22 17:00:00 +0900
categories: Kubernetes EKS EIP AWS
comments: true
---
# EKS로 동작하는 K8s Pod에 EIP를 붙여 보자
Kubernetes에서는 Service를 통해 cluster 내에서 동작하는 서비스를 외부로 노출한다.  
(서비스를 Service로 노출하다니!)  
이 Service는 platform의 영향을 받게 되는데, EKS는 Load Balance로 할 경우 Elastic Load Balancer를 붙여 준다.  
그 외에도 Cluster IP나 Node Port 등이 존재하는데, 이에 대한 설명은 생략하고, Service가 아닌 Pod에 Elastic IP를 붙일 순 없을까 생각해 봤다.  
(정확하게 말하자면, out-bound의 IP를 고정 IP로 지정하기)  

## Pod에 고정 IP 필요한 경우
매우 드문 경우이겠지만, 몇몇 서비스 들은 IP주소로 접근을 제한하기도 한다.  
IP 필터링으로 보안을 강화 하는건 당연한 것이지만, 이번 경우는 IP만 열려있으면 별도의 인증없이 서비스 이용이 가능하다는 것이다.  
그레서 IP가 매우 중요한 인증수단이 되버리는데, AWS에서는 고정 IP를 받기위해서 Elatic IP를 써야한다. 그것이 Kubernetes 일지라도...

## Pod 정보 받기
  * kubectl을 이용하여 pod에 할당된 IP를 확인
    ~~~ ssh
    $ kubectl get pod
    NAME                                    READY   STATUS    RESTARTS   AGE
    your-pod-name-unique-randomdvalue       1/1     Running   0          38d

    $ kubectl describe pod/your-pod-name-unique-randomdvalue
    Name:           your-pod-name-unique-randomdvalue
    Namespace:      default
    Node:           ip-99-99-99-11.ec2.internal/99.99.99.11
    Start Time:     Fri, 02 Nov 2018 20:24:16 +0900
    Labels:         pod-template-hash=2463000000
                    run=your-pod-name
    Annotations:    <none>
    Status:         Running
    IP:             99.99.99.222
    Controlled By:  ReplicaSet/your-pod-name-replica
    Containers:
    ...
    ~~~
    + IP는 Node가 배포된 VPC의 IP range안에서 랜덤하게 할당됨(Private IP)

## Pod의 Private IP를 이용해 ENI찾기
  * Elastic Network Interface(이하 ENI)
    + Pod에게 private IP가 있다는 말은 ENI가 존재한다는 의미와 동일
    + AWS EKS는 kubernetes의 모든 동작을 지원하는데, AWS의 resource를 적극 활용함
      - Storage는 EBS를 활용하는 것 처럼 가상 네트워크 인터페이스 조차도 AWS 자원(ENI)을 활용하네...
  * AWS Console에서 EC2 > Network Interface 검색
    + ENI의 ID 획득

## Elastic IP 설정
  * Elastic IP(이하 EIP)
    + EIP를 할당(allocate)
    + Associate할 대상 선택 시, Resource type을 Instance가 아닌 Network interface로 선택
      - Pod은 한 worker node(= instance) 아래에 여러개가 존재
      - 개별 pod은 각자에게 할당된 ENI에 연결
      - 만약 Instance로 지정하려 하면 설정 적용이 실패
    + ENI에 EIP를 붙이고 나면, 해당 pod에서 나가는 traffic의 source는 EIP 주소가 됨

## 문제점
Pod이 죽으면 ENI도 사라져 EIP는 (아마)주인을 잃게되고 비용만 발생하게 된다.  
(EIP는 무료이지만, 할당 후 사용하지 않으면 비용이 발생함)  
Pod 이라는 생사를 보증할 수 없는 가상의 존재에 고정 IP를 부여하고, 이 IP로만 인증 해야하는 아키텍처는 지양해야 마땅하겠다.  
하지만, 이 바닥은 늘 이상적으로 돌아가지 않으니 Pod이 바뀌거나 scale out/in 되더라도 유지할 수 있는 방법을 고민할 필요가 있을 것이다.  
  * Ingress는 있는데 Egress는?
    + Azure에는 있네? 
    + [AKS(Azure Kubernetes Service)의 송신 트래픽에 고정 공용 IP 주소 사용](https://docs.microsoft.com/ko-kr/azure/aks/egress)
    + 혹시 EKS에도 있는데 나만 모르는것 아닐까?
  * Egress Proxy를 만들어 EIP를 할당하면 가능할까?
    