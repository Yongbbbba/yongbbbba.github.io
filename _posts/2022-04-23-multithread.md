---
title: "[TIL][C++]소수구하기를 통해 확인하는 멀티스레드 성능"
excerpt: ""

categories:
  - TIL
tags:
  - TIL
  - C++
 
last_modified_at: 2022-04-23T17:00:00
toc: true
toc_sticky: true
---

# 소수구하기를 통해 확인하는 멀티스레드 성능

## 싱글스레드로 소수 구하기

```c++
#include <iostream>
#include <memory>
#include <mutex>
#include <chrono>
#include <vector>
#include <thread>

using namespace std;

const int max_count = 500000;
const int thread_count = 4;

bool IsPrimeNum(int num) {
	if (num == 0 || num == 1) return false;
	if (num == 2 || num == 3) return true;
	int temp = num / 2;
	for (int i = 2; i <= temp; i++) {
		if (num % i == 0) return false;
	}
	return true;
}

void PrintVector(vector<int>& vec) {
	for (auto& num : vec) {
		cout << num << " ";
	}
	cout << endl;
}

int main() {
	int num = 1;
	vector<int> prime_arr;

	auto t0 = chrono::system_clock::now();
	while (num <= max_count) {
		if (IsPrimeNum(num)) {
			prime_arr.push_back(num);
		}
		num++;
	}

	// PrintVector(prime_arr);

	auto t1 = chrono::system_clock::now();

	auto duration = chrono::duration_cast<chrono::milliseconds>(t1 - t0).count();

	cout << "경과시간 : " << duration << " 밀리초 " << endl;
	return 0;
}

// 16499 밀리초
```

## 멀티스레드로 소수구하기

- 주의할 점
  - num에 mutex 설정하기
  - prime_arr에 mutex 설정하기
    - 여기에도 mutex를 설정해야하는건가 의문을 가질 수 있는데, c++의 vector는 가지고 있는 원소의 수보다 큰 케파를 잡고 있는다. 만약 이 케파보다 원소의 수가 커지면 resizing을 하게되는데, resizing을 수행한 스레드외의 스레드가 기존 vector의 주소를 가리키며 작업을 수행하게 되면 쓰레기값에다 vector push_back을 하게 되는 꼴이므로 크래시가 발생한다. 그래서 mutex를 잡아야한다.

```c++
#include <iostream>
#include <memory>
#include <mutex>
#include <chrono>
#include <vector>
#include <thread>

using namespace std;

const int max_count = 500000;
const int thread_count = 4;

bool IsPrimeNum(int num) {
	if (num == 0 || num == 1) return false;
	if (num == 2 || num == 3) return true;
	int temp = num / 2;
	for (int i = 2; i <= temp; i++) {
		if (num % i == 0) return false;
	}
	return true;
}

void PrintVector(vector<int>& vec) {
	for (auto& num : vec) {
		cout << num << " ";
	}
	cout << endl;
}

int main() {
	int num = 1;
	recursive_mutex num_mutex;
	vector<int> prime_arr;
	recursive_mutex vector_mutex;

	auto t0 = chrono::system_clock::now();

	vector<shared_ptr<thread>> threads;

	for (int i = 0; i < thread_count; i++) {
		shared_ptr<thread> th(new thread([&]() {
			while (true) {
				int n;
				{
					lock_guard<recursive_mutex> num_lock(num_mutex);
					n = num++;
				}
				if (n > max_count) break;

				if (IsPrimeNum(n)) {
					lock_guard<recursive_mutex> vector_lock(vector_mutex);
					prime_arr.push_back(n);
				}
			}
			}));
		threads.push_back(th);
	}

	for (auto thread : threads) {
		thread->join();
	}

	// PrintVector(prime_arr);

	auto t1 = chrono::system_clock::now();

	auto duration = chrono::duration_cast<chrono::milliseconds>(t1 - t0).count();

	cout << "경과시간 : " << duration << " 밀리초 " << endl;
	return 0;
}

// 5524 밀리초
```

