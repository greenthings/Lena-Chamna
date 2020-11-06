+++
authors = [
    "Lena"
]
title = "[iOS] AppDelegate와 SceneDelegate"
date = 2020-03-06T23:26:10+09:00
description = "iOS 13 이후 AppDelegate와 SceneDelegate"
tags = [
    "iOS13", "AppDelegate", "SceneDelegate"
]
images = [
  "/images/iOS13_AppDelegate_and_SceneDelegate.png"
]
draft = false
+++

AppDelegate와 SceneDelegate에 대해서 알아봅시다 😁 <br><br>

<!--more-->

##    <  📑 목차  >
* [AppDelegate와 SceneDelegate](#AppDelegate와-SceneDelegate)
* [사전 지식 (용어)](#사전-지식-(용어))
* [iOS13부터 바뀐 점](#iOS13부터-바뀐점)
* [Scene?](#Scene?)
* [Scene Session?](#Scene-Session?)
* [iOS13부터 AppDelegate가 하는 일?](#iOS13부터-AppDelegate가-하는-일?)
* [Deployment Target이 iOS 13 미만인 상황에서는?](#Deployment-Target이-iOS-13-미만인-상황에서는?)
* [참고](#참고)
  

<br>

<img src = "https://i.imgur.com/GlXvqQQ.png" width = "70%">  
<br>

Xcode에서 프로젝트를 생성하면 자동으로 추가되어있는 swift 파일 중에 `AppDelegate.swift`와 `SceneDelegate.swift` 파일이 있어요!
오늘은 이 두 swift 파일에 있는 AppDelegate클래스와 SceneDelegate클래스에 대해서 알아보려고 합니다 😁 
<br><br>

# <span style="color: #6666FF">AppDelegate와 SceneDelegate</span>

## 사전 지식 (용어)

1.`객체 (Object)`: 클래스의 인스턴스.<br>
2.`인스턴스 (Instance)`: 클래스(즉, 객체)나 구조체, 열거형의 구체적인/특정한 발생(A specific occurrence)<br>
3.`클래스 (Class)`: 특정한 타입의 객체에 공통되는 동작과 속성을 묘사한 코드 조각으로 본질적으로 객체의 청사진을 제공.<br> 
(Start Developing iOS Apps (Swift)- iOS and Swift Terminology Glossary 참고)
<br><br>
<br><br>

## <span style="color: #6666FF">iOS13부터 바뀐점</span>

* 바뀌기 이전(~iOS12)<br>
<img src = "https://i.imgur.com/CDwg5Ny.png" width = "60%">

* 바뀐 후 (iOS 13)<br>
<img src = "https://i.imgur.com/D4VfgIv.png" width = "60%">

<br>(사진은 wwdc 2019 영상에서 가져왔습니다)

**1. iOS12까지는 대부분 하나의 앱에 하나의 `window`였지만 iOS 13부터는 window의 개념이 `scene`으로 대체되고 아래의 사진처럼 하나의 앱에서 여러개의 scene을 가질 수 있습니다.**  
<img src = "https://i.imgur.com/sK5PvQo.jpg" width = "40%">

**2. AppDelegate의 역할 중 UI의 상태를 알 수 있는 UILifeCycle에 대한 부분을 `SceneDelegate`가 하게 됐습니다.**  
<img src = "https://i.imgur.com/tGlsHON.png" width = "40%">

**3. 그리고 AppDelegate에 `Session Lifecycle`에 대한 역할이 추가됐습니다.**  
<img src = "https://i.imgur.com/enmQrnB.png" width = "40%">


Scene Session이 생성되거나 삭제될 때 AppDelegate에 알리는 두 메소드가 추가됐습니다.
Scene Session은 앱에서 생성한 모든 scene의 정보를 관리합니다.
<br><br>
<br><br>

**<span style="color: orange">iOS 13부터는 window의 개념이 `scene`으로 대체됐다고 하는데요! 그럼 Scene은 뭘까요?</span>** 

## <span style="color: #6666FF">Scene? </span>

>UIKit는 UIWindowScene 객체를 사용하는 앱 UI의 각 인스턴스를 관리합니다. **Scene에는 UI의 하나의 인스턴스를 나타내는 windows와 view controllers가 들어있습니다.** 또한 각 **scene에 해당하는 UIWindowSceneDelegate 객체**를 가지고 있고, 이 객체는 **UIKit와 앱 간의 상호 작용을 조정**하는 데 사용합니다. Scene들은 같은 메모리와 앱 프로세스 공간을 공유하면서 서로 동시에 실행됩니다. **결과적으로 하나의 앱은 여러 scene과 scene delegate 객체를 동시에 활성화**할 수 있습니다.  
(Scenes - Apple Developer Document 참고)

<br>
<br><br>

**<span style="color: orange">UI의 상태를 알 수 있는 UILifeCycle에 대한 역할을 `SceneDelegate`가 하게 됐죠! 역할이 분리된 대신 `AppDelegate`에서 Scene Session을 통해서 scene에 대한 정보를 업데이트 받는데요! 그럼 Scene Session은 뭘까요??</span>**

## <span style="color: #6666FF">Scene Session?</span>


>**UISceneSession 객체**는 scene의 고유의 런타임 인스턴스를 관리합니다. 사용자가 앱에 새로운 scene을 추가하거나 프로그래밍적으로 scene을 요청하면, 시스탬은 그 scene을 추적하는 session 객체를 생성합니다. **그 session에는 고유한 식별자와 scene의 구성 세부사항(configuration details)가 들어있습니다.** UIKit는 session 정보를 그 scene 자체의 생애(life time)동안 유지하고 app switcher에서 사용자가 그 scene을 클로징하는 것에 대응하여 그 session을 파괴합니다. session 객체는 직접 생성하지않고 UIKit가 앱의 사용자 인터페이스에 대응하여 생성합니다. 또한 위 3번에서 소개한 두 메소드를 통해서 UIKit에 새로운 scene과 session을 프로그래밍적 방식으로 생성할 수 있습니다.  
(UISceneSession - Apple Developer Document 참고)

<br>
<br><br>

**<span style="color: orange">그럼 SceneDelegate가 추가된 iOS13에서 AppDelegate는 어떤 일을 할까요??</span>**

## <span style="color: #6666FF">iOS13부터 AppDelegate가 하는 일?</span>

> 이전에는 앱이 foreground에 들어가거나 background로 이동할 때 앱의 상태를 업데이트하는 등의 앱의 주요 생명 주기 이벤트를 관리했었지만 더이상 하지 않습니다.

<br>
현재 하는 일은 5가지 입니다.<br>

1. 앱의 가장 중요한 데이터 구조를 초기화하는 것
2. 앱의 scene을 환경설정(Configuration)하는 것
3. 앱 밖에서 발생한 알림(배터리 부족, 다운로드 완료 등)에 대응하는 것
4. 특정한 scenes, views, view controllers에 한정되지 않고 앱 자체를 타겟하는 이벤트에 대응하는 것. 
5. 애플 푸쉬 알림 서브스와 같이 실행시 요구되는 모든 서비스를 등록하는것.

(UIApplicationDelegate - Apple Developer Document 참고)
 <br><br><br>
 <img src = "https://i.imgur.com/NYKSNdl.png" width = "80%"><br>  

AppDelegate 클래스는 새로운 프로젝트를 생성할 때마다 자동으로 생성됩니다. 저는 이 AppDelegate 클래스가 채택하고 있는 UIApplicationDelegate에 들어가 봤는데요. 위 사진과 같은 메소드들이 있었습니다. 
보통의 경우, 앱을 초기화하고 앱레벨의 이벤트에 반응하기 위해서는 Xcode가 제공하는 이 클래스를 사용해야 합니다. UIApplicationDelegate 프로토콜은 앱을 설정하고, 앱의 상태 변화에 대응하며, 다른 앱 레벨의 이벤트를 처리하는 데 사용하는 여러 가지 메소드를 정의합니다.
<br><br>



## <span style="color: #6666FF">Deployment Target이 iOS 13 미만인 상황에서는?</span>
**Deployment Target이 iOS 13 미만인 상황에서도 UIScene과 UISceneDelegate를 사용할 수 있을까요? 만약 앱이 iOS 13 미만의 버전도 지원해야 한다면 어떻게 해야할까요?**

<br>

iOS 12 이하는 하나의 앱에 하나의 window를 가지고 있기 때문에(즉, multi window를 사용하지 않기 때문에) iOS 13에서 추가된 부분을 삭제하고 이전 버전(~iOS12)과 설정을 똑같이 바꿔주면 이전 방식대로과 동일하게 할 수 있습니다.<br><br>
방법은 Xcode를 새로 열고<br><br>

[ step 1 ] iOS13에서 새로 생긴 `SceneDelegate.swift` 파일 삭제

[ step 2 ]iOS13에서 `AppDelegate`에 추가된  `UISceneSession`과 관련된 두 메소드 삭제

<img src = "https://i.imgur.com/5JCf9LM.png" width = "90%">

[ step 3 ] iOS13에서 `SceneDelegate`로 옮겨진 `window` 프로퍼티를 AppDelegate로 다시 옮기기

```swift
      var window: UIWindow?
```

[ step 4  ]`info.plist`에서 Scene과 관련된 Manifest인 `Application Scene Manifest `삭제
<br><br>

직접 해봤는데 Build Success가 뜨네요! iOS13 미만인 경우 이렇게 하면 될 것 같습니다.

<br>

<br>

## 참고

1. [DevelopiOSAppsSwift - BuildABasicUI](https://developer.apple.com/library/archive/referencelibrary/GettingStarted/DevelopiOSAppsSwift/BuildABasicUI.html#//apple_ref/doc/uid/TP40015214-CH5-SW1)
2. [iOS ) AppDelegate.swift의 역할](https://zeddios.tistory.com/218) - zedd님 블로그
3. [Architecting Your App for Multiple Windows](https://developer.apple.com/videos/play/wwdc2019/258/) - wwdc 2019
4. [SceneDelegate (1) - Architecting Your App for Multiple Windows](https://zeddios.tistory.com/811) - zedd님 블로그
5. [SceneDelegate와 AppDelegate part 1 [번역]](https://usinuniverse.bitbucket.io/blog/scenedelegatepart1.html)
6. [Scenes - Apple Developer Document](https://developer.apple.com/documentation/uikit/app_and_environment/scenes)
7. [UIscenesession - Apple Developer Document](https://developer.apple.com/documentation/uikit/uiscenesession)
8. [UIApplicationDelegate - Apple Developer Document](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)
<br><br>

**도움 주신 분**  

야곰  

**감사합니다🙏**

<br><br>

> 애플 공식문서를 번역해봤는데 오역이 있을 수 있습니다 😢 
**궁금한 점, 틀린 내용, 오타 지적, 오역 지적, 칭찬, 격려 등 모두 환영합니다! 댓글로 남겨주세요!** 
읽어주셔서 감사합니다! 😊 🙏 