---
title: "[C++] 레퍼런스와 포인터의 차이점"
excerpt: ""

categories:
  - TIL
tags:
  - C++
 
last_modified_at: 2021-01-29T23:00:00
toc: true
toc_sticky: true
---

# 레퍼런스(참조자)와 포인터는 같은 것일까? 



## 참조자의 기능

```c++
#include <iostream>

int main() {
    int a = 3;
    int& another_a = 3;
    
    another_a = 5;
    std::cout << "a : " << a << std::endl;
    std::cout << "another_a : " << another_a << std::endl;
    
    return 0;
}
```



위 코드를 컴파일해서 실행시키면 결과는 아래와 같다.

```
a : 5
another_a : 5
```

이는 포인터를 사용했을 때와 동일한 출력 결과다.



```cpp
#include <iostream>

int main() {
    int a = 3;
    int* another_a = &a;
    
    *another_a = 5;
    std::cout << "a : " << a << std::endl;
    std::cout << "another_a : " << *another_a << std::endl;
    
    return 0;
}
```

```
a : 5
another_a : 5
```



그렇다면 포인터와 레퍼런스는 동일한 역할을 하고 있는 것일까? 



## 레퍼런스와 포인터의 차이

### 1. 레퍼런스는 반드시 어떤 것을 참조하는지 지정해야함

```cpp
int& another_a; // 이런 코드는 불가능하다.
int* another_a; // 반면 이런 코드는 가능하다.
```



### 2. 레퍼런스는 한 번 참조하면 다른 것을 참조할 수 없다.

```cpp
int a = 10;
int& another_a = a; // a를 참조함

int b = 3;
another_a = b; // 이는 불가

int* p = &a; // p는 a의 주소를 가리킴
p = &b; // p는 a를 버리고 b의 주소를 가리킬 수 있음
```



### 3. 레퍼런스는 메모리 상에 존재하지 않을 수도 있다

```cpp
int a = 10;
int* p = &a; // 64비트 운영체제의 경우 p는 8바이트 차지

int & another_a = a; // another_a는 메모리를 차지하지 않음. 새로 변수를 생성한 것이 아님.
```

하지만 반드시 그런 것은 아니다. 이는 추후 [R-value, L-value에 대한 포스팅](https://yongbbbba.github.io/til/LvalueRvalue/)을 할 때 다시 한 번 언급할 것이다.



# 참고자료

- [모두의 코드 🙏](https://modoocode.com/)

