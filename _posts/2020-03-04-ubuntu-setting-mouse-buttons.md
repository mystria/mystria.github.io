---
layout: post
title: "VMWare 위 Ubuntu에서 마우스 특수 버튼 활성화"
date: 2020-03-04 18:00:00 +0900
categories: Ubuntu VMware
comments: true
---
# VMware 상의 Ubuntu에서 마우스 특수 버튼을 활성화
WMware에서 여러 OS를 설치하여 동작시켜보면, 실제 마우스와 달리 아주 기본적인 기능만 동작한다.  
그러나 뒤로가기 버튼 등 특수 버튼을 활성화 시키기 위해서는 여러가지 작업이 필요한데, 여기저기 둘러보면서 의미 있었던 것만 정리하였다.  

## VMware가 특수 버튼 인식하게 하기
* VMware상의 OS(guest system)에서 input 확인, [참고](https://harrysekim.tistory.com/entry/Linux-%EC%9A%B0%EB%B6%84%ED%88%AC%EC%97%90%EC%84%9C-%EB%A7%88%EC%9A%B0%EC%8A%A4-%EB%B2%84%ED%8A%BC-%EB%A7%A4%ED%95%91%ED%95%98%EA%B8%B0)
  + 입력장치 목록 확인
    ~~~ sh
    $ xinput list
    ⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
    ⎜   ↳ Virtual core XTEST pointer              	id=4	[slave  pointer  (2)]
    ⎜   ↳ VMware VMware Virtual USB Mouse         	id=7	[slave  pointer  (2)]
    ⎜   ↳ VMware VMware Virtual USB Mouse         	id=8	[slave  pointer  (2)]
    ⎜   ↳ VirtualPS/2 VMware VMMouse              	id=10	[slave  pointer  (2)]
    ⎜   ↳ VirtualPS/2 VMware VMMouse              	id=11	[slave  pointer  (2)]
    ⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
        ↳ Virtual core XTEST keyboard             	id=5	[slave  keyboard (3)]
        ↳ Power Button                            	id=6	[slave  keyboard (3)]
        ↳ AT Translated Set 2 keyboard            	id=9	[slave  keyboard (3)]
    ~~~
    - pointer라고 되어 있는 입력 장치가 마우스
    - 목록 중에 하나 또는 여러개(하나는 이동, 하나는 클릭 등으로 분리)가 사용자의 마우스 임
  + 각 id 번호로 테스트를 시도해 마우스 입력 확인
    ~~~ sh
    $ xinput test 11
    button press   1 
    button release 1 
    button press   3 
    button release 3 
    ~~~
    - 기본적인 우클릭/좌클릭은 인식되지만, 특수 버튼은 인식이 안됨
  + 이는 VMware가 인식한 마우스 값을 guest system에 전달을 해주지 않기 때문으로 보임
* VMware의 vmx파일 수정
  + VMware UI상의 설정에서는 마우스 버튼 설정 변경을 할 수 없었음
  + 해당 이미지의 설정파일인 vmx파일을 에디터로 열어 수작업으로 설정 변경, [참고](https://askubuntu.com/a/441644)
    ~~~ txt
    mouse.vusb.enable = "TRUE"
    mouse.vusb.useBasicMouse = "FALSE"
    usb.generic.allowHID = "TRUE"
    ~~~
    - 위 옵션을 추가해야 함

## Ubuntu(16.04)에서 특수 버튼 인식하게 하기
* 본인은 16.04 LTS 버전을 사용하였지만, 대부분 비슷할 것으로 추측
* VMware에서 마우스 설정을 마치고, OS를 재시작하면 특수 버튼 인식 확인 가능
  + 입력장치 이름과 id가 바뀔 수 있음
    ~~~ sh
    $ xinput test 8
    button press   1 
    button release 1 
    button press   3 
    button release 3 
    button press   8 
    button release 8 
    ~~~
  + 8번 버튼이 추가로 인식 됨
* 추가 인식된 버튼에 기능을 매핑하기 위해 프로그램 설치 필요
  + 다음 사이트들의 가이드를 따라 적용
    - https://askubuntu.com/a/246849
      ~~~ sh
      $ sudo apt-get install xbindkeys xautomation x11-utils
      ~~~
    - https://harrysekim.tistory.com/entry/Linux-%EC%9A%B0%EB%B6%84%ED%88%AC%EC%97%90%EC%84%9C-%EB%A7%88%EC%9A%B0%EC%8A%A4-%EB%B2%84%ED%8A%BC-%EB%A7%A4%ED%95%91%ED%95%98%EA%B8%B0
      ~~~ sh
      $ sudo apt-get install xautomation xbindkeys xbindkeys-config 
      ~~~
  + 본인은 첫 번째 가이드로 진행
  + 다소 복잡한 설정이 필요해 보이지만 그냥 위 프로그램을 설치한 것만으로도 해결 됨
  





