+++
authors = [
    "Lena"
]
title = "**setNeedsDisplay( )** vs **layoutIfNeeded( )** vs **setNeedsLayout( )**"
date = 2021-02-12T22:58:44+09:00
description = "description"
tags = [
    "View", "iOS", ""
]
categories = [
     "iOS"
]
series = ["iOS", "View"]
images = [
  "/images/imageName.png"
]
draft = true

+++

한줄 소개글 <br>

<br>

<!--more-->



## <  📑 목차  >

<br><br>

## <span style="color: #6666FF">setNeedsDisplay()</span>

#### <span style="color:orange">**1. setNeedsDisplay()**</span>

애플 공식 문서 보기 👉🏻 [setNeedsDisplay()](https://developer.apple.com/documentation/uikit/uiview/1622437-setneedsdisplay)

```swift
/* 실제 사용 예제
myButton은 UIButton*/ 
myButton.setNeedsDisplay()
```



공식문서를 바탕으로 정리를 해보자면!

1. `setNeedsDisplay()`는 UIView의 인스턴스 메서드로, 특정 UIView의 모습을 업데이트하고 싶을 때 다음 UIView의 업데이트 주기에서 [draw(_ rect: CGRect)](https://developer.apple.com/documentation/uikit/uiview/1622529-draw)메서드를 통해 뷰를 다시 그려줘야 함을 시스템에 알릴 수 있습니다.

2. `setNeedsDisplay()`는 요청을 기록하고 즉시 반환합니다. 
3. view는 무효화된 모든 view가 업데이트되는 다음 드로잉 사이클(drawing cycle)까지 실제로 다시 그려지지 않습니다.
4. view의 내용이나 모양이 변경될 때만 View를 다시 그리도록 요청해야 합니다. 만약 단순히 view의 geometry를 바꾼다면, view는 일반적으로 다시 그리지 않습니다. 대신, 기존의 콘텐츠는 view의 ContentMode 프로퍼티의 값에 기초하여 조정됩니다. 기존 콘텐츠를 다시 표시하면 변경되지 않은 콘텐츠를 다시 그리는 것을 피하여 성능을 향상시킵니다.
5. 만약 [`CAEAGLLayer`](https://developer.apple.com/documentation/quartzcore/caeagllayer) 객체의 도움을 받는(사용하여 만든) view라면, 이 메서드는 효과가 없습니다. UIKit이나 Core Graphics 처럼 네이티브 드로잉 기술을 사용한 view들을 대상으로 합니다.

더 간단하게 정리하자면,

1️⃣ UIView의 내용을 다시 그리고 싶을 때(업데이트 하고 싶을 때), `setNeedsDisplay()`를 호출합니다.

2️⃣ view가 업데이트 되는 다음 드로잉 사이클에서 UIView가 업데이트 되도록 시스템에 알려주게 됩니다.

3️⃣ 다음 주기에서 UIView가  `draw(_ rect: CGRect)`를 호출하면서 view가 새롭게 업데이트 됩니다.

4️⃣ 따라서 `setNeedsDisplay()`를 호출했다고 바로 view가 업데이트 되는 것은 아닙니다. 다음 업데이트 주기에 되는 것이지요!

<span style="color:orange">**2. 그럼 draw(_ rect: CGRect)는 언제 불릴까요?**</span>



그럼 이쯤에서 궁금한게 생깁니다. draw(_ rect: CGRect)는 언제 불리는지 말이에요!

이 메서드는 view가 처음 생성될 때 이 메서드를 사용해서 view가 그려지게 됩니다.

`UIView`의 `func draw(_ rect: CGRect)`는 **기본적으로 아무 처리도 하지 않기 때문**에, `UIView`를 상속받은 custom view의 경우에는 `super.rect(rect)` 를 **부르지 않아도 됩니다.**

하지만 `UILabel`, `UITextView` 등 `UIView`를 상속받은 view를 상속받은 custom view의 `func draw(_ rect: CGRect)`에서는 super를 불러주어야 합니다.



<span style="color:orange">**3. setNeedsDisplay()는 언제 사용할까요?**</span>

### 그럼 언제 사용할까요?

1. `setNeedsDisplay`

- 뷰가 처음 생성될 때 레이아웃이 구성되는 사이클을 처음부터 실행시키는 메소드입니다.
- 뷰에 추가된 모든 subView를 전부 새로 그릴 필요가 있을 때 사용되는 메소드입니다.
- 모든 superView의 모든 subView를 바로 새로 그리고 싶다👉🏻 `setNeedsDisplay()`



## <span style="color: #6666FF">layoutIfNeeded()</span>

#### <span style="color:orange">**1. layoutIfNeeded()**</span>

애플 공식 문서 보기 👉🏻  [layoutIfNeeded()](https://developer.apple.com/documentation/uikit/uiview/1622507-layoutifneeded)

```swift
/* 실제 사용 예제
myButton은 UIButton*/ 
myButton.setNeedsDisplay()
```

공식문서를 바탕으로 정리를 해보자면!

이 방법을 사용하여 뷰가 레이아웃을 즉시 업데이트하도록 합니다. Auto Layout(자동 레이아웃)을 사용하는 경우 레이아웃 엔진은 필요에 따라 뷰의 위치를 업데이트하여 제약 조건의 변경 사항을 충족합니다. 메시지를 루트 보기로 수신하는 보기를 사용하여 이 메서드는 루트에서 시작하는 보기 하위 트리를 배치합니다. 보류 중인 레이아웃 업데이트가 없으면 레이아웃을 수정하거나 레이아웃 관련 콜백을 호출하지 않고 이 방법을 종료합니다.

<img src="image url" width="50%">

## <span style="color: #6666FF">setNeedsLayout()</span>

#### <span style="color:orange">**1. setNeedsLayout()**</span>

<br><br>

## <span style="color: #6666FF">참고</span>

1. [iOS / drawRect 와 setNeedsDisplay](https://unnnyong.me/2019/05/29/ios-%F0%9F%87%B0%F0%9F%87%B7-drawrect-%EC%99%80-setneedsdisplay/)

<br>

