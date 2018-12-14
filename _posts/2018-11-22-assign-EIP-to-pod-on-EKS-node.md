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
그 외에도 Cluster IP나 Node Port 등이 존재하는데, 이에 대한 설명은 생략하고, Elastic IP를 붙일 순 없을까 생각해 봤다.  

## Pod에 고정 IP 필요한 경우

## Pod 정보 받기

## Pod의 Private IP를 이용해 ENI찾기
ENI(Elastic Network Interface)

## EIP 설정
설정할 때 Instance가 아니라 Network로 해서 ENI에 연결


no basic auth credentials일때 aws-auth-cm.yaml 다시 실행

