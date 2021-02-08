---
title: "[Vim] Pycharm에서 vim plugin을 사용할 때 복사/붙여넣기가 되지 않는 경우 : 클립보드 일치 시키기 "
excerpt: ""

categories:
  - 끄적이
tags:
  - vim
 
last_modified_at: 2021-02-08T13:00:00
toc: true
toc_sticky: true
---


pycharm에서 vim plugin을 사용할 때 다른 곳에서 복사한 것을 pycharm에 붙여넣거나, 반대로 pycharm의 내용을 밖에다 붙여넣는 것이 되지 않을 것이다. vim과 시스템 클립보드가 일치하지 않아서 생기는 현상이다. 

이를 해결하기 위해서는 홈디렉토리에 `.ideavimrc` 파일을 만들고 `set clipboard+=unnamed` 라고 입력 후 저장하고 파이참을 재실행하면 동일한 클립보드를 사용할 수 있다. 이는 plugin이 아닌 터미널에서 vim 쓸 때도 마찬가지의 설정이다. 



vim에 대해 더 알아보고 싶다면 이 [링크](https://yongbbbba.github.io/%EB%81%84%EC%A0%81%EC%9D%B4/vim/)로 ..