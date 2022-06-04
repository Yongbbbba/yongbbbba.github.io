---
title: "[C++]복사 생성자와 이동 생성자 유의점과 속도 체크"
excerpt: ""

categories:
  - TIL
tags:
  - C++
 
last_modified_at: 2022-06-04T23:00:00
toc: true
toc_sticky: true
---



# 복사 생성자와 이동 생성자copy constructor && move constructor

## 복사 생성자를 만들지 않았을 때 유의해야할 점

C++에서 사용자가 생성자를 만들지 않으면 컴파일러가 알아서 기본적으로 생성자들을 만들어준다. 그런데 객체의 멤버 변수 중 포인터 변수가 있을 경우에 얕은 복사가 일어날지 깊은 복사가 일어날지 궁금해졌다. 보통은 얕은 복사를 의도하지 않기 때문에. 

```c++
// 포인터 멤버변수가 있고, move, copy 생성자를 정의하지 않았을 때 실험
// move 생성자와 copy 생성자의 속도 차이 실험
class Test
{
public:
	int* mPtr = nullptr;
	int mNum = 0;
};


int main()
{
	ios::sync_with_stdio(0);
	cin.tie(0);

	// 복사 생성자/이동 생성자 따로 만들지 않으면 깊은 복사가 일어남
	// 포인터 멤버 변수를 가지고 있을 때 깊은 복사가 일어나면 의도하지 않은 동작이 일어날 가능성이 매우 높음
	int arr[4] = { 1,2,3,4 };
	Test a;
	a.mNum = 10;
	a.mPtr = arr;
	Test b(a);
	cout << "a.mPtr " << a.mPtr << endl;
	cout << "b.mPtr " << b.mPtr << endl;
//	 a와 b의 mPtr 변수는 같은 값을 가지게 된다. 
// a.mPtr 00A0FD3C
// b.mPtr 00A0FD3C

}
```

대부분 위와 같은 결과물을 기대하지는 않을 것이기 때문에 copy contructor를 사용자가 잘 정의하는 것이 중요하다는 것을 다시 한 번 상기. 



## 복사 생성자와 이동 생성자의 속도 비교

책이나 블로그 글들을 보면 이동 생성자가 복사 생성자보다 더 빠르다고 한다. 정말 그럴지 궁금해서 테스트 해봤다. 

```c++

class TestArray
{
public:
	TestArray(int size) : mArr(new int[size]), mSize(size)
	{
		for (int i = 0; i < mSize; i++)
		{
			mArr[i] = i;
		}
		cout << "constructor" << endl;
	}

	// copy constructor
	TestArray(const TestArray& other) : mArr(new int[other.mSize]), mSize(other.mSize)
	{
		for (int i = 0; i < mSize; i++)
		{
			mArr[i] = other.mArr[i];
		}
		cout << "copy constructor" << endl;
	}

	// move constructor
	TestArray(TestArray&& other) : mArr(other.mArr), mSize(other.mSize)
	{
		// 소유권 삭제 
		other.mArr = nullptr;
	}
	~TestArray()
	{
		delete[] mArr;
	}

	int* mArr;
	int mSize;
};

int main()
{
	cout << "복사생성자, 이동생성자 속도 비교" << endl;
	TestArray testArray(1000000);
	// copy 
	auto start = chrono::high_resolution_clock::now();
	TestArray copy(testArray);
	auto end = chrono::high_resolution_clock::now();
	chrono::duration<double> diff = end - start;
	cout << "copy speed: " << diff.count() * 1000 << endl;

	cout << endl;

	// move 
	start = chrono::high_resolution_clock::now();
	TestArray testMove(move(testArray));
	end = chrono::high_resolution_clock::now();
	diff = end - start;
	cout << "move speed: " << diff.count() * 1000 << endl;

	return 0;
    
    // 이동 생성자가 월등히 빠름
    // copy speed: 4.279
    // move speed: 0.0009
}
```

위와 같이 이동 생성자가 월등히 빠르다. 내 프로젝트에서 이동 생성자를 활용한 최적화를 할 수 있는 여지가 있는지 체크해볼 필요가 있겠다. 



## 복사 생성자와 이동 생성자 속도 차이의 이유

copy semantics와 move semantics는 둘다 자신과 동일한 타입의 객체를 인자로 받아서, 새롭게 생성한 객체에 자신의 내용물을 입력한다는 점에서는 동일하다. 하지만 복사 생성자는 새롭게 생성되는 객체의 멤버 변수를 위한 메모리 공간을 새로 할당하기 때문에 cost가 더 발생한다. 하지만 move는 그냥 너나 가져라~ 하면서 소유권을 던져주기 때문에 새로 뭔가를 만들 필요가 없어서 속도가 더 빠르다. 