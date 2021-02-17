---
title: "[알고리즘] 정렬 : 버블 정렬과 선택 정렬"
excerpt: ""

categories:
  - algorithm
tags:
  - C++
  - 알고리즘
 
last_modified_at: 2021-02-17T08:00:00
toc: true
toc_sticky: true
---



# 버블 정렬

- 버블 정렬의 컨셉은 i번째 원소가 i+1번째 원소보다 크다면 둘의 자리를 바꿔주는 것이다.
- 버블 정렬은 가장 구현하기 쉬운 정렬이지만 효율이 아주 좋지 못하다. O(n^2)의 시간 복잡도를 가지고 있다. 따라서 n이 조금만 커져도 연산량이 아주 많이 늘어나기 때문에 n이 작고 빠른 구현이 필요할 때 사용하면 좋다.



## 버블 정렬 구현 코드

```c++
#include <bits/stdc++.h>
using namespace std;

int main() {
    int arr[10]  = {3,2,5,6,7,10,1,4,9,8};
    // 버블 정렬
    for (int i=0; i < 10; i++) {
        for (int j=0; j < 9-i; j++) {
            if (arr[j] > arr[j+1]) swap(arr[j], arr[j+1]);
        }
    }
    
     for (auto num : arr) {
        cout << num << ' ';
    }
    return 0;
}
```



# 선택 정렬

- 선택 정렬의 컨셉은 가장 작은 수를 앞으로 보내주는 것이다.
- 만약 원소의 개수가 n개 라면 n * (n+1) / 2의 연산을 하게 되는데 big-O 표기법으로는 O(n^2)의 시간 복잡도를 가진다.



## 선택 정렬 구현 코드

```c++
#include <bits/stdc++.h>
using namespace std;

int main() {
    int arr[10]  = {3,2,5,6,7,10,1,4,9,8};
    // 선택 정렬
    int idx;
    for (int i=0; i<10; i++) {
        int min = 11;
        for (int j=i; j<10; j++) {
            if (min > arr[j]) {
                min = arr[j];
                idx = j;
                }
        }
        swap(arr[i], arr[idx]);
    }
         for (auto num : arr) {
        cout << num << ' ';
    }
    return 0;
}
```



# 버블 정렬과 선택 정렬의 연산량

- 둘다 O(n^2)의 시간 복잡도를 가지고 있지만 실제로는 버블 정렬이 더 오래 걸린다. 
- 그 이유는 버블 정렬의 경우는 매번 자리를 바꿔주는 연산을 하지만, 선택 정렬은 작은 것을 쭉 찾아내서 나중에 맨 앞으로 보내주기 때문에 선택 정렬이 연산량이 더 적기 때문이다. 
- 결과적으로 여러가지 정렬 방법 중에 버블 정렬이 가장 연산이 오래 걸린다.