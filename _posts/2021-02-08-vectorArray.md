---
title: "[코드없는프로그래밍][C++] STL : vector와 array, deque 정리"
excerpt: ""

categories:
  - TIL
tags:
  - C++
  - algorithm
  - STL
 
last_modified_at: 2021-02-08T21:00:00
toc: true
toc_sticky: true
---



# vector intro



## vector의 특징

- Dynamic size array (동적으로 힙에 할당)
- Sequence container (순서를 가지는 컨테이너)

- 포인터를 통해 만드는 동적 배열은 delete를 일일이 해줘야하지만 vector는 알아서 메모리 해제된다.

## vector의 초기화 방법

```c++
#include <iostream>
#include <vector>

int main() {
    // 첫 번째 방법
    std::vector<int> vec1{0,1,2,3,4};
    
    // 두 번째 방법
    std::vector<int> vec2;
    for (int i=0; i<4; i++) {
        vec2[i] = i;
    }
    
    // 세 번째 방법
    std::vector<int> vec3;
    for (int i=0; i<4; i++) {
        vec3.push_back(i);
    }
}
```



## vector 순회 방법

```c++
std::vector<int> vec1{0,1,2,3,4};

for (std::size_t i=0; i <vec1.size(); i++){
    std::cout << vec1[i] << std::endl;
}

for (auto itr = vec1.begin(); itr != vec1.end(); itr++) {
    std::cout << (*itr) << std::endl;
}

// ranged for, 가장 안전하고 추천되는 방법
for (const int & num : vec1) {
    std::cout << num << std::endl;
}
```



# vector 기초



## vector의 시간 복잡도

```c++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> nums(1000,1); // 1을 1000개 넣기
    // 연속적으로 정의된 자료형이기 때문에 특정 index의 원소에 접근하는 것은 O(1)의 시간 복잡도를 가진다
    nums[100];
    nums.emplace_back(2);
    nums.pop_bacK();
    
    // 특정위치에 삽입하거나 삭제하는 것은 O(n)의 시간 복잡도를 가진다.
    nums.erase(nums.begin()); // 첫 번째 원소를 삭제 후 나머지 원소들을 한 칸씩 앞으로 땡겨야 한다.
    return 0;
}
```



## push_back과 emplace_back



- push_back은 '객체'를 집어넣는 형식이라서 객체가 없이 삽입을 하려면 임시 객체(r-value)가 필요하다. 즉 복사하는 작업이 필요하다.
- emplace_back은 c++ 11에서 도입된 방법으로, 가변인자 템플릿을 이용하여 객체 생성에 필요한 인자만 받은 후 함수 내에서 객체를 생성해서 삽입하는 방법이다.
- emplace_back이 쓸데없는 복사를 하지 않기 때문에 성능상 유리하다.



# 벡터 메모리, noexcept



- 맨뒤에 삽입하는 것은 일반적으로 O(1)의 시간 복잡도를 가지지만, 이것이 항상 개런티 되는 것은 아니다. 일부 상황의 경우 O(n)의 시간 복잡도를 가지게 된다.

```c++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> nums;
    std::cout << sizeof(nums); 
}
```

위 코드에서 nums는 따로 초기화를 해주지 않아도 24바이트의 공간을 가지고 있다.

stack에 nums가 만들어지고 nums는 heap의 빈 공간을 가리키고 있다. nums는 stack 공간에서 포인터 정보(heap의 시작 위치. 64비트 컴퓨터에서는 8바이트를 차지한다), 나머지 16 바이트 중 8바이트는 size에 대한 정보, 8바이트는 capacity에 대한 정보를 갖는다.



```c++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> nums{1,2,3,4,5};
    std::cout << nums.size() <<std::endl; // 5 출력
    std::cout << nums.capacity() << std::endl; // 5 출력

    nums.emplace_back(6);
    std::cout << nums.size() <<std::endl; // 6 출력
    std::cout << nums.capacity() << std::endl; // 10 출력, vector가 확보한 메모리
    
}
```



메모리가 1바이트씩 늘어나는 것이 아니라 성능을 위해서 일반적으로 capacity를 2배씩 늘린다. (reserve를 이용하여 프로그래머가 메모리 크기를 할당할 수 있음)



## 삽입이 O(n)의 시간 복잡도를 가지는 경우



- 할당된 메모리 공간을 다 썼을 때 vector는 시퀀스 자료형이므로 다음 공간을 할당하려고 할 것이다. 

- 그런데 그 공간을 다른 변수가 사용하고 있다면, 그쪽 방향으로 증가시킬 수가 없다. 

- 그래서 move 또는 copy를 통해 새로운 공간을 할당하고 기존 것은 메모리 해제되고 nums는 새로운 공간을 가리키게 된다. 

- 이럴 때 O(n)의 시간 복잡도를 가지게 되는데 reserve를 통해서 미리 메모리를 확보해놓으면 이런 불상사가 사라질 수 있다. (하지만 너무 많이 확보해놓으면 메모리 낭비다)



# array

- C 스타일의 배열 선언 방식인 `int nums[100];` 보다 `std::array<int,100> nums;`를 추천

- 컴파일 타임에 메모리 크기 결정되어 stack frame에 올라옴 : vector와의 큰 차이

- vector와 array 모두 random access를 지원해준다.
- vector와 array 모두 시퀀스 타입이다.
- array가 메모리 할당 시간이 vector보다 비교적 빠르다.
- 배열을 쓰고 싶은데 사이즈가 정해져있고 작다면, 무조건 array를 쓰면 된다.



# deque (double-ended-queue)

- O(1)의 random access를 지원해준다.
- 실제 업무에서는 거의 쓰지도 않는다.
- 앞뒤에 데이터를 넣고 뺄 수 있다.
- 데이터가 연속적으로 저장되어 있지는 않다.
- 하드웨어적 관점에서 캐시 미스가 날 수도 있다.
- 일단 이런게 있나보다 하고 넘어가자... PS에서 거의 쓰지도 않을 것 같다.

