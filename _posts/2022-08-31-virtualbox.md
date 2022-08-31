---
title: "[Windows11] virtualbox 설치 및 머신 추가 난항기 - 구버전(5.x)설치와 Hyper-V와 관련된 에러"
excerpt: ""

categories:
  - TIL
tags:
  - virtualbox
 
last_modified_at: 2022-08-31T18:30:00
toc: true
toc_sticky: true
typora-root-url: ../
---



# 문제

virtualbox를 이용해서 토이 플젝 하나를 시도하고 있었는데 두 가지 문제가 발생하였다. 



1. 참고하고 있던 자료에서 virtualbox 5.x 버전을 설치하라고 하였으나 설치를 시도하면 아래 이미지와 같이 나왔던 것 같다. `이 앱은 이 장치에서 실행할 수 없습니다.` 

![윈도우11 : 코어격리 메모리 무결성 의 "이 설정은 관리자가 관리합니다" 해제 방법 :: 지누의 글](/_posts/2022-08-31-virtualbox.assets/img.png)



2. virtualbox에서 머신을 추가해서 centos를 설치 후 실행하려고 하였는데 실행이 되지 않았다.  이건 두 가지 문제가 있었다. 
   1. `Raw-mode is unavailable courtesy of Hyper-V` 와 함께 실행이 되지않음
   2. 로그를 까서 error를 검색해보면 fasoo DRM이 어쩌고, C:\WINDOWS\system32\wintab32.dll이 어쩌고가 나와있음



이것 때문에 환경세팅에서 삽질에 삽질을 거듭하였다. 구글링과 계속 지우고 깔고를 반복.. 



## 해결 방법

1. 메모리 무결성을 꺼주니까 해결됨

​	![img](/_posts/2022-08-31-virtualbox.assets/img-16619375704164.png)

위와 같이 코어 격리의 메모리 무결성을 꺼주고 재부팅을 하고 나서는 구버전도 정상적으로 설치가 되었다.



2. fasoo DRM을 삭제하고 관련 디렉토리도 다 삭제하고 레지스트리에서도 다 삭제하고, hyper-v도 꺼줬더니 해결됨

   다만, hyper-v 끄는 문제에서 애를 먹었는데 `제어판 -> 프로그램 -> 프로그램 및 기능 -> Windows 기능 켜기/끄기`에서 분명 Hyper-V가 꺼져있었는데도 불구하고 자꾸 `Raw-mode is unavailable courtesy of Hyper-V` 에러 문구가 발생하여 가상머신이 실행되지 않았다. 그래서 추가로 cmd에서 `bcdedit /set hypervisorlaunchtype off`를 실행시켜주고 나니까 다신 에러 문구가 뜨지 않고 정상적으로 가상머신이 실행되었다. 

