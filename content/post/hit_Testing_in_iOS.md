+++

authors = [
    "Lena"
]
title = "Hit-Testing in iOS "
date = 2021-01-14T21:41:58+09:00
description = "explain how to find UIView object to handle events with reverse pre-order depth-first traverse algorithm and Hit-testing"
tags = [
    "iOS", "UITouch", "UIResponder", "Dealing with Events", "Hit-testing", "reverse pre-order depth-first traverse algorithm"
]
categories = [
     "iOS", "View", "Event", "Deep Dive"
]
series = ["View in iOS"]
images = [
  "/images/hit-test-depth-first-traversal.png"
]
draft = false

+++

 <br>
 1. Hit-testing 에 대해 설명합니다.  
  2. Hit-testing 가 어떻게 이벤트를 처리할 UIView 객체를 찾는지 그 과정과 로직에 대해 설명합니다.

<br>

<!--more-->

안녕하세요! 오늘은 이벤트를 처리할 UIView 객체를 찾는 과정에 대해 알아보겠습니다. 

사실 제가 모바일 개발에 재미를 느낀 이유 중 하나가 모바일이 사람과 디바이스를 잇는 가장 가까운 사용자 인터페이스가 아닐까? 그러면 모바일 개발은 사람들의 삶의 가장 가까운 곳에서 편리함을 줄 수 있는게 아닐까? 싶어서 매력을 느꼈던 거거든요. 뭔가 사람과 기계가 소통한다는 느낌같아서 귀엽기도 하고요 ㅎ 그래서 사용자의 터치를 받은 순간부터 터치를 받을 View를 지정하는 것까지의 내용을 다루는 오늘 주제가 많이 흥미로웠습니다 ㅎㅎ 

사족이 길었네요 그럼 이제 본격적으로 시작하겠습니다! 🤗

아래 글은 [Hit-Testing in iOS](http://smnh.me/hit-testing-in-ios/) 참고하여 제 나름대로 해석하고 설명한 글입니다!

##    <  📑 목차  >

* Responder
* Hit-testing이란?
* Hit-testing은 언제 실행될까요?
* Event에 반응할 UIView 객체는 어떻게 찾을까요?
  * 뷰 계층 탐색 알고리즘 (view hierachy searching algorithm)
  * View Stack, View들의 관계 그리고 가장 깊은 view
  * Hit-test  프로세스

<br><br>



## <span style="color: #6666FF">Responder</span>



<img src="https://docs-assets.developer.apple.com/published/7c21d852b9/f17df5bc-d80b-4e17-81cf-4277b1e0f6e4.png" alt="A flow diagram: On the left, a sample app contains a label (UILabel), a text field for the user to input text (UITextField), and a button (UIButton) to  press after entering text in the field. On the right, the flow diagram shows how, after the user pressed the button, the event moves through the responder chain—from UIView, to UIViewController, to UIWindow, UIApplication, and finally to UIApplicationDelegate." style="zoom: 50%;" />

(이미치 출처: [Apple Developer Documentation](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events))

HitTest에 대해 보기 전에 UIResponder에 대한 내용은 [Using Responders and the Responder Chain to Handle Events](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events) 를 먼저 참고하여 responder 객체를 사용해서 events를 받고 처리하는 과정을 먼저 이해하면 좋겠습니다 : )

UIResponder에 대한 공식 문서를 보다가 이런 Note를 발견했는데요! 여기에서도 `hitTest(_:with:)` 를 언급하고 있더라구요! 

<img src="https://i.imgur.com/wHinduq.png" style="zoom:67%;" />



## <span style="color: #6666FF">Hit-testing이란?</span>

터치 이벤트를 받으면 어떤 일이 일어날까요? 우선 그 이벤트를 수신하고 이에 반응할 객체에 이벤트가 전달이 되어야 하겠죠? Hit-testing는 포인트(터치 포인트 등)가 화면에 그려진 그래픽 객체(UIView 등)와 만나는지(intersect, 교차하다, 가로지르다) 여부를 결정하는 프로세스입니다. iOS는 터치 이벤트를 수신해야 하는 사용자의 손가락 아래 가장 앞쪽 UIView 를 결정하기 위해 Hit-testing를 사용합니다. (여기서 최상단이란 View Stack에서의 최상단을 뜻합니다.)<br>

즉, 간단하게 말하면 터치 이벤트가 발생한 뷰를 찾는 것인데요. 좀 더 자세히 말하자면 터치 이벤트가 발생한 포인트에 있는 뷰들 중 최상단 뷰를 찾는 것입니다. 보통 그렇게 찾은 뷰가 그 이벤트를 처리할 수 있는 First Responder가 되죠.



## <span style="color: #6666FF">Hit-testing은 언제 실행될까요?</span>

<img src="http://d33wubrfki0l68.cloudfront.net/1dc3a853f81e9c3addb07de969b95d973d078b3f/0e4b2/images/hit-test-touch-event-flow.png" alt="Touch event flow" style="zoom:60%;" />

위 다이어그램에 가장 마지막

Hit-testing가 완료되고 터치 포인트 아래의 맨 앞(frontmost) view가 결정되면 Hit-test view는 터치 이벤트 시퀀스의 모든 단계(began, moved, ended, canceled 등)에 대해 UITouch 객체와 연결됩니다. 히트 테스트 보기 외에도 해당 보기 또는 상위 보기에 연결된 모든 제스처 인식기가 UITouch 개체와 연결됩니다. 그런 다음, Hit-test view는 터치 이벤트 시퀀스를 수신(receive)하기 시작합니다.

참고로 손가락이 Hit-test view의 bound 밖으로 다른 view로 이동하더라도 hit-test view는 터치 이벤트 스퀀스가 끝날 때까지 계속 모든 터치를 수신합니다. 쉽게 말해서 ViewM 위치에서 터치를 시작하고 손가락을 떼지 않고 움직여서 하트💜 를 그리고 ViewM이 터치 이벤트를 받는 UIView 객체일 때, 비록 ViewM의 손가락이 위치에서 벗어났지만 ViewM은 계속 그 터치 이벤트를 받는다는 의미입니다!

> *“The touch object is associated with its hit-test view for its lifetime, even if the touch later moves outside the view.”*
> [Event Handling Guide for iOS, iOS Developer Library](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW4)

## <span style="color: #6666FF">Event에 반응할 UIView 객체는 어떻게 찾을까요?</span>

#### <span style="color:orange">**뷰 계층 탐색 알고리즘 (view hierachy searching algorithm)**</span>

> It implements it by searching the view hierarchy using reverse pre-order depth-first traversal algorithm.
>
> ...
>
> ... hit-testing uses depth-first traversal in reverse pre-order.

hit-testing은 reverse pre-order depth-first traversal algorithm를 사용해서 뷰 계층을 탐색함으로써 시행합니다. 이게 뭔지 자세히는 모르겠지만 군옥수수수님 블로그를 보니까 직역하면 역순 [깊이 우선](https://www.geeksforgeeks.org/depth-first-search-or-dfs-for-a-graph/) 순회 알고리즘이라고 하네요! 아래 gif 에서 왼쪽 DFS가 깊이 우선 팀섹 알고리즘이고 BFS가 넓이 우선 탐색 알고리즘인데 왼쪽 이미지를 참고해주세요!) 

<img src="https://miro.medium.com/max/1680/0*miG6xdyYzdvrB67S.gif" alt="Breadth-first vs Depth-first Tree Traversal in Javascript | by Kenny Hom |  Medium" style="zoom:80%;" />

(이미지 출처: miro.medium.com)

아래 이미지는 View의 계층을 표현한 것입니다. 위의 이미지와 비슷하게 생겼죠?

위에서 말한 알고리즘을 사용한 방식은 먼저 루트 노드(root node)를 방문한 다음 상위에서 하위 인덱스로 하위 트리를 순회하는 방식입니다. 이러한 종류의 순회를 통해 순회 반복 횟수를 줄이고 터치 포인트를 포함하는 첫번째 가장 깊은 하위 view(first deepest descendant view)가 발견되면 검색 프로세스를 중지할 수 있습니다. 이것은 subview가 항상 superview 앞에 렌더링되고 sibling view(형제 뷰)가 항상 하위 인덱스가 있는 sibling view 앞에 subview 배열에 렌더링 되기 때문입니다. 따라서 여러 개의 겹치는 뷰에 특정 포인트가 포함된 경우 맨 오른쪽 하위 트리에서 가장 깊은 뷰가 맨 앞 뷰가 됩니다.

> *“Visually, the content of a subview obscures all or part of the content of its parent view. Each superview stores its subviews in an ordered array and the order in that array also affects the visibility of each subview. If two sibling subviews overlap each other, the one that was added last (or was moved to the end of the subview array) appears on top of the other.”*
> [View Programming Guide for iOS, iOS Developer Library](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW24)



<br>

#### <span style="color:orange">**View Stack, View들의 관계 그리고 가장 깊은 view**</span>

앞으로 가장 깊은 view라는 표현이 종종 나올텐데요. 그럼 **가장 깊은 view라는 건 어떤 개념일까요?**

결론부터 말하자면 **view stack 의 최상단에 위치하는 뷰가 가장 깊은 뷰입니다.** view stack은 Last In First Out 이니까 가장 마지막에 추가된 view(🎉)가 view stack 에서 마지막 인덱스의 view이면서 view stack 의 최상단에 있는 view이고 동시에 가장 깊은 view입니다. 그리고 view stack 에 있는 모든 view들이 다 겹쳐있을 경우 사용자는 가장 마지막에 추가된 view(🎉)만 볼 수 있습니다. 

좀 더 풀어서 설명을 하자면,

view들은 서로 두 가지의 관계(Parent-Child 또는 Sibling)를 가질 수 있는데요. 아래 이미지를 보시면 

* Parent-Child: MainView와 ViewA / MainView와 ViewB / MainView와 ViewC / ViewA와 ViewA.1 / ViewA와 ViewA.2 / (나머지는 추측이 가능하니 생략) 
* Sibling 관계: ViewA와 ViewB, ViewC / (나머지는 추측이 가능하니 생략) 

입니다. 

Parent view 위에 Child view가 올라가기 때문에 Parent-Child 관게에 있어서 Child가 더 깊다고 할 수 있습니다. Sibling 관계에서는 인덱스로 깊이의 순서를 결정하고 가장 마지막 인덱스의 view가 가장 깊은 view가 됩니다. 즉 그렇기 때문에 가장 마지막에 추가된 view가 가장 깊다고 볼 수 있습니다. 이제 이해가 가셨나요? 👻

<img src="/Users/keunnalee/Library/Application Support/typora-user-images/image-20210130233708109.png" alt="image-20210130233708109" style="zoom:80%;" />

(참고로 위 이미지에서 왼쪽에서 오른쪽의 트리 브랜치 배열은 subview 배열의 순서를 반영합니다. 즉 ViewC가 가장 마지막에 추가된, view stack에 가장 최상단에 있고 가장 깊은 view입니다.)

#### <span style="color:orange">**Hit-test  프로세스**</span>



<img src="http://d33wubrfki0l68.cloudfront.net/60d215400d2340e2334016ea6914aef24cfe6939/d4938/images/hit-test-depth-first-traversal.png" alt="계층 깊이 우선 순회보기" style="zoom:80%;" />

depth-first traversal in reverse pre-order 적용하면 터치 포인트를 포함하는 첫 번째 가장 깊은 하위(descendant) view 가 발견되면 순회를 중지 할 수 있습니다. 순회(traversal) 알고리즘은 view 계층 구조의 root view 인 `UIWindow`에 `hitTest:withEvent:`메시지를 보내는 것으로 시작됩니다 . 이 메서드에서 반환 된 값은 터치 포인트를 포함하는 가장 앞쪽 뷰(frontmost view)입니다.

다음 순서도는 hit-test 로직을 보여줍니다.

<img src="http://d33wubrfki0l68.cloudfront.net/2321d34cb19b0a4dbefa426561c761e10672fcc7/237b8/images/hit-test-flowchart.png" alt="적중 테스트 순서도" style="zoom:70%;" />

이 `hitTest:withEvent:`메서드는 먼저 view가 터치를받을 수 있는지 확인합니다. 

```swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    if !isUserInteractionEnabled || isHidden || alpha <= 0.01 {
        return nil
 
    }
 
    if self.point(inside: point, with: event) {
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

다음과 같은 경우 view가 터치를받을 수 있습니다.

- view가 숨겨지지 않았습니다.
  `self.isHidden == false`
- view에는 사용자 상호 작용이 활성화되어 있습니다.
  `self.userInteractionEnabled == true`
- view의 알파 수준(투명도)이 0.01보다 큽니다.
  `self.alpha > 0.01`
- view에는 다음과 같은 점이 포함됩니다.
  `pointInside:withEvent: == true`



그런 다음 view가 터치를 받는 것이 허용된 경우, 위 메서드는 receiver의 subtree를 순회(traverse, 가로지르다, 횡단하다)힙니다. 어떻게요? <span style="color:orange"> ` ⭐️ hitTest:withEvent:`메시지를 마지막부터 첫번째까지 각 subview에 메시지를 보내는데, receiver의 subtree 중 하나가 `nil`값이 아닌 값을 반환 할 때까지 전송합니다. </span>  기본적으로는 subview 중 하나에 의해 반환 된 첫 번째 non `nil`  값은 터치 포인트 아래의 가장 앞쪽(frontmost) view이며 receiver가 반환합니다. 모든 receiver subviews에서 `nil`이 반환 되거나 receiver 에게 subview 가 없는 경우 receiver는 자신을 반환합니다.

반면 뷰가 터치를받을 수 없는 경우, 이 메서드는 receiver의 subtree를 전혀 탐색하지 않고 `nil`을 반환합니니다. 이렇게 함으로써 hit-test  프로세스가 뷰 계층 구조의 모든 view를 방문하지 않을 수 있습니다.



## <span style="color: #6666FF">참고</span>

1. [hitTest:withEvent:](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest?language=objc)

2. [hitTest(_:with:)](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest)

3. [Hit-Testing in iOS](http://smnh.me/hit-testing-in-ios/) <- 읽어보시면 좋을 것 같아요!

4. [iOS ) hitTest](https://zeddios.tistory.com/536)

5.  [[ios\] Hit Testing in iOS](https://baked-corn.tistory.com/128)

#### 이미지 출처: [Hit-Testing in iOS](http://smnh.me/hit-testing-in-ios/)

