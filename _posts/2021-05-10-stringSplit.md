---
title: "[C++] C++에서 문자열 split 하기(stringstream, getline 정리)  "
excerpt: ""

categories:
  - TIL
tags:
  - C++
  
 
last_modified_at: 2021-05-10T19:30:00
toc: true
toc_sticky: true
---



# C++에서 문자열 split해서 저장하기

어제 코테를 보다가 문자열을 공백을 기준으로 split 해야하는 상황이 있었다. C++로는 문자열을 많이 다뤄본 적이 없어서 숨간 멈칫하고 바로 python으로 바꿔서 풀었다 ㅋㅋㅋ 이런 상황이 또 나와서는 안되기에 split하는 방법을 찾아봤다.



C++에서는 python처럼 따로 split 함수가 있지 않고, stringstream과 getline을 이용해서 split을 구현할 수 있다. 그래서 우선 stringstream 과 getline을 잘 알아야 한다.



## getline

여기서 말하는 getline은 `<string>`에 정의된 getline을 말한다. 공식 문서를 살펴보면 아래와 같이 되어있다.

```c++
istream& getline (istream& is, string& str, char delim);
istream& getline (istream& is, string& str);
```

쉽게 말해서 입력 스트림(is)에 있는 문자열을 str에 저장하는 함수인데, 세 번째 인자를 넣어줄 경우에는 구분자 역할을 한다. string을 쭉 읽다가 delimiter를 발견하게 되면 delimiter는 버려주고 읽어들이기를 종료한다. 만약에 세 번째 인자를 넣어주지 않는 경우에는 디폴트로 개행문자가 설정된다. 

getline은 입력받은 문자열을  return한다.



## stringstream

`<sstream>`에 정의되어있다. C++은 stream은 내용이 생각보다 많고 복잡했었는데.. 그래서 여기에서 언급할 내용은 아닌 것 같고 스트림은 쉽게 말해 입출력을 추상화한 것이다. sstream은 문자열과 스트림의 기능을 합쳐놓은 것이라고 생각하고 넘어가면 된다.



이정도까지 알면 split을 구현할 수 있다.



------



```c++
#include<iostream>
#include<string>
#include<sstream>>
 
using namespace std;
 
int main()
{
    string input = "Life,is,short,you,need,python(?)";
    istringstream ss(input);
    string result;
    while (getline(ss, result, ','))
    {
        cout << result << " ";
    }
 
    return 0;
}
```



위 코드를 설명하자면, 

1. input을 입력스트림 ss에 저장한다.
2. while 문에서 입력스트림을 쭉 읽다가 delimiter를 만나면 delimiter는 버려주고 읽어들이기를 종료한다. 그리고 이전까지 읽어들인 문자열을 result에 저장한다.
3. result를 출력한다.
4. 첫 번째 반복이 끝나면 입력스트림에는 기존 input의 `Life,`부분은 빠져있게 되는데, 이러한 작업을 반복하다면 끝에 가서는 입력 스트림이 모두 비어질 것이고 getline은 ss를 return 하므로 반복문은 종료된다.



위에서는 출력을 예시로 들었지만 이걸 배열에 담고 싶으면 배열에 담으면 된다.