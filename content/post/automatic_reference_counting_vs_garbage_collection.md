+++
authors = [
    "Lena"
]
title = "Swift의 Automatic Reference Counting(ARC) vs Java의 Garbage Collection(GC)"
date = 2020-11-15T23:24:49+09:00
description = "Swift의 Automatic Reference Counting(ARC) vs Java의 Garbage Collection(GC)"
tags = [
    "Memory", "ARC", "메모리 관리 기법", "MRC", "MRR", "GC", "Swift", "Objective-C", "Java"
]
categories = [
     "Swift", "Memory", "Deep Dive", "Effective Swift"
]
series = ["Memory", "Effective Swift"]
images = [
  "/images/ARC_Illustration.jpg"
]
draft = false

+++

1️⃣Java의 Garbage Collection(GC)와 2️⃣Swift의 Automatic Reference Counting(ARC) 3️⃣MRR(Manual Retain-Release) or MRC(Manual Reference Counting)에 대해 소개합니다 :)

<br>

<!--more-->

2020년 10월인가 11월부터 이펙티브 자바라는 책으로 스터디를 하고 있어요!

<img src="https://i.imgur.com/OpygiIv.png" style="zoom:33%;" />

👉🏻👉🏻 [Effective Swift](https://theswiftists.github.io/effective-swift/)

Swift라는 언어가 다른 프로그래밍 언어에 비해서 업데이트가 빨라 바뀌는게 많아서 그런지 Swift는 Effective 시리즈가 없더라구요! 그래서 Effective 시리즈 중 많은 프로그래머들에게 인정받고 있는 Effective Java 책 기반으로 Effective Swift를 해보면 어떨까? 라는 생각에 친구들과 함께 공부하고 있습니다. 

내용이 쉽지 않지만 그래도 재밌어요! 🤓

아래는 제가 작성한 스터디 문서를 토대로 작성했습니다!

[Item 7. 다 쓴 객체 참조를 해제하라](https://github.com/TheSwiftists/effective-swift/blob/main/2%EC%9E%A5_%EA%B0%9D%EC%B2%B4_%EC%83%9D%EC%84%B1%EA%B3%BC_%ED%8C%8C%EA%B4%B4/item7.md)

## <  📑 목차  >

1. Java의 Garbage Collection(GC)<br>
   1.1. 동작방식<br>
   1.2. 장단점<br>

2. MRR(Manual Retain-Release) or MRC(Manual Reference Counting) <br>

3. Swift의 Automatic Reference Counting(ARC) <br>
   3.1. 동작방식<br>
   3.2. 레퍼런스 카운팅 시점<br>
   3.3. 장단점<br>

4. 메모리 누수 (Memory Leak)<br>
   4.1. 메모리 누수는 언제 일어날까?<br>
   4.2. 강한 참조(Strong Reference)와 강한 참조 순환(Reference Cycle) 예시<br>
   4.3. 해결 방법<br>
   4.4. Weak 참조(약한 참조)와 Unowned 참조(미소유 참조)<br>
   4.5. Strong 참조, Weak 참조, Unowned 참조 비교표<br>

5. 참고

   <br><br>

## <span style="color: #6666FF">Java의 Garbage Collection(GC)</span>

<img src="https://t1.daumcdn.net/cfile/tistory/275F4333578E05A021" alt="img" style="zoom:75%;" />

<p align="center">이미지 출처: Naver D2</p>

<span style="color:orange">**1. 동작 방식**</span>

* Java의 경우 JVM에 의해 **힙(heap) 영역에 생성 객체가 할당**됩니다.
* JVM은 객체들이 **더이상 코드에 참조되지 않을 때를 추적**합니다.
* GC는 **Unreachable Objects들을 수거**합니다.
* Unreachable Objects: 유효한 최초의 참조(Root Set of References가 이루어지지 않는 객체)

~~~java
class A { 
  int i = 5; 
}
class B { 
  A a = new A(); 
}
class C {
   B b;
   public static void main(String args[]) {
      C c = new C();
      c.b = new B();
      // instance of A, B, and C created
      c.b = null;
      // instance of B and A eligible to be garbage collected.
}
~~~

* **런타임**시 **백그라운드**에서 사용되지 않는 객체와 객체 그래프들(objects and object graphs)을 감지하는 방식으로 동작합니다.
  이 동작은 일정 시간 경과한 뒤나 런타임 메모리가 낮아졌을 때 중간 간격(intermediate intervals)으로 발생하며 <u>**정확한 순간에 해제되지 않습니다.**</u>
* GC는 **사용자가 강제로 수행할 수 없고 언제 일어나는지도 불확실합니다.**
* GC는 객체를 메모리에서 제거하기 전에 해당 객체의 `finalize( )` 메소드 호출합니다. 

<span style="color:orange">**2. 장단점**</span>

* **Garbage Collection의 장점**
  1. Retain Cycles을 포함한 전체 객체 그래프를 정리(clean up)할 수 있습니다.
* **Garbage Collection의 단점**
  1. GC는 **객체 릴리즈(objects release)에 대한 정확한 시간이 결정되지 않습니다**(undetermined).
  2. Full GC가 발생하면 애플리케이션의 다른 스레드가 일시적으로 보류될 수 있습니다. 
     - STW (Stop-The-World): Full GC가 발생하면 JVM은 애플리캐이션 실행을 멈추고 GC를 실행하는 쓰레드만 작동합니다. 

<br>

## <span style="color: #6666FF">Objective-C의 MRC(Manual Reference Counting)</span>

Objective-C는 RC 방법으로 메모리를 관리해 왔습니다. 개발자가 메모리 할당 및 해제를 직접 관리해야 했기 때문에 **MRC(Manual Reference Counting)**, 또는 할당(`retain`)과 해제(`release`)를 명시적으로 호출하기 때문에 **MRR(Manual Retain-Release)** 이라는 이름으로 불렸습니다.

<img src="https://blog.kakaocdn.net/dn/bin8O7/btqFIjeJxHu/RgkMxKEn59q35DWrMhm2v1/img.png" alt="../Art/ARC_Illustration.jpg" style="zoom:20%;" />

   ARC가 나오기 전에는 개발자가 직접 메모리 관리에 신경을 써야 했습니다. 메모리 관리를 직접 한다는 것은 인스턴스를 메모리에 할당하고 필요하지 않을 때 메모리에서 해제시키는 코드를 적절하게 사용해야 한다는 의미입니다. ARC 이전에는 Retain Cycle을 추적하여 인스턴스를 retain 한 만큼 명시적으로 release 시켜 주어야 했습니다. 

 2011년부터 Objective-C의 메모리 관리 방식은 MRC에서 ARC로 대체되었고, 2014년 발표된 Swift도 ARC를 사용한 메모리 관리 방식을 채택했습니다. ARC는 개발자가 직접 작성해야 했던 `retain`과 `release` 코드를 complie할 때 생성하여 참조를 관리합니다.

   > 기본적으로 `retain`과 `release`는 메모리의 heap 영역에 할당/해제하는 작업이기 때문에 ARC도 class instance 및 function, closure 같은 참조 타입에만 적용됩니다.

## <span style="color: #6666FF">Swift의 Automatic Reference Counting(ARC) </span>
: 객체의 레퍼런스 카운팅(reference counting, 참조 수)을 제공하는 메모리 관리 기법입니다.

<img src="https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Art/ARC_Illustration.jpg" alt="../Art/ARC_Illustration.jpg" style="zoom:75%;" />

<p align="center">이미지 출처: Apple Documentation Archive</p>


<span style="color:orange">**1. 동작 방식**</span>
   * 레퍼런스 카운팅(Reference Counting)은 **각 객체의 레퍼런스(= 참조)의 수를 계산하는 방식**으로 동작합니다.
   * **런타임에 레퍼런스 카운트를 증가시키거나 감소시키는 메모리 참조나 해제 코드(retain이나 release)를 컴파일 때 컴파일러가 일정한 규칙에 의해 생성해 삽입하고(위 이미지 참고), 객체의 레퍼런스 카운트가 0에 도달했을 때 객체의 할당 해제를 표시합니다.**
   * **개발자는 ARC가 언제 참조 카운트를 올리고 내리는지 규칙을 알고, 적절한 시점에 코드가 생성되도록 조절**할 수 있습니다.
   * **레퍼런스 카운트가 0이 되면 그 객체는 확실히 접근할 수 없습니다(unreachable)**.
   * **런타임에 [비동기적](https://stackoverflow.com/a/748235)**으로 객체를 제거합니다. 

<span style="color:orange">**2. 레퍼런스 카운팅 시점**</span>

- **레퍼런스 카운트가 오르는 경우**<br>
  레퍼런스 카운트는 클래스 인스턴스(Class Instance) 등의 레퍼런스 타입 값을 변수에 할당할 때 증가합니다.   
  클래스 인스턴스를 변수에 할당할 때 메모리에서는 **힙(heap) 영역에 저장된 instance meta data의 주소값을 변수에 할당**하는 작업이 일어납니다.   
즉, 메모리의 **<u>힙(heap) 영역에 저장되어 있는 instance meta data의 주소값이 변수에 할당되는 시점이 참조 카운트가 증가하는 시점</u>**입니다.
  
<img src="https://images.velog.io/images/cskim/post/8572c476-6691-4f93-b72e-4f43cce35dac/image.png" alt="img" style="zoom:20%;" />
  
  <p align="center">이미지 출처: https://velog.io/@cskim</p>
  
- **레퍼런스 카운트가 내려가는 경우**<br>
  레퍼런스 카운트가 감소하는 시점은 **인스턴스를 참조하던 stack 영역의 변수들이 메모리에서 해제될 때** 입니다.

<img src="https://images.velog.io/images/cskim/post/cf077539-43c2-4c24-b22f-5c03ada0874c/image.png" alt="img" style="zoom:20%;" />

<p align="center">이미지 출처: https://velog.io/@cskim</p>

<span style="color:orange">**3. 장단점**</span>

* **Automatic Reference Counting의 장점**
  1. 객체가 사용되지 않을 때 실시간으로 결정론적 파괴([deterministic destruction](https://docs.elementscompiler.com/Concepts/ARCvsGC/))가 일어납니다. <br>
     * 참고로, [결정론적 알고리즘(*deterministic* algorithm](https://ko.wikipedia.org/wiki/%EA%B2%B0%EC%A0%95%EB%A1%A0%EC%A0%81_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98))은 '*예측한 그대로 동작하는*' 알고리즘을 뜻하는데 이곳에서도 이와 같은 맥락으로 *예측 그대로 파괴된다*는 의미라고 해석할 수 있을 것 같습니다.
  2. 백그라운드 프로세싱이 없으므로 모바일 장치와 같은 저전력 시스템에서 더 효율적입니다.

* **Automatic Reference Counting의 단점**
  1. Retain Cycles에 대처할 수 없습니다.

## <span style="color: #6666FF">메모리 누수 (Memory Leak) </span>

<span style="color:orange">**1. 메모리 누수는 언제 일어날까?**</span>
서로 다른 인스턴스가 서로를 **강하게 참조**하고 있어서 참조 횟수를 0으로 만들지 못하고 영원히 메모리에서 해제되지 않는 **순환 참조** 관계일 때 메모리 누수가 발생합니다.

<span style="color:orange">**2. 강한 참조(Strong Reference)와 강한 참조 순환(Reference Cycle) 예시**</span>

```swift
class Swift {
  var swiftReference: Java?
}

class Java {
  var javaReference: Swift?
}

var swift: Swift? = Swift()      // swift Reference Count: 1
var java: Java? = Java()         // java Reference Count: 1
swift?.swiftReference = java     // java Reference Count: 2
java?.javaReference = swift      // swift Reference Count: 2
swift = nil                      // swift Reference Count: 1
java = nil                       // java Reference Count: 1
```

<span style="color:orange">**3. 해결 방법**</span>
강한 참조 순환 문제를 해결하기 위한 방법으로는 **약한 참조(weak reference)**와 **미소유 참조(unowned reference) 참조** 두 가지가 있습니다. 두 참조는 순환 참조 상황에 있는 A 인스턴스가 다른 B 인스턴스를 강하게 잡고 있지 않도록(인스턴스를 참조하지만 RC를 증가시키지 않도록) 하여 강한 참조 순환을 만들지 않도록 합니다. 

<span style="color:orange">**4. Weak 참조(약한 참조)와 Unowned 참조(미소유 참조)**</span>

* **Weak Reference(약한 참조)**<br>
  약한 참조는 해당 참조가 남아있더라도 ARC는 인스턴스를 메모리에서 해제시킬 수 있습니다.   
앞서 언급한 대로 A 인스턴스가 다른 B 인스턴스를 강하게 잡고 있지 않기 때문입니다.  
  ARC는 자동으로 약한 참조를 하는 객체에 **nil**을 할당할 수 있습니다. 이때, 런타임에 객체에 nil을 할당해야 하기 때문에 **옵셔널 타입의 변수(var)**로 선언해야 합니다.<br>

  약한 참조는 강한 참조 순환 문제를 만드는 두 인스턴스에서 **다른 한쪽의 인스턴스가 상대적으로 먼저 메모리에서 해제될 가능성이 있을 때** 사용하면 됩니다.  ( Ex - delegate )
  
  ```swift
  class Swift {
    var swiftReference: Java?
  }
  
  class Java {
    weak var javaReference: Swift? // 옵셔널 타입의 변수
  }
  
  var swift: Swift? = Swift()      // swift Reference Count: 1
  var java: Java? = Java()         // java Reference Count: 1
  swift?.swiftReference = java     // java Reference Count: 2
  java?.javaReference = swift      // swift Reference Count: 1
  swift = nil                      // swift Reference Count: 0 -> java Reference Count: 1
  java = nil                       // java Reference Count: 0

  ```
  
* **Unowned Reference(미소유 참조)**<br>
  미소유 참조도 약한 참조와 마찬가지로 레퍼런스 카운트를 증가시키지 않으면서 인스턴스를 참조합니다.   
하지만 미소유 참조는 **인스턴스를 참조하는 도중에 해당 인스턴스가 메모리에서 사라질 일이 없다고 가정**하기 때문에 약한 참조와 달리 암묵적으로 옵셔널을 해제(`!`)하여 선언합니다.<br>
  미소유 참조는 참조하고 있던 인스턴스가 먼저 메모리에서 해제될 때 `nil`을 할당할 수 없어 오류를 발생시키므로 사용시 주의해야합니다.
  
  ```swift
  class SomeClass {
    weak var weakVariable: Int?
    unowned var unownedVariable: Int!
  }
  ```

<span style="color:orange">**5. Strong 참조, Weak 참조, Unowned 참조 비교표**</span>



![](https://i.imgur.com/F2fecnQ.png)

## <span style="color: #6666FF">참고</span>

1. [Automatic Reference Counting - the swift programming language 5.3](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)
2. [ARC vs. GC](https://docs.elementscompiler.com/Concepts/ARCvsGC/)
3. [Garbage Collection vs Automatic Reference Counting](https://medium.com/computed-comparisons/garbage-collection-vs-automatic-reference-counting-a420bd4c7c81)
4. [[JAVA] Garbage Collection의 기초](https://itmining.tistory.com/24)
5. [Java Reference와 GC](https://d2.naver.com/helloworld/329631)
6. [ARC](https://velog.io/@cskim/ARCAutomatic-Reference-Counting)

**이미지 출처**

- Java의 Garbage Collection(GC): [Naver D2 - Java Reference와 GC](https://d2.naver.com/helloworld/329631)
- Swift의 ARC 이미지: [Transitioning to ARC Release Notes](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)
- Swift의 ARC 레퍼런스 카운팅 시점 이미지: [ARC](https://velog.io/@cskim/ARCAutomatic-Reference-Counting)

### Strong 참조, Weak 참조, Unowned 참조 비교표 출처

* [[Swift] 메모리 관리와 ARC](https://velog.io/@cskim/ARCAutomatic-Reference-Counting)