---
title: "[C] (내용추가예정) 문자열 저장 방법 차이(pointer vs char 배열)"
excerpt: ""

categories:
  - C
tags:
  - C
 
last_modified_at: 2021-12-11T17:00:00
toc: true
toc_sticky: true
---



# 문자열 저장 방법 차이(pointer vs char 배열)

C언어에서는 문자열을 char 타입을 원소로 가지는 배열에 저장해서 문자열을 나타낸다. 또한, C에서 배열의 이름은 sizeof와 &연산자를 사용할 때를 제외하고는 포인터로 형변환이 되는 것을 이용해 문자열을 포인터에 저장할 수 있다. 그 모양은 아래와 같다.

```c
	const char* str = "abcdefghi";
	char c_str[] = { "abcdefghi" };
```

이 둘은 완전히 동일할까? visual studio 2019의 디스어셈블리를 통해 확인해보면 아래와 같은 결과를 얻을 수 있다.

```assembly
 	const char* str = "abcdefghi";
009052C5  mov         dword ptr [str],offset string "sentence" (0907CD8h)  
	char c_str[] = { "abcdefghi" };
009052CC  mov         eax,dword ptr [string "sentence" (0907CD8h)]  
009052D1  mov         dword ptr [c_str],eax  
009052D4  mov         ecx,dword ptr [string "\xb4\xd9\xbd\xc3 \xc0\xd4\xb7\xc2\xc7\xd8\xc1\xd6\xbc\xbc\xbf\xe4\n" (0907CDCh)]  
009052DA  mov         dword ptr [ebp-18h],ecx  
009052DD  mov         dx,word ptr [string "\xbf\xb5\xbe\xee" (0907CE0h)]  
009052E4  mov         word ptr [ebp-14h],dx  
```

뭔진 몰라도 두 문자열 저장 방법이 동일하지 않다는 것을 알 수 있다. 

전자의 경우는 "abcdefghi"를 메모리에 올려두고, 그 메모리 주소를 str 포인터로 가리키고 있는 것이고, 후자의 경우는 char 타입의 배열을 선언하고 메모리에 할당을 하고, 각 char 원소를 해당 배열에 저장하는 방식이다.



(좀 부정확해서 내용 추가보강 예정 ... )