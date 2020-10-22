+++
authors = [
    "Lena"
]
title = "간단한 예제로 살펴보는 iOS Design/Architecture Pattern: MVVM"
date = 2020-10-21T23:26:10+09:00
description = "MVVM Design/Architecture Pattern with UIKit"
tags = [
    "Design Pattern", "Architecture Pattern", "MVVM"
]
images = [
  "/images/iOS_MVVM.png"
]
draft = false
+++

MVVM을 구현한 간단한 예제를 살펴보며 MVVM에 대해 알아봅시다 🙌🏻<br><br>

<!--more-->

##    <  📑 목차  >
* [MVVM이란?](#MVVM이란?)
  * [MVC에서 MVVM을 찾게 된 과정](#MVC에서-MVVM을-찾게-된-과정)
* [MVVM의 규칙들](#MVVM의-규칙들)
* [간단한 MVVM 예제](#간단한-MVVM-예제)
* [View와 ViewModel 바인딩 이해하기](#View와-ViewModel-바인딩-이해하기)
* [더 용이해진 테스트](#더-용이해진-테스트)
* [결론](#결론)
* [참고](#참고)


<br><br>
🗣 이 글은 MVC 패턴은 익숙하지만 MVVM은 처음 접한 분들을 위한 글입니다. 잘못된 부분이나 애매한 부분에 대해서 댓글로 피드백 주시면 감사하겠습니다🙌🏻 또한 댓글을 통한 토론도 환영입니다!

<br><br>

## MVVM이란? 
<br>

<img src = "https://i.imgur.com/84VHiS4.png" width = "60%">

(출처: [Stanford 강의](https://youtu.be/4GjXq2Sr55Q), 위 강의는 SwiftUI 기반으로 MVVM을 설명하고 있습니다. 이 글은 UIKit을 기반으로 작성되었습니다.)<br><br>

위 이미지에서 **Works in concert with the concept of "reactive" user-interfaces.** 이 부분이 중요한 포인트입니다.<br>

MVVM을 정리해보자면 **❶ UI 로직과 비즈니스 로직을 분리하고, ❷ 리액티브한 UI 컨셉과 함께 협력하여 작동하는 디자인 패턴(아키텍쳐 패턴)** 라고 할 수 있을 것 같습니다.<br><br>

제가 MVVM에 대해서 찾아보게 된 계기도 바로 이 때문인데요. 그럼 본격적으로 MVVM에 대해서 좀 더 알아봅시다 🙌🏻

### MVC에서 MVVM을 찾게 된 과정

  * 상황 
    1. 입력/터치 이벤트가 발생했을 때 Model의 State가 변경되고 View가 State의 변화를 감지하고 있다가 변경되면 State에 맞게 View를 업데이트 하도록 구현 하고 싶었고, KVO/Notification을 이용해서 구현했습니다. 
    2. 그런데 View와 Model이 자신들의 역할에 충실한 것(View는 화면을 그리는 것, Model은 앱 데이터, 비즈니르 로직)을 우선시하여 코드를 짰더니 View Controller가 점점 무거워졌습니다.
    3. 또한 정상적으로 동작하는지 확인하기 위해 매번 시뮬레이터를 실행하면서 결과를 확인했었는데요. 화면이 많아지고 기능이 많아지다보니 이 작업이 상당히 비효율적이라고 느꼈었습니다. 
    4. 그래서 테스트 코드를 작성하기 시작했습니다. 그런데 이 때 View나 ViewController 인스턴스를 생성해야 하고 생각보다 제약과 불편점이 많았습니다. 
    5. 이를 테스트하기 위해서는 View와 비즈니스 로직을 분리하는 일이 필요하다는 걸 느꼈습니다.
    6. 그러던 중 1-5번 고민을 해결할 수 있는 좀 더 좋은 방법을 찾아가 발견한 것이 MVVM 패턴입니다. <br>

<br><br>

## MVVM의 규칙들

💡 MVVM에 대해서 공부하면서 MVVM이 뭔지, 어떻게 구현하는지 알아보기 위해 많은 예제와 자료를 봤는데요. 공부하면서 든 생각은 <br> **"MVVM에 정형화된 형식같은 건 없다. 다만 공통적으로 적용되는 규칙들이 있다."** 입니다.<br>

MVVM에 대한 설명을 모아보면 이렇습니다. <br>(참고로, 위 영상 [Stanford 강의](https://youtu.be/4GjXq2Sr55Q) 에서 MVVM에 대한 강의를 보면 MVVM에 대한 컨셉을 이해하기 좋습니다. 저의 경우 Stanford 강의에 나온 이미지와 함께 보면서 더 이해하기 쉬웠기 때문에 이미지도 함께 첨부하겠습니다. )<br>

<img src = "https://i.imgur.com/pW7dmjD.jpg" width = "70%"><br><br>


| **<span style="color:orange">View</span>** |
| -------- |
| 1. MVVM은 MVC와 달리 ViewController를 View로 취급한다. <br>2. 모든 UI 로직이 ViewModel에 있으므로 View/ViewController가 가벼워진다.(MVC에서보다)<br>3. View는 ViewModel을 참조한다(반대는 X).<br>4. View는 Model을 참조하지 않는다(반대도 O).<br>5. **View는 발표(publication)을  구독(subscribe)하고, 주시(관찰, observe)한다.**<br />     | 

| **<span style="color:orange">ViewModel</span>** |
| -------- |
| 1. MVVM은 **ViewModel을 통해 UI 로직과 비즈니스 로직**을 분리했다.<br>2. MVVM은 MVC와 달리 ViewModel이 있다.<br>3.  ViewModel은 Model을 참조한다(반대는 X).<br>4. **View 없이 테스트가 가능**하다.<br>5. ViewModel은 View input으로부터 Model을 업데이트한다. <br>6. ViewModel은 Model이 변경되면 View에 반영한다. (Model output으로부터 View를 업데이트한다.) <br>7. ViewModel은 **View에 직접적으로 이야기하지 않는다. 무언가 바뀌었다고 발표(publish)** 한다. <br>9. 모든 UI 컨트롤의 상태를 알려주는 프로퍼티들을 포함한다.     |

| **<span style="color:orange">Model</span>** |
| -------- |
| 1. **UI에 독립적이다.**<br>2. **SwiftUI나 UIKit을 import 하지 않는다.**<br>3. **App이 하는 일에 대한 데이터와 로직을 캡슐화**하려고 한다.<br>4. Model이 변경됐을 때 ViewModel에게 알린다.     |
<br><br><br>

## 간단한 MVVM 예제 

MVVM은 주로  RxSwift, RxCocoa, SwiftUI, Combine과 함께 사용합니다. 

그렇다면 RxSwift, RxCocoa, SwiftUI, Combine을 알아야만 MVVM을 사용할 수 있을까요? 그렇지 않습니다.

[raywenderlich - iOS MVVM Tutorial: Refactoring from MVC]( https://www.raywenderlich.com/6733535-ios-mvvm-tutorial-refactoring-from-mvc )에 위 4가지 라이브러리, 프레임워크 없이 MVVM을 구현한 예제가 나와있습니다. <br>자세한 구현 사항은 튜토리얼을 따라하면서 보면 될 것 같습니다. 여기서 얘기할 것은 이 예제를 바탕으로 한 MVVM을 설명해보고자 합니다. <br>이 글을 읽고 위 튜토리얼을 따라하면 이해가 더 잘 될것 같네요.<br><br>

**예제 화면과 파일 구조**<br>

<img src = "https://i.imgur.com/zfpmdb9.png" width = "40%"><img src = "https://i.imgur.com/GZ9Nrzi.png" width = "32.5%"><br>

인데요. 이 파일들 중 `WeatherViewController`, `WeatherViewModel`, `Observable` 클래스를 살펴볼 것입니다. 일단 `WeatherViewController`를 먼저 봐볼까요? <br><br>

~~~~swift
import UIKit

class WeatherViewController: UIViewController {
  @IBOutlet weak var cityLabel: UILabel!
  @IBOutlet weak var dateLabel: UILabel!
  @IBOutlet weak var currentIcon: UIImageView!
  @IBOutlet weak var currentSummaryLabel: UILabel!
  @IBOutlet weak var forecastSummary: UITextView!
  
  private let viewModel = WeatherViewModel()
  
  override func viewDidLoad() {
    // bind view model outputs to the views
    viewModel.locationName.bind { [weak self] locationName in // ❽
      self?.cityLabel.text = locationName // ❾
    }
    
    viewModel.date.bind { [weak self] date in
      self?.dateLabel.text = date
    }
    
    viewModel.icon.bind { [weak self] image in
      self?.currentIcon.image = image
    }
    
    viewModel.summary.bind { [weak self] summary in
      self?.currentSummaryLabel.text = summary
    }
    
    viewModel.forecastSummary.bind { [weak self] forecast in
      self?.forecastSummary.text = forecast
    }
    
  }
  
  @IBAction func promptForLocation(_ sender: Any) {
    ...
    let submitAction = UIAlertAction(
      title: "Submit",
      style: .default) { [unowned alert, weak self] _ in
        guard let newLocation = alert.textFields?.first?.text else { return }
        self?.viewModel.changeLocation(to: newLocation) // ❷, update ViewModel
    }
    ...
  }
   
  // 나머지 구현부 생략
}

~~~~

1. ViewModel을 소유하고 있습니다.
2. ViewController가 가지고 있는 View들(여기에서는 IBOutlet으로 연결된 View들)을 `viewDidLoad()` 에서 ViewModel과 바인드(bind) 해주고 있습니다.
3. `IBAction func promptForLocation(_:)`에서도 보면 ViewModel을 업데이트(`changeLocation(to:)`)해주고 있네요.

<br><br>

여기서 View들과 ViewModel을 바인드하는 부분을 좀 더 살펴보죠.<br>

~~~~swift
import Foundation
import UIKit.UIImage

public class WeatherViewModel {
  
  private static let defaultAddress = "Anchorage, AK"
  private let geocoder = LocationGeocoder()
  let locationName = Observable("Loading...") // ❹, ❺
  let date = Observable(" ")
  let icon: Observable<UIImage?> = Observable(nil)  //no image initially
  let summary = Observable(" ")
  let forecastSummary = Observable(" ")
  
  init() {
    changeLocation(to: Self.defaultAddress)
  }
  
  func changeLocation(to newLocation: String) {
    locationName.value = "Loading..."
    // CoreLocation에서 location의 장소 이름을 가져온다.
    geocoder.geocode(addressString: newLocation) { [weak self] locations in
      guard let self = self else { return }
      if let location = locations.first {
        self.locationName.value = location.name // ❸ 👊🏻이 부분
        self.fetchWeatherForLocation(location)
        return
      }
      self.locationName.value = "Not found"
      self.date.value = ""
      self.icon.value = nil
      self.summary.value = ""
      self.forecastSummary.value = ""
    }
  }
  // 나머지 구현부 생략
}
~~~~

`WeatherViewModel` 클래스입니다. 여기에 프로퍼티들을 보니까 `Observable` 타입이네요. 그렇다면 `Observable` 클래스도 어떻게 생겼는지 보죠.<br><br>

~~~~swift
import Foundation

final class Observable<T> {
  typealias Listener = (T) -> Void // ❼
  var listener: Listener?
  var value: T {
    didSet { // ❻
      listener?(value) // ❻
    }
  }
  init(_ value: T) {  // ❺
    self.value = value
  }
  func bind(listener: Listener?) {
    self.listener = listener
    listener?(value)
  }
}
~~~~

<br>음... 어떻게 동작하는걸까요? 

**위에 날씨를 검색할 "Paris"를 입력하고 Submit 버튼을 눌렀다고 가정해봅시다.**<br><br>

~~~~swift
// WeatherViewController 클래스
@IBAction func promptForLocation(_ sender: Any) { // ❷
  ...
  self?.viewModel.changeLocation(to: newLocation) // ❸
  ...
}
~~~~

~~~~swift
// WeatherViewModel 클래스
func changeLocation(to newLocation: String) {
  ...
  self.locationName.value = location.name // ❸ 👊🏻이 부분
  ...
}
~~~~

이 부분에서 ViewModel을 업데이트 해주고 있죠? <br> 그럼 이제 ViewModel에서 변경된 value에 맞게 View를 업데이트 해줍니다.<br><br><br>

## View와 ViewModel 바인딩 이해하기
<br>
좀 더 상세히 풀어서 설명해 볼께요.<br><br>  

**이 부분을 이해하는게 중요한 포인트입니다**  ⭐️⭐️⭐️ <br><br>
(각 번호에 해당하는 코드는 위에 코드블록에 주석으로 표시해놨습니다.)

> 1. 사용자가 날씨 검색을 할 도시로 "Paris"를 입력하고 Submit 버튼을 눌렀다.
>
> 2.  `WeatherViewController` 클래스에서 `@IBAction func promptForLocation(_:)` 메소드가 호출된다.
>
> 3. `func promptForLocation(_:)`에서 `changeLocation(to:)`메소드를 호출한다. 이 메소드에서  `WeatherViewMode` 클래스가 가지고 있는 프로퍼티 `locationName`을 업데이트한다. → `self.locationName.value = location.name`  👊🏻이 부분
>
> 4. `locationName` 은 `Observable`  타입이다.
>
> 5. `Observable` 은 init의 매개 변수로 `value`를 주입하고, `Observable` 에서는 제네릭을 사용하고 있기 때문에 이 때 `value`의 타입과 같게 타입이 결정된다. 따라서 여기서는  `WeatherViewModel` 클래스에서 String으로 초기화 하였기에 String 타입이다. → `let locationName = Observable("Loading...")` 부분
>
> 6. `value`가 변경되면(didSet) `listener?(value)`  즉, 클로저를 실행한다.
>
> 7. `listener?(value)` 는 `Listener` 타입이다. → `Listener = (T) -> Void`
>
> 8. 그럼 여기서 실행할 클로저는 무엇이냐면,  `WeatherViewController` 클래스에서 View와 ViewModel을 바인드 해줬던 **👍🏻이 부분**이다. →<br>
>
>    ```swift
>    override func viewDidLoad() {
>        viewModel.locationName.bind { [weak self] locationName in // ❽ 👍🏻이 부분 
>          self?.cityLabel.text = locationName 
>        }
>    }
>    ```
>
>   9. 클로저에 담겨 온 `location.name` value를 `cityLabel.text `로 설정한다.



 <br>이 동작 흐름이 이해가 되셨다면 거의 다 오신겁니다!👍🏻 이 부분을 이해했다면 나머지 부분은 MVC와 비슷합니다.

<br><br>

## 더 용이해진 테스트 

자 이렇게 되면 View 없이 ViewModel만 가지고 테스트하기 훨씬 용이합니다. 시뮬레이터를 실행하거나 View나 ViewController 인스턴스를 생성해서 장소 이름을 변경하면서 `cityLabel`의 text가 장소 이름에 맞게 제대로 업데이트 되는지 확인하는 것 보다 정상적으로 동작하는지 확인하기 위해 아래와 같이 ViewModel에서 장소 이름을 설정하면 올바른 locationName을 가지고 오는지 ViewModel만 가지고 확인할 수 있으니까요. 🎉 <br>

~~~~swift
import XCTest
@testable import Grados

class WeatherViewModelTests: XCTestCase {
  
  func testChangeLocationUpdatesLocationName() {

    let expectation = self.expectation( 
      description: "Find location using geocoder") 
    let viewModel = WeatherViewModel() 
    viewModel.locationName.bind { // locationName의 초기값은 "Loading..."
      if $0.caseInsensitiveCompare("Richmond, VA") == .orderedSame {
        expectation.fulfill()
      }
    }
    DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
      viewModel.changeLocation(to: "Richmond, VA") 
    }
    waitForExpectations(timeout: 8, handler: nil)
  }
  
}

~~~~

<br><br>

## 결론

MVVM 맛보기 정도의 글이라고 생각해주시면 좋을 것 같습니다. MVC에서 단점이라고 느꼈던 점들([MVC에서 MVVM을 찾게 된 과정](#MVC에서-MVVM을-찾게-된-과정))에 대해서 MVVM에서는 이렇게 할 수 있구나 느낀 점이 있는데 혹시 저처럼 MVVM이 처음이신 분들에게 공유하면 좋을 것 같아서 정리해봤습니다. 저도 MVVM에 대해서 잘 알려면 아직 멀었지만 앞으로 MVVM 관련해서 더 공부하면서 점점 더 나은 코드와 구조, 성능의 앱을 만들기 위해서 고민할 수 있을 것 같아 많이 기대가 됩니다.👻 
<br><br>

## 참고

[raywenderlich - iOS MVVM Tutorial: Refactoring from MVC](https://www.raywenderlich.com/6733535-ios-mvvm-tutorial-refactoring-from-mvc )

[Stanford 강의 Lecture 2: MVVM and the Swift Type System](https://youtu.be/4GjXq2Sr55Q)

