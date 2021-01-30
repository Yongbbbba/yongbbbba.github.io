---
title: "[C++][코드없는프로그래밍] 레퍼런스 : L value, R value 챕터 내용 정리"
excerpt: ""

categories:
  - TIL
tags:
  - C++
 
last_modified_at: 2021-01-30T16:00:00
toc: true
toc_sticky: true
---





[레퍼런스와 포인터의 차이를 설명했던 앞선 글](https://yongbbbba.github.io/til/reference/)에서 추가되는 내용이다.



# function에 argument를 넘겨주는 방식

## pass by (value, pointer, reference)

```cpp
void fooV(int a) // pass by value
{
    int b = a + 1;
}

void fooP(int* ap) // pass by pointer
{
    int b = *ap + 1;
}

void fooR(int& a) //pass by reference
{
    int b = a+1;
}
```



컴파일 후 어셈브릴 코드를 살펴보면 pass by pointer와 reference는 완벽히 동일하게 동작한다. 둘의 차이점에 대해서는 [앞선 글](https://yongbbbba.github.io/til/reference/#%EB%A0%88%ED%8D%BC%EB%9F%B0%EC%8A%A4%EC%99%80-%ED%8F%AC%EC%9D%B8%ED%84%B0%EC%9D%98-%EC%B0%A8%EC%9D%B4)에서 설명하였다.

추가로 pointer가 엄밀하게 필요한 경우가 아니면 pass by reference를 사용하는 것이 좋다. 그 이유는 포인터를 그대로 노출하게 되면 버그를 일으킬 확률이 높기 때문에 더 안전하게 코드를 자끼 위해서 참조를 사용하는 것이 좋다. 포인터를 없애주면 이상한 개발자가(당연히 내가 될 수도 있다_) 자기 멋대로 함수안에서 ptr=nullptr; 할 가능조차 없애준다. 특히 void fooR(const int& a)와 같이 const를 붙여주면 더욱 좋다. a를 재정의하게 ㅗ디면 컴파일 오류가 나기 때문에 더 안전한 코드를 짤 수 있는 것이다.



# L Value & R Value 레퍼런스 알아보기

## Lvalue와 Rvalue의 정의

먼저 둘을 정의를 살펴보자.

```
Lvalue : 객체를 참조하는 표현식이다. 메모리 위치를 가지고 있다.
Rvalue : C++ 표준에거 Rvalue를 정의할 때 제외 개념으로서 정의하고 있다. Rvalue는 Lvalue를 제외한 모든 것이다. 다시 말해 구분가능한 메모리 영억을 가지는 객체로 나타낼 필요가 없는, 임시로 존재하는 표현식이다.
```



말만 들으면 이해하기 쉽지가 않다. 쉽게 코드로 이해하자면 `int a=0`에서 =을 기준으로 왼쪽은 Lvalue, 오른쪽은 Rvalue이다. 한 번 부르고 다시 부를 수 있다면 Lvalue, 한 번 부르고 다시 부르지 않는 temporary한 value를 Rvalue라고 한다. 

또하나의 예로 `std::stirng s = "abc";`에서 s는 Lvalue, "abc"는 Rvalue인 것이다.

일반적으로 pass by value보다 reference가 copy가 더 적게 일어나기 때문에 더 좋다.

![img](/assets/post_images/2021-01-30-LvalueRvalue/image.png)



## std::move(무브 키워드)

무브 키워드는 resource ownership을 다른 object에 넘겨주게 된다.

```cpp
#include <iostream>
#include <string>

int main(){
    std::string a = "nocodeprogram";
    std::cout << "a: " << a << std::endl;
    
    std::string b = std::move(a);
    sotd::cout << "b: " << b << std::endl;
    std::cout << "a: " << a << std::endl;
    
    return 0 ;
}
```

```
a: nocodeprogram
b: nocodeprogram
a: 
```



위의 코드의 결과이다. 마지막 줄을 출력할 때 a의 ownership이 std::move를 통해 b로 넘어갔기 때문에 a는 더이상 참조하고 있는 것이 없다.

![img](/assets/post_images/2021-01-30-LvalueRvalue/image.png)



copy를 최대한 적게 하기 위해서는 위와 같이 코드 최적화를 해야한다. (아직은 좀 어려운 개념이라 이해가 잘 안된다 추후에 더 공부해봐야겠다.. )



# 참고 자료

- [코드없는 프로그래밍](https://nocodeprogram.com/)