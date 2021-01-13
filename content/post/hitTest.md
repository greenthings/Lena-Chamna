+++

authors = [
    "Lena"
]
title = "View가 겹쳐있을 때 HitTest로 Touch Event 받기 "
date = 2021-01-14T21:41:58+09:00
description = "Uploading array of images using multipart form Data in swift"
tags = [
    "iOS", "HTTP", "multipart/form-data", "uploading images", "image", "URLSession"
]
categories = [
     "iOS", "View"
]
series = ["View in iOS"]
images = [
  "/images/hit-test-depth-first-traversal.png"
]
draft = true

+++

 <br> View들이 겹쳐있을 때 뒤(behind)에 위치한 View가 Touch Event를 받을 수 있게 하는 HitTest에 대해 소개합니다. 

<br>

<!--more-->



##    <  📑 목차  >

* Responder
* HitTest

<br><br>

## <span style="color: #6666FF">Responder</span>

저는 View 의 Evnet에 관하여 처리할 때는 보통 UIResponder를 사용하곤 했습니다. 

<img src="https://docs-assets.developer.apple.com/published/7c21d852b9/f17df5bc-d80b-4e17-81cf-4277b1e0f6e4.png" alt="A flow diagram: On the left, a sample app contains a label (UILabel), a text field for the user to input text (UITextField), and a button (UIButton) to  press after entering text in the field. On the right, the flow diagram shows how, after the user pressed the button, the event moves through the responder chain—from UIView, to UIViewController, to UIWindow, UIApplication, and finally to UIApplicationDelegate." style="zoom: 50%;" />

(이미치 출처: [Apple Developer Documentation](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events))

UIResponder에 대한 내용은 [Using Responders and the Responder Chain to Handle Events](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events) 를 먼저 참고하여 responder 객체를 사용해서 events를 받고 처리하는 과정을 먼저 이해하면 좋겠습니다 : )

위 공식 문서를 보다가 이런 Note를 발견했는데요!

<img src="/Users/keunnalee/Library/Application Support/typora-user-images/image-20210113223407709.png" alt="image-20210113223407709" style="zoom:90%;" />

바로 HitTest입니다.

#### <span style="color:orange">**HitTest**</span>

#### 참고

1. [hitTest:withEvent:](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest?language=objc)

2. [hitTest(_:with:)](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest)

3. [Hit-Testing in iOS](http://smnh.me/hit-testing-in-ios/)

4. [iOS ) hitTest](https://zeddios.tistory.com/536)

#### 이미지 출처: [Hit-Testing in iOS](http://smnh.me/hit-testing-in-ios/)

