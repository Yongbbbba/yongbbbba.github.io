---
title: "[C++][알고리즘][BOJ] 10093_숫자 : 꼭 주의해야하는 integer overflow "
excerpt: ""

categories:
  - TIL
tags:
  - C++
  - 알고리즘
 
last_modified_at: 2021-02-22T00:20:00
toc: true
toc_sticky: true
---





# boj_10093_숫자



처음에 딱 보고 너무 쉬운 문제라고 생각했다. 그런데 정답율을 보니까 23%여서 의아했다.

왜 그럴까 생각을 해봤는데 두 숫자가 같을 때 또는 두 수의 크고 작음을 고려하지 않아서 많이들 실수했겠구나 싶었다. 뿌듯함을 느끼고 제출을 했는데 받아들인 결과는 출력초과였다.

```c++
#include <bits/stdc++.h>

using namespace std;

int main()
{
    int N, M;
    cin >> N >> M;
    int cnt = N == M ? 0 : abs(N-M) -1; // 조건에 두 수가 같은 수가 아니라고는 말 안했으니까 이에 대해 일종의 방어코드 작성
    cout << cnt << '\n';
    
    if (cnt != 0)
    {
        if (N < M)
        {
            for (int i=N+1; i<M; i++)
            {
                cout << i << ' ';
            }
        }
        else 
        {
            for (int i=M+1; i < N; i++)
            {
                cout << i << ' ';
            }
        }
    }
    return 0;
}
```



뒤에 공백을 더 붙여 출력해서 그런걸까? 아냐 그럴리가 없는데 싶었다. 

질문글을 슬쩍 봤는데 long long을 써야한다는 어떤 글을 봤다. 그래서 같은 로직대로 python으로 짜서 제출해보니까 통과가 되었다. 

```python
N, M = map(int, input().split())

cnt = 0 if N == M else abs(N-M) -1
print(cnt)
if N != M:
    if N < M:
        for i in range(N+1, M):
            print(i, end=' ')
    else:
        for i in range(M+1, N):
            print(i, end=' ')
```



역시 자료형 크기의 문제였던 것이다. 

얼마 전 [강의](https://youtu.be/9MMKsrvRiw4)를 볼 때 초보들이 너무나도 자주 실수하는 경우가 `integer overflow`가 나는 경우라는 말을 들었는데 지금까지 그런 문제는 아직까지 접하지 못 해서 당분간은 마주하지 않겠지 싶었다. 그냥 그런가보다 하고 넘어갔던 것 같다. 대충 int가 21억 정도까지 표현할 수 있다는 것만 알고 있었다.



그래서 문제 조건을 다시 살펴봤는데 입력되는 수가 `10 ** 15`였다. 두 수의 차이가 100,000인거지 입력되는 수는 그정도의 범위였다. 그래서 in형을 범위를 다시 살펴보니 대략 `2.1 * 10**9` 였다. 다시 말해 int 자료형을 사용하면 integer overflow가 나는 것이다. long long 의 경우 `9.2 * 10**18`까지이다. 그러니까 long long을 사용했어야 하는 문제였다.



```c++
#include <bits/stdc++.h>

using namespace std;

int main()
{
    long long N, M;
    cin >> N >> M;
    long long cnt = N == M ? 0 : abs(N-M) -1;
    cout << cnt << '\n';
    
    if (cnt != 0)
    {
        if (N < M)
        {
            for (long long i=N+1; i<M; i++)
            {
                cout << i << ' ';
            }
        }
        else 
        {
            for (long long i=M+1; i < N; i++)
            {
                 cout << i << ' ';
            }
        }
    }

    
    return 0;
}
```



역시나 통과다. 강의에서 몇 번 실수를 해봐야 안 까먹는다고 했는데 나도 오늘 이거 겪어봤으니 당분간은 주의하지 않을까 ...