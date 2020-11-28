+++

authors = [
    "Lena"
]
title = "동시성 프로그래밍(Concurrency programming): dispatchQueue 사용시 주의할 점들"
date = 2020-11-28T14:47:51+09:00
description = "Concurrency programming: cautions when using dispatchQueue"
tags = [
    "iOS", "Concurrency programming", "GCD", "DispatchQueue", "Sync", "Async"
]
categories = [
     "iOS", "Concurrency programming", "GCD"
]
series = ["Concurrency programming"]
images = [
  "/images/mainQueue(Serial)+Sync.png"
]
draft = false

+++

[동시성 프로그래밍 시리즈] dispatchQueue 사용시 주의할 점들에 대해 소개합니다.<br>

<br>

<!--more-->



##    <  📑 목차  >

* DispatchQueue(디스패치큐) 사용시 주의할 점

  * 1. 메인 큐에서의 주의할 점

    * 1.1. UI 업데이트는 메인큐에서 해야합니다

    * 1.2. 메인 큐에서 다른 큐로 보낼때 sync 메서드를 사용하면 안됩니다

  * 2. 현재의 큐에서 현재의 큐로 작업을 보낼 때 주의할 점

<br><br>

### 👉🏻  [동시성 프로그래밍(Concurrency programming): 스레드와 큐(Thread and Queue)](https://lena-chamna.netlify.app/post/concurrency_programming_thread_and_queue/)에서 이어지는 내용입니다.ㄴ

<br><br>

## <span style="color: #6666FF">DispatchQueue(디스패치큐) 사용시 주의할 점</span>

#### <span style="color:orange">**1. 메인 큐에서의 유의사항**</span>

<img src="https://i.imgur.com/X7mO4mq.png" style="zoom:50%;" />

<br>

**1.1. UI 업데이트는 메인큐에서 해야합니다**

 UI 관련 일들은 **메인큐**에서 처리해야 합니다. iOS 뿐만 아니라 모든 OS에서 화면을 표시하는 일은 한 개의 스레드에서만 담당해야하는데요. 그래야 **간섭이 일어나지 않기 때문**이라고 합니다. (간섭이 일어나면 화면이 정상적으로 표시되지 않고 화면이 깜빡일 수 있다고 하네요!) 그렇기 때문에 **다른 스레드로 보낸 작업에서 UI 업데이트를 해야하는 상황이 있을 때에는 UI 업데이트에 대한 작업을 메인 스레드로 보내 처리를 해야합니다**.

```swift
// API에서 비동기로 이미지를 받아와 imageView에 넣어주는 예시

URLSession.shared.dataTask(with: url){
  //이미지 다운로드 
  DispatchQueue.main.async{ //메인큐
    // 다운로드한 이미지를 이미지뷰에 표시(UI업데이트)
    self.imageView.image = image
  }
}
```

<br>

**1.2. 메인 큐에서 다른 큐로 보낼때 sync 메서드를 사용하면 안됩니다**

메인큐에서 다른큐로 작업을 보낼 때 동기적으로 보내면 안됩니다. 항상 비동기적으로 보내야합니다.

지난 글에서 메인큐는 Serial 직렬이라고 언급했었죠? 시리얼큐에서 동기적으로 작업을 다른큐로 보내게되면 작업의 흐름이 이렇게 됩니다.

메인 스레드는 즉각적으로 반응해야하는 UI 관련 작업을 수행하고 있는데 동기적으로(sync로) 작업을 다른 큐에 보내버리면 메인 큐에서 다른 큐로 보낸 작업이 끝날 때까지 메인 스레드는 block 상태가 되어버립니다. **즉, UI 반응이 멈출 수 밖에 없습니다.** 그렇기 때문에 메인큐에서 다른큐로 작업을 보낼 때 Sync를 사용하면 안됩니다. 

<br><br>

#### <span style="color:orange">2. 현재의 큐에서 현재의 큐로 작업을 보낼 때 주의할 점</span>

현재 큐에서 현재 큐로 동기적(sync)으로 작업을 보내면 안됩니다. 만약 그럴 경우 **현재의 큐를 block하는 동시에 다시 현재의 큐에 접근하기 때문에 [교착상태(Deadlock)](https://en.wikipedia.org/wiki/Deadlock)(간단히 말해, 작업이 진행이 안되는 상태)이 발생할 수 있습니다.**

```swift
// 이러면 안됩니다! 예시
// 글로벌큐 사용. 글로벌큐의 기본 설정은 동시(Concurrent).
DispatchQueue.global().async{ // (B) 다시 Thread2로 작업을 보냄
  DispatchQueue.global().sync{ // (A) sync로 Queue에 작업을 보냄.
    // 작업(Task)
  }
}
```

Thread2에서 sync로 Queue에 작업을 보낸 상황(A)을 가정해봅시다. 이 상황에서 동기(sync)로 작업을 보냈기 때문에 그 작업이 끝날 때 까지 Thread2에서는 기다리고 있습니다. 즉, Thread2는 block 상태입니다. 하지만 같은 큐에서 같은 큐로 비동기적(async)으로 보냈기 때문에 큐는 다시 Thread2로 작업을 보냅니다(B). **그럼 Thread2 입장에서는 먼저 보낸 작업이 끝날 때까지 block하고 있었는데 큐가 다시 Thread2에 접근해 작업을 보내려고 하니까 deadlock이 발생할 수 있습니다.**

이 때, 글로벌큐는 기본 설정이 동시(concurrent)이기 때문에 (B)에서 Thread2가 아닌 다른 스레드(예를 들어 Thread3,4,5...)로 작업을 배치할 수 있습니다. 이럴 경우에는 deadlock이 발생하지 않습니다. 하지만 앞서 언급했듯이 **어떤 스레드에 어떻게 분배가 될지는 OS(시스템)에서 결정을 하기 때문에 deadlock을 야기할 수 있는 가능성이 얼마든지 있으므로 하면 안됩니다.**

<br><br>

## <span style="color: #6666FF">참고</span>

1. [DispatchQueue](https://developer.apple.com/documentation/dispatch/dispatchqueue)
2. [iOS Concurrency(동시성) 프로그래밍, 동기 비동기 처리 그리고 GCD/Operation](https://www.inflearn.com/course/iOS-Concurrency-GCD-Operation)
3. [Concurrency Programming Guide - Concurrency and Application Design](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ConcurrencyandApplicationDesign/ConcurrencyandApplicationDesign.html#//apple_ref/doc/uid/TP40008091-CH100-SW1)

<br>

**피드백 주신 JK 감사합니다 🙏🏻**