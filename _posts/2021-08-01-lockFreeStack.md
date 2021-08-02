---
title: "[C++][TIL] Lock-Free Stack 정리"
excerpt: ""

categories:
  - TIL
tags:
  - C++
 
last_modified_at: 2021-08-02T17:00:00
toc: true
toc_sticky: true
---



# Lock Free

## 개요

멀티쓰레드 환경에서 lock을 사용해서 공유자원에 대한 race condition문제, 동기화 문제 등을 해결할 수 있다. 이 때문에 싱글쓰레드보다 멀티쓰레드가 항상 우월할 것이라고 생각하기 쉽다. 하지만 무차별적인 lock은 기아현상을 유발할 수 있다. 실제 중요한 쓰레드가 상대적으로 별로 중요하지 않은 쓰레드에 밀려서 자원획들을 못하고 있을 수 있는 것이다. 

성능이슈 또한 존재한다. CRITICAL_SECTION을 사용하는 Mutex 역시 시스템 자원이다. lock을 거는 것 하나하나가 커널에 자원 요청을 하는 것이고 lock을 거는 것 자체가 오버헤드가 된다. 게다가 공유자원을 하나의 쓰레드가 얻게 되면 lock을 얻고 싶어하는 다른 쓰레드는 blocking 상태에 빠진다. 이런 공유자원이 자주 사용되는 자원이 아니라 빈번하게 사용되는 자원이라면 수없이 lock-unlock이 반복되고 blocking되는 쓰레드가 넘쳐날 것이다. 이러면 차라리 싱글쓰레드 서버가 멀티쓰레드 보다 빠른 상황이 연출된다. 멀티쓰레드 프로그래밍이 더 힘든데도 불구하고 말이다.

이러한 문제를 해결하는 방법을 통칭하여 Non-Blocking(공유 자원 관리) 알고리즘이라 한다. 



## Non-Blocking 알고리즘

Non-Blocking 알고리즘에도 등급이 있다. 가장 이상적인 것은 전체 쓰레드가 공유 자원에 접근하면서도 대기하지 않고 그냥 진행이 되는 것이다. 이것을 Wait-free 수준의 알고리즘이라고 한다. 하지만 이를 제대로 구현하려면 많은 제약이 따른다. 일반적인 경우 Lock-Free 수준의 알고리즘을 사용한다. 이후 내용도 Lock-Free 알고리즘에 대해 주로 설명할 것이다. 이에 앞서서 Lock-Based 알고리즘에 대해 먼저 살펴볼 것이다.



## Lock Based stack/queue

producer-consumer 모델처럼 한쪽은 큐나 스택에 데이터를 넣고, 한쪽은 데이터를 꺼내는 상황을 무한반복할 때  lock을 걸어주지 않으면 언젠가 크래시가 발생하게 된다. 아래처럼 임의의 상황을 가정하여보자.

```c++
queue<int> q;

void Push()
{
    while (true)
    {
        int value = rand() % 100;
        q.push(value);
        this_thread::sleep_for::(10ms); // 데이터 생산 속도와 소비 속도에 차이가 발생함
    }
}

void Pop()
{
    while (true)
    {
        if (q.empty())
            continue;
        int data = q.front();
        q.pop();
        cout << data << endl;
    }
}

int main()
{
    thread t1(Push);
    thread t2(Pop);
    
    t1.join();
    t2.join();
}
```

이러한 문제를 해결하기 위한 방법은 다양한데 대표적으로 mutex를 사용하는 방법이 있다. 또한 queue나 stack은 자주 사용하는 자료구조이기 때문에 해당 자료구조 안에 lock을 끼워넣는 방법을 생각해볼 수도 있다. 특히 네트워크 프로그래밍에서 패킷을 처리하는 과정에서 먼저 온 패킷을 먼저 처리해주는 상황이 빈번한게 일어나기 때문에 이럴 때 lock based queue가 유용하게 활용할 수 있다. 



### Lock-Based-Stack

```c++
// ConcurrentStack.h
#pragma once

#pragma once
#include <mutex>

template<typename T>
class LockStack
{
public:
	LockStack() {}
	LockStack(const LockStack&) = delete;  // 복사생성자 시도하면 막아버리기
	LockStack& operator=(const LockStack&) = delete;

	void Push(T value)
	{
		lock_guard<mutex> lock(_mutex);
		_stack.push(std::move(value));  // 더 빠른 연산을 위해서. move를 쓴다는거는 value를 rvalue로 만드는 것.
		_condVar.notify_one();  // 데이터 들어오면 스레드 하나 꺠운다.
	}

	bool TryPop(T& value)
	{
		// 기존의 방법에서는 stack이 비어있는지 확인한 후에 pop을 해주는데, 멀티스레드 환경에서는 empty를 확인 후에 pop을 하는 것이 의미가 없다. 진짜 empty인지 정확하지 않기 때문에 이를 묶어서 처리한다.
		lock_guard<mutex> lock(_mutex);
		if (_stack.empty())
			return false;
		// empty -> top -> pop
		value = std::move(_stack.top());
		_stack.pop();
		return true;
	}

	void WaitPop(T& value)
	{
		unique_lock<mutex> lock(_mutex);
		_condVar.wait(lock, [this] {return _stack.empty() == false;});  // 데이터가 들어와서 notify_one을 호출하면 조건 체크해서 스택에 데이터가 들어온 것을 확인한 후에 일어나서 lock을 걸고 작업한다.
		value = std::move(_stack.top());
		_stack.pop();
	}

private:
	stack<T> _stack; 
	mutex _mutex;
	condition_variable _condVar;
};
```



## Lock free stack

개요에서 이야기한 것처럼 lock을 사용하게 되면 오버헤드가 있기 때문에 lock을 덜 쓰면 속도가 더 빨라지지 않을까라는 생각이 든다. (하지만 동기화를 해주는 부분에서 고려해야하는 많은 조건이 있기 때문에 구현이 까다롭다. 그리고 속도를 측정해보면 생각보다 빠르지 않은 경우가 있다). Lock-Free는 지금도 활발하게 연구되는 분야로,  Lock-Free를 구현하는 다양한 방법들이 존재한다.. Lock-Free를 구현하는 알고리즘에 대한 논문도 많이 나오고 특허로 등록되는 경우도 있다.

Lock-Free는 주로 Atomic 연산을 활용해서 구현할 수 있다. atomic 연산은 실행이 한 번에 완료되는 것을 보장하므로 쓰레드 동기화 문제를 해결할 수 있다. 

여러 aotmic 연산이 있는데 그중에서 가장 쓰기 좋은 것이 CAS(Compare-and-swap, windows에서 비슷한 함수는 InterlockedCompareExchange)이다. CAS는 쉽게 말해서 병렬 연산에서 발생할 수 있는 문제를 하드웨어적으로 해결하는 방법 중 하나라고 생각하면 된다. 

CAS 연산에는 3개의 인자를 넘겨주게 된다. 작업할 대상 메모리의 위치(V), 예상하는 기존 값(expected), 새로 설정할 값(desired). CAS연산은 V위치에 있는 값이 expected와 같은 경우에 desired로 변경하는 단일 연산이다. 만약 이전 값이 expected와 다르다면 아무런 동작도 하지 않는다. 그리고 값을 desired로 변경했건 못했건 간에 어떤 경우라도 현재 V의 값을 리턴한다. 

만약 여러 스레드가 동시에 CAS 연산을 사용해 한 변수의 값을 변경하려고 한다면, 스레드 가운데 하나만 성공적으로 값을 변경할 것이고, 다른 스레드들은 모두 실패한다. 대신 값을 변경하지 못했다고 해서 lock을 확보하는 것처럼 대기 상태에 들어가는 대신, 값을 변경하지 못했지만 다시 시도할 수 있다고 알림을 받는 것과 같아진다. 다시 말해  CAS연산에 실패한 스레드도 대기상태에 들어가지 않기 때문에 스레드마다 CAS연산을 다시 시도할 것인지, 아니면 다른 방법을 취할 것인지, 아니면 아예 아무 조치도 취하지 않을 것인지를 프로그래머가 결정할 수 있다. 



### ABA 문제 

하지만 CAS를 사용할 때 주의해야할 부분이 있는데 바로 ABA 문제이다. 

ABA문제란 CAS를 사용해서 자료구조의 값을 변경할 때, 포인터가 시스템에 의해 재사용되면서 생기는 문제이다.

아래와 같이 pop을 하는 상황을 가정해보자.

```c++
// ConcurrentStack.h
#pramga once

template<typename T>
class LockFreeStack
{
    struct Node
    {
        Node(const T& value) : data(value), next(nullptr)
        {
            
        }
        
        T data;
        Node* next;
    }
public:
    // 
    // ...
    //
    
    bool TryPop()
    {
        Node* oldHead = _head;
        if (oldHead == nulltpr)
            return false;
        if (_head.compare_exchange_weak(oldHead, oldHead->next))
            return true;
    }
private:
    atomic<Node*> _head;
};
```

언뜻 보면 아무런 문제가 없어보인다. 만약 스레드1에서 CAS연산을 시작하기 직전에 스레드2에서 pop과 push를 한 번씩 했다고 하자. 그러면 oldHead와 스레드2가 push까지한 상태에서의 head와 값이 다르기 때문에 true를 반환하면 안될 것으로 예상이 된다. 그런데 시스템에 의해서 oldHead의 주소를 재활용해서 push를 해버리게 되면 이 주소값이 동일하기 때문에 CAS이 true를 반환하게 되는 괴이한 일이 벌어지게 되는데 이러한 상황을 ABA 문제라고 한다.

ABA 문제를 해결하기위해서 많은 연구가 진행되고 있는데 대표적으로 DCAS(Double compare-and-swap)이 있다. 위의 예에서는 atomic으로 선언한 popCount를 두어서 현재 pop을 시도중인 다른 스레드들이 있는지 한 번더 체크하는 식으로해서 Pop을 진행하는 방식이 있다. (-> 해당 [블로그 링크](https://chfhrqnfrhc.tistory.com/entry/ABA-Problem-1?category=509454) 참고)

### 

- 출처
  - https://ozt88.tistory.com/38
  - https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-4/
  - https://effectivesquid.tistory.com/entry/Lock-Free-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98Non-Blocking-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98
  - https://chfhrqnfrhc.tistory.com/entry/ABA-Problem-1?category=509454

