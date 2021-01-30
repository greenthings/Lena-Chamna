+++
authors = [
    "Lena"
]
title = "Hit test 사용하기"
date = 2021-01-15 T00:51:55+09:00
description = "give examples of practical use of hit test"
tags = [
    "iOS", "Dealing with Events", "Hit-testing"
]
categories = [
    "iOS", "View", "Event", "Deep Dive"
]
series = ["View in iOS"]
images = [
  "/images/hit-test-increase-touch-area.png"
]
draft = false

+++

Hit test 의 활용 예시를 소개합니다 <br>

<br>

<!--more-->

👉🏻👉🏻👉🏻 Hit-testing과 Hit-testing이 이벤트를 처리할 UIView 객체를 찾는 과정과 로직에 대한 설명은 [이전 포스팅](https://lena-chamna.netlify.app/post/hit_testing_in_ios/) 을 확인해주세요! 👈🏻👈🏻👈🏻

## <  📑 목차  >

1. **터치 면적 넓히기**
2. **터치 이벤트를 그냥 통과 시키기**

<br><br>

안녕하세요! HitTest의 사용 예시에 대해 소개하겠습니다. 아는 분이 HitTest 키워드를 알려주셔서 흥미로워서 공부해봤는데 지난 포스트와 이번 포스트 모두 재밌는것 같아요 👻  

Hit test는 그럼 언제 사용할까요? 

 A view에서 처리하려는 터치 이벤트가 해당 터치 이벤트 시퀀스의 모든 단계에 대해 B view로 리다이렉트 되어야 할 때 **`hitTest:withEvent:`** 메서드를 재정의해서 사용할 수 있습니다. 

## <span style="color: #6666FF">터치 면적 넓히기(Increasing view touch area)</span>

뷰의 터치 영역이 bound보다 커야 하는 경우 **`hitTest:withEvent:`** 메서드를 재정의해서 사용할 수 있습니다. 아래 코드는 사이즈가 20x20인 UIView의 터치 영역을 10 point씩 증가시키는 예시입니다.

![Increasing touch area](http://d33wubrfki0l68.cloudfront.net/03e4c7beab133dd5f7d70101286ed6a4205fd6da/cb502/images/hit-test-increase-touch-area.png)

```swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    if !isUserInteractionEnabled || isHidden || alpha <= 0.01 {  return nil }
 
    let touchRect = bounds.insetBy(dx: -10, dy: -10)
    if touchRect.contains(point) {
        for subview in subviews.reversed() {
            let convertedPoint = subview.convert(point, from: self)
            if let hitTestView = subview.hitTest(convertedPoint, with: event) {
                return hitTestView
            }
        }
        return self
    }
    return nil
}
```





## <span style="color: #6666FF">터치 이벤트를 아래 view로 전달하기(Passing touch events through to views below)</span>

view hierarchy 뷰 계층에서 가장 최상단에 있는 view가 터치 이벤트를 받는 것이 아니라 그 아래에 있는 view가 터치 이벤트를 받아야 하는 경우가 있습니다. 대표적인 예 중 하나는 TableView Cell 안에 있는 버튼이 있겠네요! 이럴 때 최상단 view의 custom class를 만들어서 아래와 같이 hitTest 메서드를 오버라이드 해준다면 최상단 바로 아래에 있는 view 가 터치 이벤트를 받게 할 수 있습니다.

```swift
  override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
      let hitView: UIView? = super.hitTest(point, with: event)
      if (self == hitView) { return nil }
      return hitView
  }
```



화면을 만들다보면 view를 여러 layer로 쌓아서 만들거나 뷰 계층이 복잡해지는 경우가 있는데, 이런 경우 hit test를 통해서 터치 이벤트를 받을 UIView 객체를 지정해줄 수 있는 점이 편한 것 같아요! 👍🏻



## <span style="color: #6666FF">참고</span>

1. [Hit-Testing in iOS](http://smnh.me/hit-testing-in-ios/) 

2. [iOS/swift. hitTest 이용하기](https://mrgamza.tistory.com/526)

