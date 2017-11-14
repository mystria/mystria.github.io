---
layout: post
title:  "Google G-Suite, Gmail Migration"
date:   2017-11-03 16:00:00 +0900
categories: Migration
comments: true
---
# 이번 주 Gmail을 사용하면서 겪은 실패 사례
Gmail에서 수천건 이상의 메일을 다른 사람에게 전달해야 하는데 어떡하지?

## 개요
  * Gmail의 자동 Forward 기능
  * Gmail의 Multiple Forward 기능
  * Gmail의 IMAP 또는 POP를 통한 가져오기 기능
  * G-Suite의 Migration 기능

## Gmail의 자동 Forward 기능
  * Gmail의 '환경설정(Settings)'의 '필터 및 차단된 주소(Filter and Blocked Addresses)' 탭에는 특정 조건에 해당하는 메일들을 필터링 할 수 있는 기능이 존재
  * 이 기능에서 필터링 후 해당 메일을 어떻게 할지 처리방법을 지정 가능한데, 이때 '전달(Forward)'를 선택하면 대상 주소로 메일 그대로 전달이 가능함
    + 메일 제목 앞에 FW같은 문자가 붙지않고, 발신인이 바뀌지 않은 메일을 그대로 전달함(마치 원래 본인에게 보내진 메일처럼)
    + 단, 사전에 동의를 구한 등록된 주소로만 전달 가능
  * 이 기능을 이용하여 한 메일 주소를 창구로 하여 주제 및 발신인, 메일 주소(Alias) 별 배분이 가능하기 때문에 유용하게 활용 가능
  * 그런데 이 기능을 실수로 적용하지 않았다면?!
    + 이 기능을 활성화 하더라도, 활성 시점 이후 메일에만 적용될 뿐, 기존 메일은 전달 되지 않음
    + 그런데 기존 메일을 전달해야 한다면?!

## Gmail의 Multiple Forward 기능
  * 기존의 Gmail에는 여러 메일을 선택 후 한번에 전달하는 것이 불가능
    + 지금 다시 찾을 수 없느나, migration 방법을 검색 도중 Multi-Forward기능을 보안문제로 오래전에 차단했다는 글을 본 것 같음
  * 그래서 현재는 Chrome Extensions을 이용해 여러 메일을 선택하여 한 번에 전송하게 해주는 확장앱이 많이 출시 됨
    + CloudHQ의 [Multi Email Foward for Gmail][cloudhq-forward-multiple-emails]
      - 가입이 필요하고, 무료 고객일 경우 하루에 200건만 보낼 수 있음
      - 메일을 선택하면 화면에 전달버튼이 생기고,
      - 메일 발송 시 batch로 하나씩 메일을 전달하는 방식
      - 해당 앱에 Gmail 일부 권한을 OAuth로 허용해야 함(좀 찝찝)
    + 더 있지만 안써봄... 위와 비슷할 듯
  * 이 경우 문제점은 발송량도 적을 뿐더러, 시간이 오래걸리고, 메일 제목에 FW가 붙고 발신인이 본인으로 변경됨
    + 메일 전달이니까 당연한 증상

## Gmail의 IMAP 또는 POP를 통한 가져오기 기능
  * Gmail의 '환경설정(Settings)'의 '전달 및 POP/IMAP(Forwarding and POP/IMAP)' 탭에는 현재 Gmail의 메일을 외부에서(예를들면 Outlook이나 다른 메일 앱, 또는 메일 서비스) 가져갈 수 있도록 허용해주는 기능이 존재
    + 해당 화면에서 '자세히 알아보기(Learn more)'를 확인하면 상세한 사용법이 설명되어 있음
      - [IMAP][gmail-imap-setting]
      - [POP][gmail-pop-setting]
    + 메일을 제공할 때 원본을 남기도록 설정해 두어야 원본이 사라지지 않음
  * 또한, Gmail의 '환경설정(Settings)'의 '계정 및 가져오기(Accounts)' 탭에는 위 처럼 가져갈 수 있도록 허용된 메일 서비스의 메일을 가져올 수 있음
    + '다른 계정에서 메일 확인하기(Check mail from other accounts)'에 가져올 메일 계정 등록
      - POP나 IMAP 주소, ID, Password를 입력해야 함
      - 가져올 때 원본을 남기도록 설정해 두어야 원본이 사라지지 않음(POP는 제공측에서만 설정가능 한듯)
    + 이 기능을 이용해 Gmail에서 다른 모든 메일 계정(네이버, 다음, 네이트 등등)을 한 곳에서 볼 수 있음
  * 즉, 한 Gmail 계정은 메일을 가져갈 수 있게 개방하고, 다른 Gmail 계정에서 가져오기를 수행하면 됨
  * 온전한 메일과 발신인의 정보를 간직한채 그대로 전달이 가능함
  * 이 경우 문제점은 모든 메일을 가져오게 되어 불필요한 메일도 가져오고, 시간이 엄청 오래걸림
    + 게다가 과거 메일 부터 가져오기 때문에 최근 메일을 처리하기 위해서는 끝까지 기다려야 함
    + 메일을 처리하고나면 바로 바로 삭제하는 사용자라면 유용할 듯

## G-Suite의 Migration 기능
  * 만약 G-Suite(Google for Works)를 사용중이라면 [Data migration][gsuite-email-migration] 기능을 활용 가능
  * 관리자(Admin) 메뉴에서 Data migration을 선택하면 메일 migration가능(주소록과 일정도 지원)
    1. Migration 할 대상 선택 - Email
    2. Migration source 입력 (G Suite, Gmail, GoDaddy, MS Exchage 외 IMAP 지원)
      - Gmail일 경우 Migration source만 선택하면 됨
    3. Connection protocol은 Auto로...(G-Suite의 경우 IMAP으로 지정됨)
      - 즉, source의 전달 및 POP/IMAP에서 IMAP을 활성화 시켜놔야 함
    4. Role account 입력 - 해당 작업을 수행할 G Suite계정의 메일주소와 암호
    5. Migration start date 선택 : 이게 가장 좋은 장점
    6. Migration options 선택 : 필요한 label만 옮길 수 있음
    7. Migration을 수행할 계정 및 대상 계정 입력
      - From 계정 정보(메일 주소와 암호)를 입력
      - To 계정 정보(메일 주소)
  * 위 과정을 수행하고나면 옮겨야할 메일을 카운팅하고, 백그라운드에서 migration 수행
    + 가끔 실패했다 재시도 하면서 아주 천천히 진행 됨
    + 개인적 경험으로 2000건 정도 옮기는데 4~5시간 정도 소요된 듯
  * 이 경우 장점은 특정 기간(start date)에 대해서 옮길 수 있고, 옮길 대상(label)도 지정 가능
    + 그러나 G-Suite는 유료 서비스
    + Migration이 끝나면 해당 메뉴에서 Exit migration을 해야함
      - Exit하지 않아도 지속적으로 수행되진 않음

## 결론
새로 오는 메일은 Filtering and Forward로 처리하고, 미처 옮기지 못한 메일은 G-Suite의 Data Migration을 이용

[cloudhq-forward-multiple-emails]: https://blog.cloudhq.net/forward-multiple-emails-gmail/
[gmail-imap-setting]: https://support.google.com/mail/answer/7126229
[gmail-pop-setting]: https://support.google.com/mail/answer/7104828
[gsuite-email-migration]: https://support.google.com/a/answer/6351474
