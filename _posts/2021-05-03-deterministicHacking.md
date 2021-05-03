---
title: "[CS][게임네트워킹의 이해] 8. Deterministic과 해킹"
excerpt: ""

categories:
  - TIL
tags:
  - CS
  - Network
 
last_modified_at: 2021-05-03T22:20:00
toc: true
toc_sticky: true
---

# Deterministic과 해킹

## 왜 Deterministic이 해킹에 취약한가?

- Input set에 기반하고 있다.  참여자들이 Input을 똑같이 공유하고, 상대방이 보낸 Input에 의존할 수 밖에 없다. 상대방의 Input을 인정해야한다. 그래서 Input data를 조작하는 등의 해킹에 취약할 수 밖에 없다.
- 따라서 Input set을 검증하는 것이 필요하다
  - 서버, 방장, 서로가 검증하는 방식 등이 있다.



## 서버를 이용하는 방식

- 중계서버를 두게 되면 모든 클라이언트의 패킷이 중계서버를 거치고, 이를 뿌리기 전에 서버에서 검사를 할 수 있다.
- 이 방법이 가지는 문제점은, 패킷을 검사하려면 당연히 현재 게임의 상태, 상황과 룰에 대해 알고 있어야 한다.
  - 예를 들면 벽 너머에 플레이어가 있는 상태에서 벽에다 총을 쏘면 상대가 맞아서는 안된다는 그런 룰들. 이런 것들을 서버는 알고 있어야 한다.
  - 중계서버는 state, rule 등이 없는 stateless 서버가 있는데, state가 없는 중계서버를 유지할 수 있다면 하나의 중계서버를 가지고 여러 게임을 동시에 서비스 할 수 있다.
  - 그런데 중계서버가 게임 상태와 룰을 가져야 한다고 하면 게임마다 중계서버가 달라져야 하기 때문에 같은 중계서버로 여러 게임을 서비스 할 수 없는 단점이 생기게 된다.
- 또한 FPS는 공격쪽에 유리하게 되어있다.
  - 이전에 말했듯이 Deterministic은 딜레이와 롤백을 사용하기 때문에 일정 시간 과거의 모습을 보게 되는 경우가 있다. 내가 상대를 발견하고 총을 갈겼는데, 이게 딜레이가 있어서 제대로 맞지 않아버리면 빗나가는 경우가 자주 있기 때문에 이런 오차를 인정해서 맞은 것으로 판정을 해버린다. 이런 특성을 이용해서 에임핵, 이동핵, 스핀샷들이 생기게 된다.
  - 이런 오차가 있기 때문에 심판 입장에서 플레이어가 핵을 쓰고 있는건지 아닌지 판단하기가 곤란한 경우가 많아진다.



## 해킹의 방식 : 내부해킹 vs 외부해킹

### 내부해킹

- exe 파일을 조작
- 내부 코드를 조작
- 게임회사와 해커들의 관계에서 게임 클라이언트 파일은 일종의 인질과 같다.
- 클라이언트 프로그램은 기계어로 되어있고, 다시 말해 이것도 코드이다. 역어셈블리를 통해서 기계어가 어떤 역할을 하고 있는지 유추할 수도 있다.
  - 기계어에서 암호화하는 부분을 발견한다면 그 부분을 패스하거나 뭐 조작하거나 하는 방식으로 해킹할 수 있다.
- 조작여부 판단하는 방법?
  - 파일 사이즈가 변경되었는지 확인
    - 하지만 이 경우에도 더미 데이터를 넣는다던가 하는 방식으로 정확하게 파일 사이즈를 맞출 수도 있는데 이럴 때는 checksum의 방식으로 검증할 수 있음
    - checksum은 전체 코드를 hash화 해서 코드가 변경되면 hash가 크게 변하기 때문에 해커가 발견하기 어려운 곳에 hash값을 숨겨놓는다.



## 외부해킹

- 게임 외부에 해킹 프로그램을 실행시키는 방식
- 외부 툴을 못 띄우게 하는 블랙아웃 방식으로 해킹 프로그램을 제어
  - mProtect 등



## 정리

- Deterministic 뿐만 아니라 Server Authority방식도 해킹에 취약하기는 마찬가지다.
- 하지만 Deterministic 방식은 input set에 의존하고, Server Authority는 기본적으로 서버와 로직을 들고 있고 서버가 명령하는대로 수행하고, 기본적으로 서버가 심판역할을 하기 때문에 Server Authority 방식이 Input set을 조작하는 형태의 해킹에서는 강력하다.

## 출처

- [Tucker Programming](https://youtu.be/11Ilw4NcZM8)