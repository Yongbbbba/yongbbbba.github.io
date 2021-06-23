---
title: "[C#] 초간단 Async, Await 공부"
excerpt: ""

categories:
  - TIL
tags:
  - CSharp
 
last_modified_at: 2021-06-16T11:00:00
toc: true
toc_sticky: true
---



# [C#] async/await

async라는 키워드를 보면 이 키워드가 바로 비동기 프로그래밍과 관련된 키워드구나라고 생각할 수 있다. 하지만 게임 서버를 공부할 때 비동기 프로그래밍에 관해 흔히 하는 착각은 이게 멀티스레드와 관련된 것이라고 생각하는 것이다. 하지만 자바스크립트를 공부할 때 비동기/싱글스레드라는 키워드를 접하진 않았는가? 그렇다면 비동기가 반드시 멀티스레드와 연관된 개념이 아니라고 유추해볼 수 있다. 유니티의 코루틴 또한 일종의 비동기인데 이것은 싱글스레드에서 돌아간다.



## 예제 코드

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace CSharp
{
 
    class Program
    {
        static Task Test()
        {
            Console.WriteLine("Start Test");
            Task t = Task.Delay(3000);
            return t;
        }

        
        static void Main(string[] args)
        {
            Task t = Test();

            Console.WriteLine("while start");

            while (true)
            {

            }
        }

    }
}

     
```

위 코드를 실행해보면 `Task t`에 딜레이를 3초 줬음에도 불구하고 `Start Test`가 먼저 출력되고 나서 딜레이 없이 곧바로 `while start`가 출력되게 된다.

만약 while문이 시작하기 전에 `t.Wait();` 를 넣어주면 task가 동작을 완료할 때까지 기다리게 된다. (block 시킨다고 생각하면 되는건지..?)

그런데 만약 그 딜레이 작업이 매우 오래 걸리는 작업이라면 그 작업의 완료를 기다리느라 CPU가 일을 안하고 놀게 되는 상황이 발생된다. 프로그램이 먹통이 되어버리는 상황과 같아진다.  보통 웹 서비스에서 DB에서 데이터를 가져올 때 이런 상황이 생길 수 있는데 이때 await을 하지 않게 되면 성능이 떨어지게 된다. 이럴 때 await이 필요하게 된다. 딜레이가 걸리면 잠깐 다른 작업을 하러 가는 것이다. 

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace CSharp
{
 
    class Program
    {
        static async void TestAsync()
        {
            Console.WriteLine("Start TestAsync");
            Task t = Task.Delay(3000);
            await t;
            Console.WriteLine("End TestAsync");
        }

        
        static void Main(string[] args)
        {
            TestAsync();


            Console.WriteLine("while start");

            while (true)
            {

            }
        }

    }
}
```

위 코드의 출력 결과는 `Start TestAsync` `while start`가 차례로 출력된 후 3초 후에 `End TestAsync`가 출력되게 된다.

