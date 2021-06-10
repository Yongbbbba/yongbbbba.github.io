---
title: "[C#] C#으로 만드는 초간단 단방향 채팅 서버 예제"
excerpt: ""

categories:
  - TIL
tags:
  - C#
  - Socket
 
last_modified_at: 2021-06-10T20:45:00
toc: true
toc_sticky: true
---



# C#으로 만드는 초간단 단방향 채팅 서버 예제

클라이언트에서 소켓을 통해 서버로 메세지를 보내면 서버에서 그대로 메시지를 출력해주는 단방향 채팅 서버

1. 소켓과 엔드포인트를 만들고
2. 소켓에 포트를 바인딩하고
3. 클라이언트에서 연결 요청을 하면 연결을 한 후에
4. 연결이 정상적으로 됐다면 클라이언트로부터 오는 메시지를 콘솔에 출력



## serverSocket

```c#
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;

namespace ServerSocket
{
    class Program
    {
        static void Main(string[] args)
        {
            // stream - TCP, datagram - UDP
            Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
            IPEndPoint ep = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 9999);  // 엔드포인트 만들어주고
            socket.Bind(ep);  // 소켓에 포트 바인딩해주기

            socket.Listen(10);  // 최대 10개의 연결을 받을 수 있게 지정

            // 클라이언트 소켓의 메시지를 서버 소켓에서 받아서 출력해줄 것임
            Socket clientSocket = socket.Accept();  // 클라이언트 쪽에서 서버 쪽의 연결 요청을 받으면 클라이언트 소켓을 만들어주기
            if (clientSocket.Connected)
                Console.WriteLine("클라이언트가 서버에 접속하였습니다.");

            // 지속적으로 클라이언트로부터 데이터를 받기 위하여 while문
            while (!Console.KeyAvailable)  // 콘솔에 키가 눌렸을 때 종료 
            {
                byte[] buff = new byte[2048];
                int n = clientSocket.Receive(buff);  // 클라이언트로부터 전송된 데이터를 받는다. 수신 버퍼에. 버퍼의 길이를 확인하기 위하여 n에 저장. 반환값이 바이트의 수임
                string result = Encoding.UTF8.GetString(buff, 0, n);  // 0번 인덱스부터 수신받은 데이터의 길이만큼 string을 출력해서 result에 넣어줌

                Console.WriteLine(result);
            }
        }
    }
}

```



## clientSocket

```c#
using System;
using System.Net.Sockets;
using System.Text;

namespace ClientSocket
{
    class Program
    {
        static void Main(string[] args)
        {
            Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
            socket.Connect("127.0.0.1", 9999);   // 서버와 연결

            if (socket.Connected)
            {
                Console.WriteLine("서버에 연결 되었습니다.");
            }

            string message = string.Empty;
            
            while ((message = Console.ReadLine()) != "x")
            {
                byte[] buff = Encoding.UTF8.GetBytes(message);  // 입력으로 받은 메시지를 UTF8으로 인코딩(바이트값으로)
                socket.Send(buff);  // 송신버퍼를 소켓에 보내게 된다. 그런 다음 소켓에서 서버로 보냄.

            }
        }
    }
}

```







## 출처

[세상의 모든코딩](https://www.youtube.com/watch?v=ki3fb995l2E)