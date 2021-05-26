---
title: "[네트워크] HTTP 헤더의 Connection : Keep-Alive의 의미"
excerpt: ""

categories:
  - TIL
tags:
  - CS
  - 네트워크
  - HTTP
 
last_modified_at: 2021-05-27T02:10:00
toc: true
toc_sticky: true
---

# HTTP 헤더의 Connection : Keep-Alive의 의미

이번 프로젝트를 진행하면서 HTTP 요청 부분을 살펴보는데 늘 대충 보고 지나치던 헤더의 **Connection : Keep-Alive** 부분에서 왠지 모를 이질감을 확 느꼈다.

HTTP는 stateless/connectless 하다고 귀에 딱지가 앉도록 들었던 것 같은데 연결을 유지한다는게 뭘까 궁금해져서 찾아봤다.

## HTTP(HyperText Transfer Protocol)란?

간단하게 말해서 웹 브라우저(클라이언트)가 서버가 대화(요청과 응답)를 해야하는데 그런 대화 방법을 정한 규약이다.

## HTTP의  stateless/connectless 

기본적으로 HTTP 방식에서는, 클라이언트에서 서버에 무언가 요청을 하면 바로 연결을 끊는다. 또한 연결을 끊으면서 이전의 상태 정보를 남겨두지 않는다. 이것이 HTTP의 stateless/connectless 한 특징이다. 

### stateless/connectless의 문제점과 Kepp-Alive의 필요성

HTTP는 하나의 요청에 하나의 응답을 하게 되어있는데, 여기서 약간의 문제가 발생한다. 요즘 웹 서비스에는 하나의 간단한 웹페이지도에도 이미지, 문서, css, javascript 파일 등 각종 데이터가 굉장히 많이 담겨있다. 웹페이지 하나 보려고 하면 이미지 하나 요청하고 연결 끊고, html 문서 하나 요쳥하고 연결 끊는 식으로 작업을 반복해야한다. 연결을 맺고 끊는 것은 TCP 통신 과정에서 꽤 비용을 수반하는 일이기 때문에 비효율적이다. 

그래서 HTTP 1.1부터는 일정 시간 연결을 유지하는 Keep-alive를 도입하게 되었다. Keep-Alive 설정을 하면, 지정된 시간동안 연결을 끊지 않고 요청을 계속해서 보낼 수 있다. 무한정으로 연결되는 것은 아니고, `Kepp-alive:timeout=5, max=100`이런 식으로 연결 유지 시간과, 최대 연결 개수를 정할 수 있다. 또한 요청할 떄 keep-alive를 담아 요청한다고 해서 반드시 그렇게 되는 것이 아니고 서버에서 그렇게 응답을 해줘야한다. 연결 좀 유지해주실 수 없나요??라고 클라이언트에서 부탁 한 번 해보는거다.

