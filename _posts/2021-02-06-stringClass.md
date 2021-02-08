---
title: "[씹어먹는 C++] String 클래스 구현"
excerpt: ""

categories:
  - TIL
tags:
  - C++

last_modified_at: 2021-02-07T11:10:00
toc: true
toc_sticky: true
---

# string 클래스 구현 목적

- 기존 C언어에서는 문자열을 나타내기 위하여 NULL 종료 문자열이라는 개념을 도입해서 문자열 끝에 NULL 문자를 붙여 문자열을 나타냈다.
- 하지만 이 때문에 만들어진 문자열의 크기를 바꾼다던지, 문자열 뒤에 다른 문자열을 붙인다던지의 작업이 매우 귀찮았다.
- 그래서 C++에는 이를 편리하게 하기 위하여 string 라이브러리가 이미 구현되어 있다.
- 이 string 라이브러리를 구현해보면서 C++의 클래스와 객체지향 프로그래밍에 대한 이해력을 높이고,  구현력을 높이는 것이 목적이다.
- 처음에는 따라서 구현해보고, 나중에 복기하면서 혼자 다시 만들어보는 중.



# string 클래스 구성 목록

### 멤버 변수

1. memory_capacity : 해당 문자열에 할당된 메모리 크기, 문자열 크기보다 더 큰 공간을 할당할 수 있다.

2. string_content : 저장할 문자열

3. string_length : 저장할 문자열의 길이

   

### 메소드

- [x] 문자 c 혹은 C 형식 문자열 str 에서 생성할 수 있는 생성자와 복사 생성자
- [x] 문자열의 길이를 리턴하는 함수(length)
- [x] 문자열 대입 함수(assign)
- [x] 문자열 메모리 할당 함수(reserve) 및 현재 할당된 크기를 알아오는 함수(capacity)
- [x] 특정 위치의 문자를 리턴하는 함수(at)
- [x] 특정 위치에 특정 문자열을 삽입하는 함수(insert)
- [ ] 특정 위치의 특정 개수의 문자를 지우는 함수(erase)
- [ ] 특정 위치를 시작으로 특정 문자열을 검색하는 함수(find)
- [ ] 두 문자열을 사전식 비교하는 함수(compare)



### 세부사항 및 특징

- 생성자

  - 파라미터에 const 붙이지 않으면 컴파일 에러는 아니고 conversion이 안된다며 표준에 어긋나다가 경고가 뜸

- 특정 위치의 문자를 리턴하는 함수(at)

  ```cpp
  // 특정 위치의 문자를 리턴
  char MyString::at(int idx) {
      // idx가 out of range라면 NULL 반환, C 스타일의 배열에 담긴 string이었다면 쓰레 값을 반환했을 것
      if (idx >= string_length || idx < 0) 
          return 0; // char이 아닌 NULL을 반환하면 반환값이 non_pointer type이기 떄문에 컴파일 경고가 뜨지만 컴파일은 그대로 진행된다. 자동으로 return 값에 대한 형변환이 이루어진다. 여기서는 그냥 0을 반환한다. 
      return string_content[idx];
  }
  ```

- 문자열 메모리 할당 함수(reserve) 

  ```cpp
  // 메모리 할당하기
  // //현재 메모리 용량이 할당하고자 하는 메모리 용량보다 작을 경우 기존 것을 해제하고 다시 할당
  // // 할당하고자 하는 용량이 현재보다 작을 경우에는 에러 반환
  void MyString::reserve(int n) {
      if (memory_capacity < n) {
          char *prev_content = string_content;
          memory_capacity = n;
          delete[] string_content;
          string_content = new char[memory_capacity];
  
          for (int i=0; i != string_length; i++) {
              string_content[i] = prev_content[i];
          }
      } else {
          std::cout << "memory size is smaller than present size" << std::endl;}
  
      }
  }
  ```

- 문자열 대입 함수(assign)

  - str.assign("abc") : 기존 문자열이 비워지고 "abc" 대입
  - 기존 할당된 메모리 크기보다 더 긴 문자열이 대입된다면 기존의 것을 메모리 해제하고 다시 할당하여 대입
  - 모두의코드의 예제에서 void가 아니라 레퍼런스를 리턴하는데 그 이유를 모르겠다.

- 문자열 삽입 함수(insert)

  - 확보한 메모리 용량을 다 사용하고 있는 상태에서 'a' 같은 글자를 한 글자씩 삽입하는 작업을 수행하게 될 때, 메모리 용량을 확보하기 위해 메모리 해제와 재할당을 반복하게 되면 작업의 낭비가 있을 수 있다. 
  - 그래서 memory_capacity를 두 배만큼 미리 늘려놓는 것도 이러한 작업을 줄이는 방법 중 하나이다. 두 배는 엄밀한 숫자는 아니다. 너무 크면 그만큼 메모리 공간 낭비가 생길 수 있으니 적절한 수치를 프로그래머가 할당해야한다.

- 문자열 제거 함수(erase)
  - 뒤에 문자를 땡겨오는 아이디어는 교재와 똑같이 생각했는데 구현을 못 했다. 구현력, 특히 C++ 라는 도구를 아직 잘 사용하지 못하고 있다. 아마 파이썬이었다면 쉽게 구현을 했겠지만.. 더 많이 연습을 해야한다.
  - string_length를 도입한 것의 장점은 string_length 이후의 문자들을 굳이 초기화할 필요가 없다는 것이다. string_length 때문에 쓰레기 문자가 되어버린 그 문자들에 접근할 수 없다.



### 전체 코드

```cpp
#include <iostream>
#include <string.h>


class MyString{
    char* string_content; // 문자열을 가리키는 포인터
    int string_length;// 문자열 길이
    int memory_capacity;// 문자열에 할당한 메모리 크기 

    public:
    // 생성자
    MyString(const char c); // 한 글자만 넣었을 때
    MyString(const char *str); // 문자열을 넣었을 때
    MyString(const MyString &str); // 복사생성자
    // 소멸자
    ~MyString();
    // 문자열을 출력해주는 함수(print, println)
    void print();
    void println();
    // 문자열 길이를 리턴하는 함수(length) 
    int length();
    // 문자열 대입 함수(assign)
    void assign(const MyString& str);
    void assign(const char* str);
    // 문자열 메모리 할당 함수(reserve) 및 현재 할당된 크기를 알아오는 함수 (capacity)
    void reserve(int n);
    int capacity();
    // 특정 위치의 문자를 리턴하는 함수(at)
    char at(int idx);
    // 특정 위치에 특정 문자열을 삽입하는 함수(insert)
    MyString& insert(int loc, const MyString& str);
    MyString& insert(int loc, char c);
    MyString& insert(int loc, const char* str);
    // 특정 위치의 특정 개수의 문자를 지우는 함수(erase)
    MyString& erase(int loc, int num);
    // 특정 위치를 시작으로 특정 문자열을 검색하는 함수(find)
    // 두 문자열을 사전식 비교하는 함수(compare)
};

// 생성자
// 한 글자만 넣었을 때
MyString::MyString(const char c) {
    string_content = new char[1];
    string_content[0] = c;
    memory_capacity = 1;
    string_length = 1;
}
// 문자열이 들어왔을 때
MyString::MyString(const char *str) {
    string_length = strlen(str);
    memory_capacity = string_length;
    string_content = new char[string_length];

    for (int i=0; i !=string_length; i++) string_content[i] = str[i];
}
// 복사 생성자
MyString::MyString(const MyString& str) {
    string_length = str.string_length;
    memory_capacity = str.memory_capacity;
    string_content = new char[string_length];
    for (int i=0; i!=string_length; i++) {
        string_content[i] = str.string_content[i];
    }
}
// 소멸자
MyString::~MyString() { delete[] string_content;}

// 문자열 출력
void MyString::print() {
    for (int i=0; i != string_length; i++) {
        std::cout << string_content[i];
    }
}
void MyString::println() {
    for (int i=0; i!= string_length; i++) {
        std::cout << string_content[i];
    }
    std::cout << std::endl;
}
// 문자열 길이 반환
int MyString::length() {
    return string_length; 
}
// 특정 위치의 문자를 리턴
char MyString::at(int idx) {
    // idx가 out of range라면 NULL 반환, C 스타일의 배열에 담긴 string이었다면 쓰레 값을 반환했을 것
    if (idx >= string_length || idx < 0) 
        return 0; // char이 아닌 NULL을 반환하면 반환값이 non_pointer type이기 떄문에 컴파일 경고가 뜨지만 컴파일은 그대로 진행된다. 자동으로 return 값에 대한 형변환이 이루어진다. 여기서는 그냥 0을 반환한다. 
    return string_content[idx];
}
// 현재 할당된 메모리 크기 확인
int MyString::capacity() {return memory_capacity;}
// 메모리 할당하기
// //현재 메모리 용량이 할당하고자 하는 메모리 용량보다 작을 경우 기존 것을 해제하고 다시 할당
// // 할당하고자 하는 용량이 현재보다 작을 경우에는 에러 반환
void MyString::reserve(int n) {
    if (memory_capacity < n) {
        char *prev_content = string_content;
        memory_capacity = n;
        string_content = new char[memory_capacity];

        for (int i=0; i != string_length; i++) {
            string_content[i] = prev_content[i];
        }
        delete[] prev_content;
    } else {
        std::cout << "memory size is smaller than present size" << std::endl;}

}

// 문자열을 새로 대입
// str의 문자열 길이가 기존보다 길다면 기존 것을 해제하고 다시 할당
void MyString::assign(const MyString& str) {
    if (str.string_length > string_length) {
        string_length = str.string_length;
        memory_capacity = string_length;
        delete[] string_content;
        string_content = new char[memory_capacity];

    } else {
        string_length = str.string_length;
    }
    for (int i=0; i!=string_length; i++) {
        string_content[i] = str.string_content[i];
    }
}
void MyString::assign(const char* str) {
    if (strlen(str) > string_length) {
        string_length = strlen(str);
        memory_capacity = string_length;
        delete[] string_content;
        string_content = new char[memory_capacity];
    } else {
        string_length = strlen(str);
    }
    for (int i=0; i!=string_length; i++) {
        string_content[i] = str[i];
    }
}
// 특정 위치에 문자 또는 문자열 삽입
MyString& MyString::insert(int loc, const MyString& str) {
    // 범위를 벗어나면 객체 리턴
    if (loc < 0 || loc > string_length) {
        return *this;
    }

    if (string_length + str.string_length > memory_capacity) {
        // 메모리 공간을 미리 확보해서 공간을 희생하는 대신 속도를 높인다.
        if (memory_capacity * 2 > str.string_length + string_length) {
            memory_capacity *= 2;
        } else {
        memory_capacity = string_length + str.string_length;
        }
        char* prev_string_content = string_content;
        string_content = new char[memory_capacity];
        // loc 이전까지의 내용 복사
        int i;
        for (i=0; i<loc; i++) {
            string_content[i] = prev_string_content[i];
        }
        // 새로운 내용 삽입
        for (int j=0; j != str.string_length; j++) {
            string_content[i+j] = str.string_content[j];
        }
        // loc 이후의 내용 복사
        for (;i<string_length;i++) {
            string_content[i + str.string_length] = prev_string_content[i];
        }

        delete[] prev_string_content;
        string_length = string_length + str.string_length;
        return *this;
    }
    // 동적 할당이 필요가 없는 경우
    for (int i= string_length-1; i>=loc; i--) {
        // 뒤로 밀기, 뒤에서부터 대입
        string_content[i + str.string_length] = string_content[i];
    }
    // 그리고 insert 되는 문자 다시 집어넣기
    for (int i=0; i< str.string_length; i++) {
        string_content[i+loc] = str.string_content[i];
    }

    string_length = string_length + str.string_length;
    return *this;
}
MyString& MyString::insert(int loc, char c) {
    MyString temp(c);
    return insert(loc, temp);
}
MyString& MyString::insert(int loc, const char* str) {
    MyString temp(str);
    return insert(loc, temp);
}

MyString& MyString::erase(int loc, int num) {
    // 지우려는 문자의 갯수가 out of range일 경우
    if ((string_length-loc) < num) return *this;
    if (loc < 0 || num < 0 || loc > string_length) return *this; 
    
    for (int i = loc+num; i < string_length; i++) {
        string_content[i-num] = string_content[i];
    }
    string_length -= num;
    return *this;
}

int main() {
    MyString str1("abcdef");
    std::cout << "string content : " ; 
    str1.println();
    std::cout << "capacity : "  << str1.capacity() << std::endl;
    std::cout << "length : " << str1.length() << std::endl;
    str1.erase(2,6);
    std::cout << "string content : "; 
    str1.println();
    std::cout << "capacity : "  << str1.capacity() << std::endl;
    std::cout << "length : " << str1.length() << std::endl;
        return 0;

}


```



