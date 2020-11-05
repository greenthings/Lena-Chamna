+++

authors = [
    "Lena"
]
title = "간단한 예제로 살펴보는 iOS Design/Architecture Pattern: Coordinator - Basic"
date = 2020-11-05T14:34:32+09:00
description = "Coordinator Design/Architecture Pattern with UIKit - Basic"
tags = [
    "Design Pattern", "Architecture Pattern", "Coordinator"
]
categories = [
    "iOS"
]
series = ["iOS Design Pattern"]
images = [
  "/images/Coordinator_Pattern _basic_wide_ver.png"
]
draft = false

+++

간단한 예제와 함께 Coordinator에 대해 알아봅시다 🙌🏻<br>

<br>

<!--more-->

##    <  📑 목차  >

* [Coordinator란?](#Coordinator란?)
* [Coordinator Pattern을 찾게 된 과정](#Coordinator-Pattern을-찾게-된-과정)
* [Coordinator 소개](#Coordinator-소개)
* [간단한 Coordinator 예제](#간단한-Coordinator-예제)
* [Coordinator를 통해 누릴 수 있는 이점은?](#Coordinator를-통해-누릴-수-있는-이점은?)
* [추가적으로 고민해볼만한 점](#추가적으로-고민해볼만한-점)
* [참고](#참고)

<br><br>

> 🗣 이 글은 Coordinator에 대해 처음 접하는 분들을 위해 쉽게 설명한 글입니다. Coordinator 패턴은 다양한 형태로 사용할 수 있기 때문에 아주 기본적인 컨셉을 소개하는 내용을 중심으로 다뤘습니다. 잘못된 부분이나 애매한 부분에 대해서 댓글로 피드백 주시면 감사하겠습니다🙌🏻 또한 댓글을 통한 토론도 환영입니다!

<br>

[전체 코드](https://github.com/dev-Lena/Coordinator)는 이곳에서 확인할 수 있습니다.<br>추가적인 내용은 간단한 예제로 살펴보는 [iOS Design/Architecture Pattern: Coordinator - Advanced]() 에 있습니다 👍🏻

<br><br>

## Coordinator란?

*Soroush Khanlou*가 NSSpain conference 2015에서 iOS 커뮤니티에 소개한 패턴으로 [Soroush Khanlou](https://khanlou.com/2015/10/coordinators-redux/) 의 글에 보면 코디네이터(Coordinator)를 이렇게 소개하고 있습니다.

> So what is a coordinator? ***A coordinator is an object that bosses one or more view controllers around.*** Taking all of the driving logic out of your view controllers, and moving that stuff one layer up is gonna make your life a lot more awesome.

> 코디네이터란? ***코디네이터는 하나 이상의 뷰 컨트롤러에게 지시를 내리는 객체입니다***.
> ... 이하 생략

여기서 말하는 지시는 화면 전환에 대한 지시를 말합니다. **Coordinator 패턴에서는 현재 View Controller에서 다음 View Controller로 이동할 때 직접 push / present 등의 화면 전환을 하는 대신 모든 화면 내비게이션을 코디네이터가 관리합니다.**  즉, View Controller에서 Navigation의 책임을 다른 클래스로 분리합니다. 따라서 View Controller들이 서로 분리될 수 있고 쉽게 재사용될 수 있습니다. 

<br>

## Coordinator Pattern을 찾게 된 과정

<br>

**1.  거대한 ViewController를 피하고 싶다.**

<img src="https://user-images.githubusercontent.com/52783516/98208138-24e20800-1f80-11eb-9a00-e80118edcebb.png" alt="image" style="zoom:25%;" />

UIViewController에서는 이렇게 많은 일을 할 수 있습니다. 반대로 말하면 많은 코드가 집중되고 그 만큼 책임이 집중되어서 쉽게 거대한(Massive)해질 수 있습니다. 저는 이런 View Controller를 가볍게 하기 위해 좋은 방법이 없나 고민을 계속 하고 있는데요. Coordinator는 UIViewController의 역할 중에서 Navigation 역할을 해주는 객체가 되어주는 것입니다. 그러면 좀 더 View Controller를 좀 더 가볍게 만들 수 있겠죠?

<br>

**2. 화면 전환하는 코드가 흩어져 있어 파악하고 관리하기 어렵다.** 

화면이 많아지면 각 View Controller에 화면을 전환하는 코드가 흩어져있어 파악/관리하기가 어렵죠. 그래서 이를 관리하는 객체가 있으면 좋겠다는 생각이 들었습니다. 그러다가 발견한게 Coordinator인데요! Coordinator에는 화면 내비게이션에 대한 코드가 모여있어서 파악/관리하기 용이합니다. 게다가 다음 포스팅에서 소개하겠지만 책임과 구분(역할)에 따라서 여러 개의 Coordinator를 사용할 수 있어 객체지향적으로 구현하기 더 수훨해집니다. 

<br>

## Coordinator 소개<br>

### Coordinator 특징<br>

- coordinator 별로 하나 또는 그 이상의 View Controller를 보유합니다. 
- 각 coordinator는 일반적으로 “**start**”라고 불리는 메서드를 사용하여 View Controller를 표시합니다.
- 각 View Controller에는 coordinator에 대한 **delegate** reference가 있습니다.
- 각 coordinator는 **child** coordinators 배열을 가지고 있습니다.
- 각 child coordinator는 **parent** coordinator에 대한 delegate reference가 있습니다.

<br>

### 추가로<br>

<br>

 [Soroush Khanlou](https://khanlou.com/2015/10/coordinators-redux/) 의 글에서 Coordinator 소개할 때  app coordinator가 iOS를 위해 특별히 만들어진 Application Controller의 스페셜 버전이라고 소개하고 있는데요. 애플리케이션 아키텍쳐에 관심이 많으신 분들은 한 번 읽어보면 좋을 것 같습니다. ( 저는 다음에 읽어볼께요 😊😝 )

> It all starts from the app coordinator. The app coordinator solves the problem of the overstuffed app delegate. The app delegate can hold on to the app coordinator and start it up. The app coordinator will set up the primary view controller for the app. You can find this pattern in the literature, in books like [Patterns of Enterprise Application Architecture](http://www.amazon.com/Patterns-Enterprise-Application-Architecture-Martin/dp/0321127420). They call it the [Application Controller](http://martinfowler.com/eaaCatalog/applicationController.html). The app coordinator is a special version of an application controller, specifically made for iOS. The app coordinator can create and configure view controllers, or it can spawn new child coordinators to perform subtasks.

> (이번에 구현할 예제의 MainCoordinator 클래스가 여기서 app coordinator라고 할 수 있습니다.)
> App Delegate에서 app coordinator를 가지고 있고 시작합니다.
> app coordinator는 앱의 첫 화면 View Controller을  세팅합니다. 
> 이 패턴을 [Patterns of Enterprise Application Architecture](http://www.amazon.com/Patterns-Enterprise-Application-Architecture-Martin/dp/0321127420) 이 책에서 찾을 수 있으며 책에서는 이러한 패턴을 [Application Controller](http://martinfowler.com/eaaCatalog/applicationController.html) 라고 부릅니다.
> app coordinator는 iOS를 위해 특별히 만들어진 Application Controller의 스페셜 버전입니다.
> app coordinator는 View Controller를 생성하고 구성할 수 있고 하위 작업을 수행하기 위해 새로운 하위 coordinator를 생성할 수 있습니다.

## 간단한 Coordinator 예제

<br>

 ![coordinator basic](/images/coordinator_basic.gif)

<br>
이 예제는 [How to use the coordinator pattern in iOS apps](https://www.hackingwithswift.com/articles/71/how-to-use-the-coordinator-pattern-in-ios-apps) 를 참고하여 만들었습니다.<br>
전체 코드는 제 [깃허브 레파지토리](https://github.com/dev-Lena/Coordinator)에 있고 Basic에 대한 내용은 [coordinator-basic 브랜치](https://github.com/dev-Lena/Coordinator/tree/basic-coordinator)에서 확인할 수 있습니다. <br>첫 세팅 이후 구현 과정은 [PR](https://github.com/dev-Lena/Coordinator/pull/2) 에 커밋으로 남겨놓았습니다. <br>

<br>

<img src="https://user-images.githubusercontent.com/52783516/98215889-b2772500-1f8b-11eb-82be-9ffafb32d36e.png" alt="image" style="zoom:67%;" />

예제 프로젝트의 구조입니다. 먼저 구조를 그리고 구현하면 훨씬 도움이 될 것 같네요!

<br><br>

1. **Initial Setting**

   본격적으로 구현을 시작하기 전에 Scene Delegate를 삭제해주세요. 
   → [SceneDelegate 삭제 방법](https://github.com/dev-Lena/Coordinator/issues/4)

   SceneDelegate와 AppDelegate에 대해 궁금하다면 [여기](https://lena-chamna.netlify.app/post/appdelegate_and_scenedelegate/)를 참고해주세요.

2. **첫 화면 열기**
   <br>
   **Coordinator 프로토콜**

   ```swift
   import UIKit
   
   protocol Coordinator {
       var childCoordinators: [Coordinator] { get set }
       var navigationController: UINavigationController { get set }
   
       func start()
   }
   ```

   <br>
   **MainCoordinator 클래스**

   ```swift
   class MainCoordinator: NSObject, Coordinator {
       
       var childCoordinators = [Coordinator]()
       var navigationController: UINavigationController
   
       init(navigationController: UINavigationController) {
           self.navigationController = navigationController
       }
   
       func start() {
           let vc = ViewController.instantiate()
           vc.coordinator = self
           navigationController.pushViewController(vc, animated: false)
       }
     // 나머지 구현부 생략
   }
   ```

   <br>
   **AppDelegate에서 첫 화면 띄우기**

   ```swift
   @main
   class AppDelegate: UIResponder, UIApplicationDelegate {
   
       var coordinator: MainCoordinator?
       var window: UIWindow?
   
       func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
           
           window = UIWindow(frame: UIScreen.main.bounds)
           let navController = UINavigationController()
   
         // coordinator 인스턴스 생성
           coordinator = MainCoordinator(navigationController: navController)
         // Coordinator로 첫 화면 열기
           coordinator?.start()
   
           window = UIWindow(frame: UIScreen.main.bounds)
           window?.rootViewController = navController
           window?.makeKeyAndVisible()
   
           return true
       }
   }
   ```

   

3. **첫 화면에서 다음 화면으로 이동하기**
   <br>
   **MainCoordinator 클래스**

   ```swift
   class MainCoordinator: NSObject, Coordinator {
     // 나머지 구현부 생략
         func buySubscription() {
           let vc = BuyViewController.instantiate()
           vc.coordinator = self
           navigationController.pushViewController(vc, animated: true)
       }
   
       func createAccount() {
           let vc = CreateAccountViewController.instantiate()
           vc.coordinator = self
           navigationController.pushViewController(vc, animated: true)
       }
   }
   ```

   <br>
   **ViewController**

   ```swift
   class ViewController: UIViewController, Storyboarded {
       
       weak var coordinator: MainCoordinator?
   
       override func viewDidLoad() {
           super.viewDidLoad()
       }
       
       @IBAction func buyTapped(_ sender: Any) {
         // coordinator를 통해 화면 전환
           self.coordinator?.buySubscription()
       }
   
       @IBAction func createAccount(_ sender: Any) {
         // coordinator를 통해 화면 전환
           self.coordinator?.createAccount()
       }
   }
   ```

   

## 추가적으로 고민해볼만한 점

1. MVVM과 함께 MVVM-C 패턴으로도 사용되는데요. 어떻게 MVVM과 함께 사용할지 고민해보는 것도 좋을 것 같습니다.
2. childCoordinators를 언제 어떻게 쓰는지에 대해서 고민해보면 좋을 것 같습니다. 

<br>

## 참고

1. [How to use the coordinator pattern in iOS apps](https://www.hackingwithswift.com/articles/71/how-to-use-the-coordinator-pattern-in-ios-apps)
2. [Coordinators Redux](https://khanlou.com/2015/10/coordinators-redux/)
3. [daveneff/Coordinator](daveneff/Coordinator)
4. [iOS : Coordinator pattern in Swift](https://medium.com/@saad.eloulladi/ios-coordinator-pattern-in-swift-39a15aa3b01b)