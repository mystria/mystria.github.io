---
layout: post
title:  "Cloud로 Migration하는 6가지 전략 (6Rs)"
date:   2021-11-30 20:00:00 +0900
categories: AWS Migration
comments: true
---

# AWS Migration 6R strategies
최근 AWS Solutions Architect Professional 자격시험을 준비하면서 On-Premise 의 application 을 AWS 로 이전(migration)관련된 많은 사례를 보게된다. 주요 문제들이 migration 방법론이다.  
그 방법론을 6가지 전략으로 구분하였는데, 이것이 AWS 만의 고유 분류인지 cloud 분야의 공통된 분류인지는 모르겠지만 대충 공통 전략으로 봐도 충분할 것 같다.  

## 여섯가지 R (6Rs)
Rehost - Replatform - Repurchase - Refactor - Retire - Retain  
[AWS 블로그 문서](https://aws.amazon.com/ko/blogs/enterprise-strategy/6-strategies-for-migrating-applications-to-the-cloud/)
* Rehost(Rehosting) : 속칭 lift-and-shift, 말 그대로 떠옮겨서 내려놓는 모양세, On-Prem 에 대응되는 resource 들을 사용하여 옮김
* Replatform(Replatforming) : 다른 말로 lift-tinker-and-shift, 살짝 수정해서 옮기는 정도, AWS에서 제공하는 서비스, RDS나 ElasticBeanstalk 등으로 대응하여 옮기는 것
* Repurchase(Repurchasing) :  drop-and-shop, 그냥 SaaS 서비스를 구매하여 거기에 입주하는 방식, Salesforce, Workday, Drupal 등 이미 존재하는 서비스를 "사서" 사용하는 것
* Refactor(Refactoring/Rearchitecturing) : 새로 개발 하는 것, 현재 기능을 cloud에 맞게 새롭게 설계하여 만들기. 강력한 비즈니스적인 요구사항(성능, 확장성 개선 등)이 있을때나 가능
* Retire : 말그대로 그냥 은퇴시킴
* Retain : 그냥 아무것도 하지 않고 남겨두는 것 (이게 제일 편한 전략?!)

## 전략별 난이도
[참조](https://cloud.netapp.com/blog/aws-migration-strategy-the-6-rs-in-depth)에 따르면 각 R 별로 난이도가 있는데,  
Retire와 Retain은 제외하고, Repurchase → Rehost → Replatform → Refactor 순으로 어렵다고 한다.  
난이도나 비용을 고려하는 것도 중요하겠지만 migration 할 대상이 어떤 특성이 있는지 분석하여 적합한 전략을 찾는게 중요할 것이다.  

## 사례별 전략
자체 데이터 센터를 cloud 로 migration 한다는 것은 유지 관리 비용 절감과 안정성 보완은 기본으로 깔고 간다. 아래는 개인적인 사례별 전략 분석.
* Repurchase: 사내용 관리 솔루션 등, 별도의 관리 인력을 두는 것이 손해인 경우
* Rehost: 독특한 구조라 손보기도 어렵고 그대로 쭉 사용할 legacy
* Replatform: 당분간 기능적으로 큰 개편이 없지만 cloud 의 fully-managed 한 자원의 이점을 노리는 경우
* Refactor: '차세대'로 재개발하는 경우