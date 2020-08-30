---
title: CompletableFuture
tags:
  - java
  - CompletableFuture
categories: JAVA
layout: post
---

# CompletableFuture

## Future Interface

* 미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있는 인터페이스
* Future는 계산이 끝났을 때 결과에 접근할 수 있는 레퍼런스를 제공
* 시간이 걸릴 수 있는 작업을 Future 내부로 설정\(Callable 객체 내부로 감싼 다음에 ExecutorService에 제출\)하면 

  호출자 스레드가 결과를 기다리는 동안 다른 유용한 작업을 수행할 수 있음

* 호출자 스레드가 블록되지 않고 다른 작업을 실행할 수 있음
* 오래 걸리는 작업이 끝나지 않을 수 있으므로, 결과를 가져오는 get 메서드를 오버로드해서 최대 타임아웃 시간을 설정하는 것이 좋음

| 동기 | 비동기 |
| :---: | :---: |
| 메서드를 호출한 다음 계산을 완료할 때까지 기다림 메서드가 반환되면 호출자는 반환값으로 계속 다른 동작을 수행 \(blocking call\) | 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당 \(non-blocking call\) |

## 동기 메서드를 비동기 메서드로 변환

* 블록 문제가 발생할 수 있는 상황에서는 타임아웃을 활용하는 것이 좋음 \(클라이언트가 영원히 블록되지 않고 익셉션 발생 가능\)

## CompletableFuture

* Future와 CompletionStage\(계산처리\)를 구현한 클래스
* Future와 CompletableFuture의 관계를 Collection과 Stream의 관계에 비유할 수 있다.

| 메서드 | 설명 |
| :--- | :--- |
| complete | 프로세스 완료 여부 |
| exceptionally | Throwable 처리 |
| runAsync | completableFuture 생성 \(runnable\) |
| supplyAsync | completableFuture factory \(supplier\) |
| thenCompose | 이전 async로 응답 받은 값을 다음 async 프로세스의 인자로 사용하는 경우 |
| thenCombine | 두 프로세스를 병렬로 처리하고 결과 값을 병합해서 사용하는 경우 |
| thenApply | 콜백을 설정하여 콜백 호출 \(function\) |
| thenAccept | 콜백을 설정하여 콜백 호출 \(consumer\) |
| xxxAsync | 해당 메서드의 비동기 버전 |

## 스트림 병렬화와 CompletableFuture 병렬화

* I/O가 포함되지 않은 계산 중심의 동작을 실행할 때는 스트림 인터페이스가 가장 `구현하기 간단하며 효율적`일 수 있다\(모든 스레드가 계산 작업을 수행하는 상황에서는 프로세스 코어 수 이상의 스레드를 가질 필요가 없다.\)
* 반면 작업이 I/O를 기다리는 작업을 병렬로 실행할때는 CompletableFuture가 더많은 유연성을 제공하며 대기/계산\(W/C\)의 비율에 적합한 `스레드 수를 설정`할 수 있다. 특히 스트림의 게으른 특성 때문에 스트림에서 I/O를 실제로 언제 처리할지 예측하기 어려운 문제도 있다.

## 참고

* [CompletableFuture Oracle docs](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)
* [ExecutorService](https://gompangs.tistory.com/entry/JAVA-ExecutorService-%EA%B4%80%EB%A0%A8-%EA%B3%B5%EB%B6%80)
* [CompletableFuture](https://trending.tistory.com/10)
* [Blocking/NonBlocking, Synchronous/Asynchronous](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/)

