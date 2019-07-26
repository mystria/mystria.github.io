---
layout: post
title:  "금주의 실패사례 - DB가 안되었던건 디스크 용량 문제"
date:   2019-07-26 16:00:00 +0900
categories: Linux
comments: true
---
# 갑자기 batch작업 서비스가 멈춤
개발용 가상 서버 위에 세 시간에 한 번 씩 동작하는 서비스가 있다. 이 서비스는 DB에 테이블을 생성하고 데이터를 쓰는 작업을 한다.  
그런데 어느날 이 서비스가 멈춰있고, 프로세스가 쌓여 서버 메모리를 가득 채우고 있었다. 무슨 일이 생긴 것일까...?  
복구 작업을 하면서 겪었던 노가다를 대충 정리하고자 한다.  

## 원인 분석 (요약)
  * 배치로 수행 중인 서비스의 결과물이 나오지 않고 있기에 다음과 같은 순서로 점검해 보았다.
    1. 서비스가 동작하는 서버 접속
    2. 서버에서 직접 서비스 호출 - 서비스가 반응이 없다 - 단순히 시간만 오래걸리는 것일까?
    3. 프로세스 확인 - 프로세스가 쌓여있다(때마다 호출 되었지만, 끝나지 않았음)
    4. 시스템 자원 확인 - 메모리를 잔뜩 쓰고 있다.
    5. 프로세스 죽임 - 메모리가 원래대로 돌아왔다.
    6. 다시 서비스 직접 호출 - 여전히 반응 없음
    7. 서비스 로그 확인 - DB 접속 중 pending
    8. 관련 DB 확인
    9. 어라.. DB도 메모리가 가득 찼네?
  * 아무래도 DB 관련 호출에서 문제가 있는 것 같았다.
    1. 해당 DB를 접근하는 모든 서비스를 차단 - 여전히 메모리가 full
    2. 해당 DB 서버를 과감하게 재시작
    3. 기동이 안됨.......-_-;;
    4. 서버 접속 - 디스크 full 에러 발생(No space left on device)

## Linux의 용량을 살려보자
  * DB서버는 Linux 위에서 동작한다. 위에서 다소 과감하게 작업했지만 이는 개발 환경이기 때문에 가능하다.
  * 다음 웹사이트 들이 도움이 많이 되었다.
    + 프로세스 한꺼번에 죽이기
      - [Linux 여러 프로세스 한꺼번에 죽이기](https://bloodguy.tistory.com/entry/Linux-%EC%97%AC%EB%9F%AC-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%ED%95%9C%EA%BA%BC%EB%B2%88%EC%97%90-%EC%A3%BD%EC%9D%B4%EA%B8%B0)
      - $ ps ax | grep httpd | awk '{print $1}' | xargs kill
    + 용량 정리를 위해 불필요한 파일 지우기
      - 디스크가 가득 찬 상황에서는 여러 기능이 제대로 동작하지 않는다. 우선 공간을 마련하자.
      - $ df -h
      - $ df -i
      - $ sudo du -ckx
      - http://amazingguni.github.io/blog/2016/05/devops-chapter-4-disk
      - $ for i in /home/*; do echo $i; find $i |wc -l; done
      - https://blog.1day1.org/493
  * 우선 공간을 확보 했으니 근본적인 문제를 해결해 보자

## 가상 환경의 Linux 시스템 용량 확장하기
  * 사실 아래 링크들을 보관해 두기 위해 이 글을 쓴 것...
  * 개발용 가상 서버이기 때문에, 용량 확장이 언제든지 가능하다. 그러나 이를 Linux에 적용하기 위해서는 다소 복잡한 절차가 필요하다.
  * 첫 번째 시도
    + [리눅스 시스템에서 디스크 용량 확장하기](https://blog.lael.be/post/7735)
  * 두 번째 시도
    + [LVM 기반 Ubuntu Disk 확장](https://youngmind.tistory.com/entry/LVM-%EA%B8%B0%EB%B0%98-Ubuntu-Disk-%ED%99%95%EC%9E%A5)
  * 간단 요약
    + 가상 하드 디스크 확장
    + vgs, vgdisplay - VolumeGroup 확인
    + pvs, pvdisplay - PhysicalVolume 확인
    + lvs, lvdisplay - LogicalVolume 확인
    + fdisk - LVM구성 확인
    + fdisk로 파티션 생성
    + pvcreate - PV 생성
    + vgextend - VG 확장
    + lvextend - LV 확장
    + resize2fs - 수정사항 적용

## 마무리
  * 결과적으로 DB가 동작하는 서버의 디스크가 가득차서 내부적으로 메모리가 가득차게 된 것 같다. 이렇게 되면 DB접속(connection)은 어떻게든 되는데, DDL(Data Definition Language) 부분이 잘 동작하지 않았던 것이다.
  * 디스크를 늘였더니 해결 됨.. 끗