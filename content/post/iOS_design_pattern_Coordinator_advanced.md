+++

authors = [
    "Lena"
]
title = "간단한 예제로 살펴보는 iOS Design/Architecture Pattern: Coordinator - Advanced"
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
  "/images/Coordinator_Pattern _advanced_wide_ver.png"
]
draft = false

+++

간단한 예제와 함께 Coordinator에 대해 알아봅시다 🙌🏻<br>

<br>

##    <  📑 목차  >

* [Coordinator를 책임과 역할에 따라 분리할 수 없을까?](#Coordinator를-책임과-역할에-따라-분리할-수-없을까?)
* [Parent Coordinator와 Child Coordinator 이해하기](#Parent-Coordinator와-Child-Coordinator-이해하기)
* [Child Coordinator의 일이 끝났을 때는?](#Child-Coordinator의-일이-끝났을-때는?)
* [커스텀 Back 버튼을 따로 만들었을 때는?](#커스텀-Back-버튼을-따로-만들었을-때는?)
* [참고](#참고)

<br><br>

> 🗣 이 글은 Coordinator에 대해 처음 접하는 분들을 위해 쉽게 설명한 글입니다. Coordinator 패턴은 다양한 형태로 사용할 수 있기 때문에 아주 기본적인 컨셉을 소개하는 내용을 중심으로 다뤘습니다. 잘못된 부분이나 애매한 부분에 대해서 댓글로 피드백 주시면 감사하겠습니다🙌🏻 또한 댓글을 통한 토론도 환영입니다!

<br>

**<span style="color:orange">[전체 코드](https://github.com/dev-Lena/Coordinator)는 이곳에서 확인할 수 있습니다.<br>Coordinator에 대한 소개 내용은 [iOS Design/Architecture Pattern: Coordinator - Basic](https://lena-chamna.netlify.app/post/ios_design_pattern_coordinator_basic/) 에 있습니다 👍🏻</span>**

<br><br>

## <span style="color: #7f00ff">Coordinator를 책임과 역할에 따라 분리할 수 없을까?</span>

하나의 Coordinator만 사용하다가 '용도별로 화면별로 Coordinator를 여러개 두고 사용할 수는 없을까?'라는 생각이 들어서 Coordinator에 대해서 좀 더 알아봤는데요. 뭔가 childCoordinators를 이용하면 여러 개를 둘 수 있을 것 같더라구요. 그래서 좀 더 알아봤습니다.<br>

<img src="https://user-images.githubusercontent.com/52783516/98239828-e95d3300-1fab-11eb-8a97-2eb46c32ed54.png" alt="image" style="zoom:67%;" />

여러 개의 Coordinator를 두었을 때의 구조는 이렇습니다. 이번 예제에서는 parent Coordinator(Main Coordinator)와 child Coordinator(Buy Coordinator) 두 개의 Coordinator를 사용합니다.

이번 글에서는 예제 코드에 대해 설명하고자 합니다. 전체 예제 코드 및 동작 예시 화면은 제 [깃허브](https://github.com/dev-Lena/Coordinator)에서 확인할 수 있습니다. 구현 과정은 [PR](https://github.com/dev-Lena/Coordinator/pull/7)에 커밋으로 남겨놨습니다. <br>

**<span style="color:orange">아래 부터는 [이전 과정](https://lena-chamna.netlify.app/post/ios_design_pattern_coordinator_basic/) 을 리팩토링하는 방식으로 진행하겠습니다.</span>**<br>

## <span style="color: #7f00ff">Parent Coordinator와 Child Coordinator 이해하기</span>

2개 이상의 여러 개의 Coordinator를 사용할 경우, 이런 구조로 확장해서 사용할 수 있습니다. 

<img src="https://miro.medium.com/max/1121/1*FJ3oNcNvPgHLD_QP6jVxpA.png" alt="Image for post" style="zoom: 85%;" />

<p align="center">이미지 출처: medium.com/@pavlepesic - How to implement flow coordinator pattern </p>

 이 때, 상위에 있는 parent coordinator와 하위에 있는 child coordinator의 관계를 맺어줘야 합니다. 이때 관계를 맺는다는 말은, 두 객체끼리 의사소통할 수 있도록 만들어주는 것을 의미하는데요. 그 과정을 소개하겠습니다.

<br>

1. 먼저 `BuyCoordinator`라는 새로운 `Coordinator` 클래스를 만듭니다. 이 coordinator는 `MainCoordinator`의 child coordinator입니다.

   ```swift
   class BuyCoordinator: Coordinator {
     // 나머지 구현부 생략
       func start() {
           // MainCoordinator.buySubscription()의 코드를 이곳으로 옮겨왔습니다.
           let vc = BuyViewController.instantiate()
       	  vc.coordinator = self
           navigationController.pushViewController(vc, animated: true)
       }
   }
   ```

2. 그리고 `BuyViewController`에서는 `BuyCoordinator`를 사용할 것이기 때문에 `BuyViewController`가 가지고 있던 coordinator 변수의 타입을 `BuyCoordinator`로 변경합니다. 

   ```swift
   class BuyViewController: UIViewController, Storyboarded {
       weak var coordinator: BuyCoordinator?
       override func viewDidLoad() {
           super.viewDidLoad()
       }   
   }
   ```

3. `BuyCoordinator`에서 `parentCoordinator`를 선언해줍니다. 이때, `MainCoordinator`에서 child를 소유하고 있기 때문에 retain cycle을 피하기 위해서 `weak` 참조로 선언해줍니다. 

   ~~~swift
   class BuyCoordinator: Coordinator {
       weak var parentCoordinator: MainCoordinator? // retain cycle을 피하기 위해 weak 참조로 선언해주세요.
       // 나머지 구현부 생략
   }
   ~~~

4. MainCoordinator 클래스에서 To Buy 버튼을 눌렀을 때 BuyCoordinator를 통해 화면이 전환하는 코드를 삽입합니다. 이때, **<span style="color:orange">parent coordinator(`MainCoordinator`)와 child coordinator(`BuyCoordinator`) 간 커뮤니케이션을 할 수 있도록 관계를 맺어줘야 합니다.</span>**

   ```swift
   class MainCoordinator: NSObject, Coordinator {
       var childCoordinators = [Coordinator]()
       
       func buySubscription() {
           // BuyCoordinator 타입의 인스턴스 생성
           let child = BuyCoordinator(navigationController: navigationController)
           // BuyCoordinator의 parent coordinator로 self 지정
           child.parentCoordinator = self
           // BuyCoordinator을 자신의 child coordinator로 추가 
           childCoordinators.append(child)
           // BuyViewController로 전환
           child.start()
       }
       // 나머지 구현부 생략
   }
   ```

<br>

## <span style="color: #7f00ff">Child Coordinator의 일이 끝났을 때는?</span>

지금까지는 coordinator를 추가하는 작업을 해줬는데요. 그렇다면 child coordinator가 작업을 끝냈을 때는 어떻게 할까요? 예를 들어 `BuyViewController`에서 다시 첫화면인 `ViewController`로 돌아간다면? parent coordinator에게 알리고 child coordinator에서 지워야합니다. 아래는 두 가지 방법을 소개합니다.

`MainCoordinator`에는 프로퍼티로 가지고 있는 `childCoordinators`배열에서 해당 coordinator를 제거하는 메서드를 추가합니다.

~~~swift
class MainCoordinator: NSObject, Coordinator {
    func childDidFinish(_ child: Coordinator?) {
      for (index, coordinator) in childCoordinators.enumerated() {
          if coordinator === child {
              childCoordinators.remove(at: index)
              break
          }
      }
      // 나머지 구현부 생략
}
~~~

이 때 `===`는 클래스의 두 인스턴스가 동일한 메모리를 가리키는지 점검하는 연산자이므로 Coordinator 프로토콜을  클래스 전용 프로토콜(class-only protocol)로 만들어주세요.

```swift
protocol Coordinator: AnyObject {
```

<br>

### 자 그렇다면 언제 이 메서드를 호출할까요?

바로 화면이 사라질 때 호출하면 됩니다. 화면이 사라질 시점에 이 작업을 하기 위해서는 ViewController의 `viewDidDisappear()` 메서드나 `Navigation Controller Delegate`의 `didShow()` 메서드에서 위 메서드를 호출하면 됩니다. <br>

**첫번째: UIViewController의 viewDidDisappear() 사용**

먼저 UIViewController의 라이프사이클 메서드를 사용하여 BuyCoordinator에 할 일이 끝났다는걸 parent coordinator에게 알리기 위해서는 BuyCoordinator에 메서드를 didFinishBuying() 추가합니다.

~~~swift
class BuyCoordinator: Coordinator {
    func didFinishBuying() {
        parentCoordinator?.childDidFinish(self)
    }
  // 나머지 구현부 생략
}
~~~

그리고 `BuyViewController.viewDidDisappear()`에서 `didFinishBuying()`메서드를 호출합니다.

~~~swift
class BuyViewController: UIViewController, Storyboarded {
   override func viewDidDisappear(_ animated: Bool) {
       super.viewDidDisappear(animated)
       coordinator?.didFinishBuying()
   }
}
~~~


이 방법은 child coordinator에서 여러 개의 ViewController를 관리할 때 `viewDidDisappear()` 메서드가 이르게 호출될 수 있는 가능성이 있기 때문에 유의해야합니다.

<br><br>

**두번째: `UINavigationControllerDelegate`의 `didShow()` 사용**

이 방법에서는 `viewDidDisappear()`를 사용하지 않습니다. <br>**`MainCoordinator`가 Navigation Controller의 상호작용을 바로 감지할 수 있기 때문이죠!** <br>**또한 그렇기 때문에 coordinator가 관리하는 ViewController가 많을수록  coordinator 스택에 혼동이 오는 걸 피하기 위해 Navigation Controller Delegate를 채택하는 방법을 추천합니다.**

먼저 `MainCoordinator`클래스에 `NSObject`를 상속하고 `UINavigationControllerDelegate`를 채택해줍니다. 그리고 `navigationController`의 `delegate`를 `self`로 지정합니다.<br>

```swift
class MainCoordinator: NSObject, Coordinator {
    func start() {
        let vc = ViewController.instantiate()
        vc.coordinator = self
      // navigationController의 delegate를 self로 지정
        navigationController.delegate = self
        navigationController.pushViewController(vc, animated: false)
    }
}
```

<br>그리고 난 다음 `didShow` 메서드 구현합니다.<br>

```swift
extension MainCoordinator: UINavigationControllerDelegate {
  func navigationController(_ navigationController: UINavigationController, didShow viewController: UIViewController, animated: Bool) {

    // 이동할 ViewController A
    guard let fromViewController = navigationController.transitionCoordinator?.viewController(forKey: .from) else {
        return
    }

    // 이동할 ViewController가 navigationController의 viewControllers에 포함되어있으면 return. 왜냐하면 여기에 포함되어있지 않아야 현재 이동할 ViewController A가 사라질 화면(뒤로 이동하니까)의미.
    if navigationController.viewControllers.contains(fromViewController) {
        return
    }

    // child coordinator가 일을 끝냈다고 알림.
    if let buyViewController = fromViewController as? BuyViewController {
        childDidFinish(buyViewController.coordinator)
    }
  }
}
```

## <br><br>

### 이제 의도한 대로 동작하는지 확인해봅시다.

`viewDidDisappear()` 를 이용한 방법이나 `UINavigationControllerDelegate`를 이용한 방법 모두 `childCoordinators`를 잘 처리해주는지 확인해봤는데요.

<img src="https://user-images.githubusercontent.com/52783516/98250868-5926ea00-1fbb-11eb-931f-c8fa0114d4f8.png" alt="image" style="zoom: 70%;" />

`viewDidDisappear()` 를 이용한 방법: childCoordinators가 `0 element`

<img src="https://user-images.githubusercontent.com/52783516/98250878-5c21da80-1fbb-11eb-8aef-4bb77de4acae.png" alt="image" style="zoom:80%;" />

`UINavigationControllerDelegate` 를 이용한 방법: 에서도 childCoordinators가 `0 element` 

두 경우 모두 잘 처리 됐네요! 👍🏻 더이상 사용하지 않는 child coordinator가 childCoordinators에서 지워졌습니다.


<br>

## <span style="color: #7f00ff">커스텀 Back 버튼을 따로 만들었을 때는?</span>

추가적으로 구현을 하다가 navigation bar에 있는 back 버튼( `<` )이 아니라 제가 만든 이전화면으로 돌아가는 버튼을 만들었을 경우가 있었는데요. 이 내용도 소개하면 좋을 것 같아서 예제 프로젝트에 버튼을 추가해서 해봤습니다. 이 때에는	`self.navigationController?.popViewController(animated: true)` 이 메서드를 사용하면 되는데요. 이 메서드를 사용하면 navigation bar에 있는 back 버튼( `<` )을 눌렀을 때와 동일하게 `didShow()`와  `childDidFinish()`가 호출됩니다. 

<img src="/Users/keunnalee/Library/Application Support/typora-user-images/image-20201106191338269.png" alt="image-20201106191338269" style="zoom: 85%;" />



## 참고

1. [Advanced coordinators in iOS](https://www.hackingwithswift.com/articles/175/advanced-coordinator-pattern-tutorial-ios)
2. [daveneff/Coordinator](daveneff/Coordinator)



## 이미지 출처

1. [How to implement flow coordinator pattern](https://medium.com/@pavlepesic/flow-coordination-pattern-5eb60cd220d5)