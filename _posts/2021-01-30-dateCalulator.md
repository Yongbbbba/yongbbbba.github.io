---
title: "[씹어먹는 C++] 객체지향프로그래밍 : 날짜계산기 만들기"
excerpt: ""

categories:
  - TIL
tags:
  - C++
 
last_modified_at: 2021-01-30T22:00:00
toc: true
toc_sticky: true
---

[모두의 코드 블로그](https://modoocode.com/172)에 있는 `씹어먹는 C++`에서 객체지향 프로그래밍 부분의 `생각해보기` 문제이다. 



```
생각 해볼 문제

문제 1

여러분은 아래와 같은 Date 클래스를 디자인 하려고 합니다. SetDate 는 말그대로 Date 함수
내부를 초기화 하는 것이고 AddDay, AddMonth, AddYear 는 일, 월, 년을 원하는 만큼 더하게
됩니다. 한 가지 주의할 점은 만일 2012 년 2 월 28 일에 3 일을 더하면 2012 년 2 월 31 일이 되는
것이 아니라 2012 년 3 월 2 일이 되겠지요? (난이도 : 上)
```



만만히 봤다가 시간을 엄청 잡아먹었다. 부족함을 느낀다.
윤년 처리는 하지 못 했고, AddDay 함수를 간결하게 쓰지 못했다.
반복문을 쓸까 하다가 switch 문을 활용하기로 하였는데, 이 안에서 재귀를 활용할 수 없을까 고믾다가 구현을 못 하고 goto문을 사용하였다. goto문은 스파게티 코드를 양산하기에 지양하라는 말을 들었었는데, 오늘의 이 스파게티 코드는 goto 문 때문이 아닌 것 같다.. 



```c++
#include <iostream>

class Date
{
    int year;
    int month; // 1부터 12 까지
    int day;   // 1부터 31 까지

public:
    void SetDate(int year_, int month_, int date_)
    {
        year = year_;
        month = month_;
        day = date_;
    }

    void AddDay(int inc)
    {

    tryAgain: //Goto 문 활용
        switch (month)
        {
        // 31일 까지 있는 달 : 1,3,5,7,8,10,12
        case 1:
        case 3:
        case 5:
        case 7:
        case 8:
        case 10:
            if (day + inc > 31)
            {
                // inc를 더했는데 달이 넘어가면, month를 1 올려주고 day를 0으로 만든다. 현재 day와 31일만큼의 차이를 inc에서 차감하고 다시 switch 문을 돌린다.
                month++;
                inc = inc - (31 - day);
                day = 0;
                goto tryAgain;
            }
            else
            {
                // inc를 더했는데 31일이 안넘으면 month에 영향이 없으므로 그냥 더하고 swith 문을 나간다.
                day = day + inc;
                break;
            }
            // 12월에는 inc를 더했을 때 년도가 바뀌는 경우가 생기므로 일종의 예외처리.
        case 12:
            if (day + inc > 31)
            {
                month = 1;
                year++;
                inc = inc - (31 - day);
                day = 0;
                goto tryAgain;
            }
            else
            {
                day = day + inc;
                break;
            }
        // 30일 까지만 있는 달 : 4,6,9,11
        case 4:
        case 6:
        case 9:
        case 11:
            if (day + inc > 30)
            {
                month++;
                inc = inc - (30 - day);
                day = 0;
                goto tryAgain;
            }
            else
            {
                day = day + inc;
                break;
            }
        // 28일까지만 있는 달 : 2
        case 2:
            if (day + inc > 28)
            {
                month++;
                inc = inc - (day + inc) % 28;

                day = 0;
                goto tryAgain;
            }
            else
            {
                day = day + inc;
                break;
            }
        }
    }

    void AddMonth(int inc)
    {
        AddYear( (month + inc) / 12);
        month = (month + inc) % 12;
           }
    void AddYear(int inc)
    {
        year += inc;
    }
    void ShowDate()
    {
        std::cout << year << "년" << month << "월" << day << "일" << std::endl;
    }
};

int main()
{
    Date date;
    date.SetDate(2021, 01, 30);
    std::cout << "현재 날짜는 ";
    date.ShowDate();
    // 3년 더하기
    date.AddYear(3);
    std::cout << "3년 더한 후 : ";
    date.ShowDate();
    // 4개월 더하기
    date.AddMonth(4);
    std::cout << "4개월 더한 후 : ";
    date.ShowDate();
    // 14개월 더하기
    date.AddMonth(14);
    std::cout << "14개월 더한 후 : ";
    date.ShowDate();
    // 1일 더하기
    date.AddDay(1);
    std::cout << "1일 더한 후 : ";
    date.ShowDate();
    // 30일 더하기
    date.AddDay(30);
    std::cout << "30일 더한 후 : ";
    date.ShowDate();
    // 100일 더하기
    date.AddDay(200);
    std::cout << "200일 더한 후 : ";
    date.ShowDate();
}

```

