---
layout: post
title:  "Ubuntu 18.04에서 한글 키보드 설정"
date:   2019-03-28 19:00:00 +0900
categories: Ubuntu
comments: true
---

# 새로 설치할 때마다 새로 찾아봐야 하는 한글 키보드 설정
Ubuntu 한글 키보드 너무 어려워...  
검색했던 것을 정리해 둔다.

## IBus
  * Ubuntu 16.04에선 잘 됐는데...
    + https://webnautes.tistory.com/1199
      - 이상하게도 안됨 (그냥 참고..)
    + http://snowdeer.github.io/linux/2018/07/11/ubuntu-18p04-install-korean-keyboard/
      - IBus, UIM 둘다 설명

## UIM
  * UIM 설치 및 적용
    + http://progtrend.blogspot.com/2018/06/ubuntu-1804-uim.html
      - 깔끔한 설명, 적용 성공!
      - 벼루(Byeoru) 사용
  * VS Code에서 "가" 관련 글자 깨짐 (강 -> 가ㅈ)
    + https://smok95.tistory.com/283
      - Droid Sans Fallback 폰트 제거

## 한영키, 한자키
  * CapsLock -> Ctrl
    + https://pragp.tistory.com/entry/Ubuntu-CapsLock%ED%82%A4-Ctrl%EB%A1%9C-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0
  * Alt_R 활성화
    + https://blog.wonkyunglee.io/26

## 다음엔 또 검색하지 않길...