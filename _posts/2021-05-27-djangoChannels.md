---
title: "[Python] Django Channels를 이용한 채팅 기능을 구현하며 학습한 내용 일부"
excerpt: ""

categories:
  - TIL
tags:
  - Python
  - Django
 
last_modified_at: 2021-05-27T01:50:00
toc: true
toc_sticky: true
---

#  Django Channels를 이용한 채팅 기능을 구현하며 학습한 내용 일부

## 자바스크립트 websocket

### 기존 서버와 브라우저의 통신방식

- HTTP 요청(request) 및 HTTP 응답(Response)
- 새로운 데이터를 받아서 DOM을 다시 렌더링 하거나, 또는 브라우저를 새로 고침하여 전체를 렌더링
- 원하는 부분만 실시간으로 상호 데이터를 교환하여 새로고침 없이 렌더링하는 기술로 발전하면서 비동시 통신기술을 많이 사용하게 되면서, 브라우저와 웹 서버 사이에서 양방향 메시지 송수신 할 수 있는 기술인 HTML5 WebSocket이 등장

### Ajax Polling과 WebSocket의 비교 ([출처](https://blog.resellerspanel.com/hepsia-control-panel/supervisor-run-background-processes.html))

![Websocket vs Ajax polling scheme](/assets/post_images/2021-05-27-djangoChannels.assets/websocket-scheme.png)

가장 큰 차이점은, WebSocket은 기존 요청-응답 관계에서 벗어난 양방향 송수신이 가능하다는 점이다.



## Redis를 쓰는 이유 

### Redis란

NoSQL 데이터베이스의 하나이다. 특징으로는 

1. Remote에 위치한
2. 프로세스로 존재하는
3. In-Memory (메모리 기반의)
4. 키-값 구조의 데이터 관리 시스템 : 비관계형이고, 키-값 구조이기 때문에 별도의 쿼리 없이 데이터를 간단히 가져올 수 있음

서비스의 특성이나 상황에 따라 1) 캐시로 사용하거나, 2) Persistence Data Storage로 사용할 수 있다. 캐시로 사용하면 더욱 빠른 채팅 서버를 구성할 수 있을 것으로 기대된다.



### django channels에서 Redis의 역할 => Channel Layers

- he primary purpose of redis in django-channel_layers is to store the necessary information required for different instances of consumers to communicate with one another.
- In the [tutorial](https://channels.readthedocs.io/en/latest/tutorial/index.html) section of channels documentation, it is clear that Redis is used as a storage layer for channel names and group names.
- django 공식 문서에서 권장하는 방법 외에 메세지 큐로도 사용이 가능하다. 하지만 해당 task가 완료됐는지는 알려주지 않는다.
- Redis 뿐만 아니라 In-Memory Channel Layer도 지원하는데, Django 공식적으로는 테스팅, 개발 단계에서만 사용할 것을 권장한다.



### 메세지 큐

#### 메시지 지향 미들웨어(Message Oriented Middleware: MOM)

비동기 메시지를 사용하는 다른 응용 프로그램, 시스템, 서비스 사이에서 데이터를 송수신하는 것을 의미

#### 메시지 큐(MessageQueue: MQ)

MOM을 구현한 시스템. 데이터를 교환할 때 시스템이 관리하는 메시지큐를 이용하는 것이 특징이다.

#### 메세지 큐의 장점

- 비동기 : 큐에 넣기 때문에 나중에 처리
- 비동조 : 어플리케이션과 분리 가능
- 탄력성 : 일부가 실패해도 전체에 영향을 주지 않음
- 과잉 : 실패할 경우 재실행 가능
- 보증 : 작업이 처리된 것을 확인 가능
- 확장성 : 다수의 프로세스들이 큐에 메시지를 보낼 수 있음



#### 메세지 큐를 사용한 비동기적인 채팅 서버를 구현해야하는 이유

사용자 또는 데이터가 많아지면 요청에 대한 응답을 기다리는 수가 증가하다가 나중에 대기시간이 길어져 서비스가 정상적으로 이루어지지 못하는 경우가 발생할 수 있다. 기존에 분산되어있던 데이터 처리를 한 곳에서 집중하고 메세지 브로커를 두어서 필요한 프로그램에 작업을 분산시키는 것이 그 목적이다. 만약 멀티스레드라면 여러 스레드가 큐에 담긴 task를 앞에 작업이 끝났거나 말거나 처리할 수 있다.



### channels 튜토리얼 따라할 때 발생하는 문제 

#### Channels-redis 관련 오류

- redis 통신 오류가 나는데, 이를 해결하려면 channels-redis는 2.x 버전을 사용해야 오류가 나지 않는다.
- 그런데 channels-redis를 2.x 버전으로 설치하면 channels도 2.x 버전으로 자동설치되는데, 어거지로 3.x 버전으로 바꿔줘야 as.asgi()가 동작한다. 근데 그러면 의존성 문제가 발생한다고 에러가 난다.

#### Windows에서 개발 후 리눅스 EC2에서 배포시 발생하는 IOCP 관련 이슈

- WIndows에서 Channels를 설치하면 비동기 작업을 수행하기 위하여 IOCP 관련 라이브러리(twisted-icopsupport)가 함께 설치된다. 그런데 이것은 Windows에서만 사용되는 것인데, windows에서 requirements를 만들고 그대로 Linux에서 install하게 되면 리눅스에서는 설치할 수 없는 라이브러리이기 때문에 에러가 난다. 그래서 이 부분을 requirements.txt에서 제거해주고 pip install 해줘야한다.



## daphne와 Nginx를 사용한 배포 문제

[victolee님의 블로그](https://victorydntmd.tistory.com/265)를 적극 참고하였다.



## 참고

- https://developer.mozilla.org/ko/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications

- https://stackoverflow.com/questions/63754950/what-role-does-redis-serve-in-django-channels
- https://sugerent.tistory.com/644
- https://velog.io/@hyeondev/Redis-%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C
- [Django channels 공식문서](https://channels.readthedocs.io/en/stable/)

