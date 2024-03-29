---
title: "[Web] Web Socket 다시 간단히 복습해보기"
excerpt: ""

categories:
  - TIL
tags:
  - Web
  - Web Socket
 
last_modified_at: 2021-06-09T22:00:00
toc: true
toc_sticky: true
---



# Web Socket 다시 간단히 복습해보기

저번 Channels 공부하면서 web socket에 대해 공부를 좀 했었지만 10분 짜리 영상을 발견해서 다시 복습할겸 영상을 시청하였다.



## 웹 소켓이란

웹 소켓이란 두 프로그램 간의 메시지를 교환하기 위한 통신 방법 중 하나이다.

(W3C와 IETF에 의해 자리잡은 표준 프로토콜 중 하나)

웹 소켓을 지원하는 브라우저의 경우 웹 소켓 포로토콜을 지원한다.



## 웹 소켓의 특징

1. 양방향 통신 (Full-Duplex)
   - 데이터 송수신을 동시에 처리할 수 있는 통신 방법
   - 클라이언트와 서버가 서로에게 원할 때 데이터를 주고 받을 수 있다.
   - 통상적인 http 통신은 client가 요청을 보내는 경우에만 server가 응답하는 단방향 통신
2. 실시간 네트워크(Real Time-Networking)
   - 웹 환경에서 연속된 데이터를 빠르게 노출
   - ex) 채팅, **주식**, 비디오 데이터
   - 여러 단말기에 빠르게 데이터를 교환



## 웹 소켓 이전의 비슷한 기술

- Polling
  - 서버로 일정 주기 요청 송신
  - real-time 통신에서는 언제 통신이 발생할지 예측이 불가능
  - 불필요한 request와 connection을 생성(바뀐게 없는데도 request를 함)
- Long Polling
  - 서버에 요청 보내고 이벤트가 생겨 응답 받을 때까지 연결 종료 x
  - 응답 받으면 끊고 다시 재요청
  - 많은 양의 메세지가 쏟아질 경우 Polling과 같아진다
- Streaming
  - 서버에 요청 보내고 끊기지 않은 연결 상태에서 끊임없이 데이터 수신
  - 클라이언트에서 서버로의 데이터 송신이 어렵다



결과적으로 이 모든 방법이 HTTP를 통해 통신하기 때문에 Request, Response 둘 다 Header가 불필요하게 크다.



## 웹 소켓의 동작 방법 - 핸드 쉐이킹

웹 소켓의 핸드쉐이킹은 http 또는 https를 통해서 이뤄진다.



## 웹 소켓의 동작 방법 - 데이터 전송

핸드 쉐이킹을 통해 연결이 확인이 됐으면 ws로 프로토콜이 변경된다.



### Frame과 Message

frame: communication에서 가장 작은 단위의 데이터(작은 헤더+payload 구성)

message: 여러 frame이 모여서 구성하는 논리적 메세지 단위

웹 소켓 통신에 사용되는 데이터는 UTF-8 인코딩



### 웹 소켓과 HTTP

두 프로토콜은 모두 OSI 모델의 7번째 계층(Application)에 있으며, TCP의 4번째 계층에 의존하고 있지만 웹 소켓은 HTTP 포트 80 및 443 이상으로 실행되도록 설계되었으며, HTTP 프록시 및 중간 계층을 지원하도록 설계되어 HTTP 프로토콜과 호환된다.

웹 소켓을 위한 별도의 포트는 없으며, 기존 포트(http-80, https-443)을 사용한다.



## 웹 소켓의 한계

- 웹 소켓은 문자열들을 주고 받을 수 있게 해줄 뿐 그 이상의 일 x

- 주고 받은 문자열의 해독은 온전히 어플리케이션에 맡긴다.

- HTTP는 형식을 정해두었기 때문에 약속에 따른 형식만 따르면 해석할 수 있지만, 웹소켓은 형식이 정해져 있지 않기 때문에 어플리케이션에서 쉽게 해석하기 힘들다.



## 참고

- [코일의 web Socket](https://www.youtube.com/watch?v=MPQHvwPxDUw&list=WL&index=3)

