---
title: "[C++][자료구조] 야매 연결 리스트"
excerpt: ""

categories:
  - algorithm
tags:
  - C++
  - 알고리즘
 
last_modified_at: 2021-02-22T11:00:00
toc: true
toc_sticky: true
---



# 야매로 연결 리스트 구현하기

- 코딩 테스트에서 연결 리스트를 사용해서 문제를 풀어야하는 경우가 있다면, `STL list`를 사용하면 된다. 그런데 드물게 STL 사용을 제한하는 테스트라면 직접 구현해서 사용해야한다.
- 보통 연결 리스트를 구현할 때 아래와 같이 `struct`를 많이 사용한다. 

```c++
struct Node
{
    struct Node *prev, *next;
    int data;
};
```

- 하지만 시간이 생명인 코딩 테스트에서 구조체를 이용한 연결 리스트 구현은 시간을 많이 잡아먹을 수 있다. 그래서 소개하는 것이 야매 연결 리스트.. 

```c++
// 실무에서는 메모리 낭비, 누수가 너무 심하기 때문에 절대 사용할 일이 없을 것이다
// 대신 구현 난이도가 포인터를 사용하는 방식보다 낮고. 시간 복잡도도 동일하기 때문에 사용이 가능
const int MX = 10000005;
int dat[MX], pre[MX], nxt[MX];
int unused = 1;

fill(pre, pre+MX, -1);
fill(nxt, pre+MX, -1);
```

- 각 변수가 의미하는 것

  - 원소를 배열로 관리
  - pre, nxt에 이전/다음 원소의 포인터 대신 배열상의 인덱스를 저장
  - `dat[i]`는  i번지 원소의 값, `pre[i]`는 i번지 원소에 대해 이전 원소의 인덱스, `nxt[i]`는 다음 원소의 인덱스, pre나 nxt의 값이 -1이면 해당 원소의 이전/다음 원소가 들어있지 않다는 의미
  - unused는 현재 사용되지 않는 인덱스, 즉 새로운 원소가 들어갈 수 있는 인덱스. 원소가 추가된 이후에는 1씩 증가함. 
  - 특별히 0번지는 연결 리스트의 시작 원소로 고정되어 있다. 0번지는 값이 들어가지 않고 단지 시작점을 나타내기 위한 더미 노드. 더미 노드를 두는 이유는 나중에 삭제/삽입의 기능을 구현할 때 더미 노드를 두지 않으면 원소가 아예 없는 경우에 대한 예외처리를 따로 해줘야하는 불편함이 있기 때문이다.
  - 위 코드에서 연결 리스트의 길이도 관리하고 싶다면 `len` 변수를 따로 두고, 원소가 추가/삭제 될 때마다 해당 변수의 값을 변경시켜주면 된다.
  - 야매 연결 리스트 예시

  ![image-20210307103405566](/assets/post_images/2021-03-07-linkedlist.assets/image-20210307103405566.png)

```c++
// 처음에는 그림을 그려가면서 이해하는 것을 권장
#include <iostream>
using namespace std;

const int MX = 1000005;
int dat[MX], pre[MX], nxt[MX];
int unused = 1;

void insert(int addr, int num){
    dat[unused] = num; // 아직 사용하지 않은 공간에 num 할당
    pre[unused] = addr; 
    nxt[unused] = nxt[addr];nxt[unused] = nxt[addr]; // addr이 가리키고 있던 곳을 새로 할당한 공간이 가리킴
    if(nxt[addr] != -1 ) pre[nxt[addr]] = unused; // 만약 nxt[addr] == 1이라면 pre[-1]이기 때문에 잘못된 접근이 된다.
    nxt[addr] = unused;
    unused++;
}

void erase(int addr){
    nxt[pre[addr]] = nxt[addr];
    if (nxt[addr] != -1) pre[nxt[addr]] = pre[addr]; // 만약 nxt[addr] == 1이라면 pre[-1]이기 때문에 잘못된 접근이 된다.
}

// 탐색하기
void traverse(){
  int cur = nxt[0];
  while(cur != -1){
    cout << dat[cur] << ' ';
    cur = nxt[cur];
  }
  cout << "\n\n";
}

void insert_test(){
  cout << "****** insert_test *****\n";
  insert(0, 10); // 10(address=1)
  traverse();
  insert(0, 30); // 30(address=2) 10
  traverse();
  insert(2, 40); // 30 40(address=3) 10
  traverse();
  insert(1, 20); // 30 40 10 20(address=4)
  traverse();
  insert(4, 70); // 30 40 10 20 70(address=5)
  traverse();
}

void erase_test(){
  cout << "****** erase_test *****\n";
  erase(1); // 30 40 20 70
  traverse();
  erase(2); // 40 20 70
  traverse();
  erase(4); // 40 70
  traverse();
  erase(5); // 40
  traverse();
}

int main(void) {
  fill(pre, pre+MX, -1);
  fill(nxt, nxt+MX, -1);
  insert_test();
  erase_test();
}
```

- insert 함수
  1. 새로운 원소를 생성
  2. 새 원소의 pre값에 삽입할 위치의 주소를 대입
  3. 새 원소의 nxt 값에 삽입할 위치의 nxt 값을 대입
  4. 삽입할 위치의 nxt 값과 삽입할 위치의 다음 원소의 pre 값을 새 원소로 변경
  5. unused 1 증가
- erase 함수
  1. 이전 위치의 nxt를 삭제할 위치의 nxt로 변경
  2. 다음 위치의 pre를 삭제할 위치의 pre로 변경

 

## 출처

- [바킹독의 실전 알고리즘](https://youtu.be/C6MX5u7r72E)



