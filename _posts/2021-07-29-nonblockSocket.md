---
title: "[C++] 초간단 TCP 논블록 소켓을 이용한 에코 서버 by winsock2"
excerpt: ""

categories:
  - TIL
tags:
  - C++
 
last_modified_at: 2021-07-29T11:00:00
toc: true
toc_sticky: true
---



# winsock2를 이용해서 만들어본 논블록 에코 서버 예제

winsock2를 통해 소켓을 생성하면 기본적으로 blocking socket이다. 아래와 같이 옵션을 줘서 non-blocking socket으로 바꿀 수 있다. 

기존 blocking socket에서 만약 send를 하다가 block이 되는 상황이 발생하면 return이 될 때까지 다음 명령을 실행할 수 없다. 만약 서버가 핸들링하고 있는 클라이언트가 여럿일 때 이렇게 block이 빈번하게 일어나면 서비스의 질이 저하될 것이다. 각 클라이언트당 스레드 하나를 할당해서 처리하는 방법도 있지만 이렇게 되면 컨텍스트 스위칭 비용이 발생하기 때문에 또한 비효율적일 수 있다. 그래서 아래와 같이 non-blocking 방법을 써서 위의 문제를 일부 해소할 수 있다. 

하지만 아래의 경우처럼 원래의 경우 block이 발생했어야하는 상황에서 return이 될 경우 무한루프를 돌면서 계속 정상적으로 실행이 되는지 체크하는 방식은 마치 spinlock의 경우처럼 cpu 사이클을 낭비하게 된다. 이를 해결하기 위해서 몇 가지 방법이 있다. 이는 추후에 올려봐야겠다.

## server.cpp

```c++
#include "pch.h"
#include <iostream>
#include <Windows.h>

#include <WinSock2.h>
#include <mswsock.h>
#include <WS2tcpip.h>
#pragma comment(lib, "ws2_32.lib")

int main()
{
	WSAData wsaData;
	if (::WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)
		return 0;

	// 블로킹(Blocking)
	//	   accept -> 접속한 클라가 있을 때
	// 	   connect -> 서버 접속 성공했을 때 
	// 	   send, sedto -> 요청한 데이터를 송신 버퍼에 복사했을 때
	// 	   recv, recvfrom  -> 수신 버퍼에 도착한 데이터가 있고, 이를 유저레벨 버퍼에 복사했을 때
	// 
	// 논블로킹(Non-Blocking)

	SOCKET listenSocket = ::socket(AF_INET, SOCK_STREAM, 0);
	if (listenSocket == INVALID_SOCKET)
		return 0;

	u_long on = 1;
	if (::ioctlsocket(listenSocket, FIONBIO, &on) == INVALID_SOCKET)  // 논블로킹소켓 옵션 설정
		return 0;

	SOCKADDR_IN serverAddr;
	::memset(&serverAddr, 0, sizeof(serverAddr));
	serverAddr.sin_family = AF_INET;
	serverAddr.sin_addr.s_addr = ::htonl(INADDR_ANY);
	serverAddr.sin_port = ::htons(7777);

	if (::bind(listenSocket, (SOCKADDR*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR)
		return 0;

	if (::listen(listenSocket, SOMAXCONN) == SOCKET_ERROR)  // maximum queue length
		return 0;

	cout << "Listen" << endl;
	 
	SOCKADDR_IN clientAddr;
	int32 addrLen = sizeof(clientAddr);


	// 논블록 소켓에서 아래와 같은 식으로 무한루프를 돌면서 무한루프를 돌면서 WSAEWOURLDBLOCK인 경우 체크를 하는 것은 CPU를 낭비하는 비효율적인 상황이다. 이를 해결할 수 있는 여러 옵션이 존재.

	// Accept
	while (true)
	{
		SOCKET clientSocket = ::accept(listenSocket, (SOCKADDR*)&clientAddr, &addrLen);
		if (clientSocket == INVALID_SOCKET)  
		// 앞선 예제에서 블록킹 소켓에서는 여기서 그냥 프로그램을 종료시키는 방법을 사용했지만 
		// 논블록킹 소켓에서는 INVALID_SOCKET이 떴다고 해서 반드시 문제가 되는 것은 아님
		{
			// 원래 블록했어야 했는데... 너가 논블로킹으로 하라며?
			if (::WSAGetLastError() == WSAEWOULDBLOCK)
				continue;

			// Error
			break;
		}
		cout << "Client Connected!" << endl;

		// Recv
		while (true)
		{
			char recvBuffer[1000];
			int32 recvLen = ::recv(clientSocket, recvBuffer, sizeof(recvBuffer), 0);
			if (recvLen == SOCKET_ERROR)
			{
				// 원래 블록했어야 했는데.. 너가 논블로킹으로 하라며? 
				if (::WSAGetLastError() == WSAEWOULDBLOCK)
					continue;

				// Error
				break;
			}
			else if (recvLen == 0)
			{
				// 연결 끊김
				break;
			}

			cout << "Recv Data Len = " << recvLen << endl;

			// 에코 서버
			// Send 
			while (true)
			{
				if (::send(clientSocket, recvBuffer, recvLen, 0) == SOCKET_ERROR)
				{
					// 원래 블록했어야 했는데.. 너가 논블로킹으로 하라며? 
					if (::WSAGetLastError() == WSAEWOULDBLOCK)
						continue;

					// ERROR
					break;
				}
				cout << "Send Data ! Len = " << recvLen << endl;
				break;
			}
		}
		
	}

	// 윈속 종료
	::WSACleanup();
}

```



## client.cpp

```c++
#include "pch.h"
#include <iostream>

#include <winsock2.h>
#include <mswsock.h>
#include <ws2tcpip.h>
#pragma comment(lib, "ws2_32.lib")

int main()
{
	// winsock 초기화 (ws2_32 라이브러리 초기화)
	// 관련 정보가 wsaData에 채워짐
	WSADATA wsaData;
	if (::WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) // 초기화 실패
		return 0;

	SOCKET clientSocket = ::socket(AF_INET, SOCK_STREAM, 0);
	
	if (clientSocket == INVALID_SOCKET)
		return 0;

	u_long on = 1;
	if (::ioctlsocket(clientSocket, FIONBIO, &on) == INVALID_SOCKET)
		return 0;

	SOCKADDR_IN serverAddr;
	::memset(&serverAddr, 0, sizeof(serverAddr));
	serverAddr.sin_family = AF_INET;
	::inet_pton(AF_INET, "127.0.0.1", &serverAddr.sin_addr);  // 사람이 알아보기 쉬운 텍스트(human-readable text)형태의 IPv4 와 IPv6 주소를 binary 형태로 변환 하는 기능
	serverAddr.sin_port = ::htons(7777);

	// Connect
	while (true)
	{
		if (::connect(clientSocket, (SOCKADDR*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR)
		{

			// 원래 블록했어야 했는데... 너가 논블로킹으로 하라며?
			if (::WSAGetLastError() == WSAEWOULDBLOCK)
				continue;

			// 이미 연결된 상태라면 break
			if (::WSAGetLastError() == WSAEISCONN)
				break;

			// Error
			break;
		}
	}

	cout << "Connectd To Server! " << endl;

	char sendBuffer[100] = "Hello World";

	// Send
	while (true)
	{
		if (::send(clientSocket, sendBuffer, sizeof(sendBuffer), 0) == SOCKET_ERROR)
		{
			// 원래 블록했어야 했는데.. 너가 논블로킹으로 하라며? 
			if (::WSAGetLastError() == WSAEWOULDBLOCK)
				continue;

			// ERROR
			break;
		}
		cout << "Send Data ! Len = " << sizeof(sendBuffer) << endl;

		// Recv
		while (true)
		{
			char recvBuffer[1000];
			int32 recvLen = ::recv(clientSocket, recvBuffer, sizeof(recvBuffer), 0);
			if (recvLen == SOCKET_ERROR)
			{
				// 원래 블록했어야 했는데.. 너가 논블로킹으로 하라며? 
				if (::WSAGetLastError() == WSAEWOULDBLOCK)
					continue;

				// Error
				break;
			}
			else if (recvLen == 0)
			{
				// 연결 끊김
				break;
			}
			cout << "Recv Data Len = " << recvLen << endl;
			break;
		}
		
		this_thread::sleep_for(1s);
	}

	// 소켓 리소스 반환
	::closesocket(clientSocket);

	// 윈속 종료
	::WSACleanup();
}


```

