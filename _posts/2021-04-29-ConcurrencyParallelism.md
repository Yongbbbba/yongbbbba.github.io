---
title: "[CS] 간단히 살펴보는 동시성과 병렬성"
excerpt: ""

categories:
  - TIL
tags:
  - CS
 
last_modified_at: 2021-04-29T19:00:00
toc: true
toc_sticky: true
---



# 동시성(Concurrency)과 병렬성(Parallelism)

![Concurrency and parallelism are two different things - Blog | luminousmen](/assets/post_images/2021-04-29-ConcurrencyParallelism.assets/concurrency-and-parallelism-are-different.jpg)

동시성과 병렬성은 동시에 여러가지를 실행하는 작업 수행하는 것을 일컫기 때문에 언뜻 보면 같은 개념인 것처럼 보인다. 하지만 까보면 그 둘은 다른 특성이다.

## 동시성

동시에 실행되는 것 **같이** 보이는 것이다.  멀티스레딩 환경에서 존재하며, time-slicing을 포함한다. 

컴퓨터 코어는 한 번에 한 가지 명령어를 처리할 수 있다. 그렇기 때문의 한 번에 다수의 스레드의 작업을 동시에 처리할 수 없다. 싱글 코어 환경에서 다수의 스레드를 매우 빠르게 Context-Switching 하면서 마치 동시에 실행되는 것 같은 착각을 불러일으키는 것이 `동시성`이다. 



## 병렬성

적어도 2개 이상의 코어가 존재한다는 조건에서 각 코어가 실제로 동시에 여러 task를 처리하는 것을 말한다. 