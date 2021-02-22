---
title: "[C++][알고리즘][BOJ] 2309_일곱 난쟁이 : 부분집합 또는 순열을 이용한 방식"
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





# boj_2309_일곱 난쟁이

- 처음 문제 딱 봤을 때는 어떻게 해야하나 도저히 접근이 안되다가 이틀 뒤에 다시 풀라고 보니까 수업 때 배운 부분집합 찾는 방법을 사용하면 될 것 같아서 이용해봄
- 다른 사람 코드 몇 개를 살펴보니 순열을 이용한 방법이 있었다. c++에 내장된 함수인 `next_permutation`을 이용한 것이다.



## 코드1 : 비트연산을 이용해서 부분집합을 구하기

```c++
#include <bits/stdc++.h>

using namespace std;

int main()
{
    vector<int> arr;
    int num;
    
    // vector에 입력받기
    for (int i=0; i<9; i++)
    {
        cin >> num;
        arr.push_back(num);
    }

    // 부분 집합을 이용.
    // 원소 갯수가 7개고 합이 100인 부분 집합 찾기
    for (int i=0; i < (1 << 9); i++)
    {
        int cnt=0;
        vector<int> temp;
        for (int j=0; j<9; j++)
        {
            if (i & (1 << j))
            {
                cnt++;
                temp.push_back(arr[j]);
            }
        }
        // 오름차순 정렬된 상태로 출력하라고 했으니까 일단 정렬
        sort(temp.begin(), temp.end());
        int sum = accumulate(temp.begin(), temp.end(), 0);
        if (cnt==7 && sum == 100)
        {
            for (auto& num : temp ) {
                cout << num << '\n';
            }
            break;
        }
        
    }
    
    return 0;
}
```



## 코드2 : 순열을 이용

```c++
#include <bits/stdc++.h>

using namespace std;

int a[9];
int main() 
{
	for (int i = 0; i < 9; i++) cin >> arr[i];
	sort(a, a + 9);
	while (next_permutation(a, a + 9)) 
    {
		int sum = 0;
		for (int i = 0; i < 7; i++) sum += a[i];
		if (sum == 100) 
        {
			for (int i = 0; i < 7; i++) cout << a[i] << '\n';
			break;
		}
	}
}
```



### 순열

#### 순열이란?

- 수학적으로 순열이란 서로 다른 n개의 원소에서  r개를 뽑아 한 줄로 세우는 경우의 수
- 순열에서는 순서가 다르면 원소의 조합이 같더라도 다른 경우의 수로 본다.

#### next_permutation

- C++의 `algorithm` 헤더에 정의

- n개의 원소의 순열을 구할 수 있음

- 기본적인 문법

  ```c++
  bool next_permutation(BidirectionalIterator first, BidirectionalIterator last);
  ```

  - 순열을 구할 컨테이너(보통 배열)의 시작과 끝 iterator를 인자로 받음
  - 해당 컨테이너에 다음 순열이 존재하면 그 컨테이너의 원소를 해당 순열 순서로 바꾸고 true를 반환, 다음 순열이 없는 마지막이라면 false를 반환

- 주의해야할 점

  - 오름차순으로 정렬된 값을 가진 컨테니어로만 사용 가능
  - 중복이 있는 원소들은 중복을 제외하고 순열을 만들어줌
    - 예를 들어 {1, 1, 2}와 같은 배열이 있을 때 6개를 만들어주는게 아니라 {1 ,1, 2}, {1, 2, 1}, {2, 1, 1} 세 개를 만들어줌.

