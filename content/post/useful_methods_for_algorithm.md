+++
authors = [
    "Lena"
]
title = "Swift로 알고리즘 풀 때 유용한 메서드 모음"
date = 2021-03-18T21:06:41+09:00
description = "Swift 알고리즘에서 많이 사용하는 메서드들을 정리합니다."
tags = [
   "Swift", "Algorithm"
]
categories = [
     "Swift", "Algorithm"
]
series = ["Algorithm"]
images = [
  "/images/algorithmImage.png"
]
draft = false

+++

Swift로 알고리즘 풀 때 자주 사용하는 메서드를 정리합니다. <br>

<br>

<!--more-->
## 바로 찾아가기
[0. 배열](#0.-배열)
  **0.1**. 배열의 첫번째 값과 마지막 값
  **0.2.** 1차원 배열 선언 방법
  **0.3.** 2차원 배열 선언 방법 + 2차원 배열 -> 1차원 배열 만들기
  **0.4.** 배열 순서 역순으로 만들기
  **0.5.** 배열 정렬하기
  **0.6.** 배열의 특정 원소 개수 세기 
  **0.7.** 배열의 각 요소의 수 세기(Counter)
  **0.8.** 고차함수
  **0.9.** 배열에서 특정 조건을 충족한 원소의 개수
  **0.10.** ForEach
[1. 두 배열 비교](#1.-두-배열-비교)
  **0.1**. 중복 제거
  **0.2.** 요소 확인
  **0.3.** 조건에 따라 거르기
  **0.4.** 중복 값만 담은 배열 만들기
  **0.5.** 중복되지 않은 값만 담은 배열 만들기
[2. Indices](#2.-Indices)
[3. enumerated](#3.-enumerated)
[4.-stride](#4.-stride)
[5. 절대값 + 진수 변환](#5.-절대값-+-진수-변환)
[6. 소수점 처리](#6.-소수점-처리)
[7. 제곱 관련](#7.-제곱-관련)
[8. 입력값 처리 관련(문자열 쪼개기)](#8.-입력값-처리-관련(문자열-쪼개기))
[9. 범위 체크](#9.-범위-체크)

[참고자료](#참고자료)

#### <span style="color:orange">**0. 배열**</span>
1. 배열의 첫번째 값과 마지막 값
```swift
// swift 4 ~
let array = [1, 2, 3, 2]
array.firstindex(of: 2) // 1
// 2를 값으로 갖고 있는 index 중 첫번째 index
array.lastIndex(of: 2) // 3
// 2를 값으로 갖고 있는 index 중 마지막 index
```
2. 1차원 배열 선언 방법
```swift
// ✅ 
let arr = [Int]() 
let arr = Array(repeating: 0, count: 5) 
let arr = [Int](repeating: 0, count: 5)
```
3. 2차원 배열 선언 방법 + 2차원 배열 -> 1차원 배열 만들기
```swift
// ✅ 2차원 배열 선언 방법
let arr = [[Int]]() 
let arr = Array(repeating: Array(repeating: 0, count: 3), count: 5) 
let arr = [[Int]](repeating: [Int](repeating: 0, count: 3), count: 5) 
let arr = Array(repeating: [Int](), count: 5) // graph에서 다른 노드와 다른 노드와의 거리를 Set으로 사용할 경우 
var graph = Array(repeating: [(Int, Int)](), count: n)

// ✅ 2차원 배열 -> 1차원 배열 만들기
var arr = [[1,2,3],[2,3],[4]] let flatten = arr.flatMap { $0 } // [1, 2, 3, 2, 3, 4] 
let reduced = arr.reduce([], +) // [1, 2, 3, 2, 3, 4] 
let joined = Array(arr.joined()) // [1, 2, 3, 2, 3, 4]
```
4. 배열 순서 역순으로 만들기
```swift
// ✅ 
arr.reverse() 
arr = arr.reversed()
```
5. 배열 정렬하기
```swift
// ✅ 오름차순 
arr.sort() arr.sort(by: <) 
arr = arr.sorted(by: <) // 정렬한 배열 반환
// ✅ 내림차순 
arr.sort(by: >) 
arr = arr.sorted(by: >) // 정렬한 배열 반환
```
6. 배열의 특정 원소 개수 세기 
```swift
// ✅ 
let arr = ["D", "D", "R", "D"] arr.filter { $0 == "D" }.count // 3
```
7. 배열의 각 요소의 수 세기(Counter)
```swift
// ✅ 
let arr = ["one", "two", "three", "four", "one"] var counter = [String: Int]() arr.forEach { counter[$0, default: 0] += 1 } print(counter) // ["three": 1, "one": 2, "two": 1, "four": 1]
```
8. 고차함수
```swift
let array = ["1","2","3","4","5"] 
// ✅ map 
// [1,2,3,4,5] 
var newArray = array.map { (value: String) -> Int in Int(value)! } 
// 위 식을 간소화 
newArray = array.map { Int($0)! } 
// ✅ filter 
// [4,5] 
newArray = newArray.filter { (value: Int) -> Bool in return value > 3 } 
newArray = newArray.filter { $0 > 3 } 
// ✅ reduce 
// 20 
let result1 = newArray.reduce(1) { (n1: Int, n2: Int) in return n1 * n2 } 
// return과 타입 생략 가능 
let result2 = newArray.reduce(1) { (n1, n2) in n1 * n2 } // 1은 초기 값 
let result3 = newArray.reduce(1) { $0*$1 } 
let result4 = newArray.reduce(1, *)

// 총 합
nums.reduce(0, +)
```
9. 배열에서 특정 조건을 충족한 원소의 개수
```swift
// ✅ 
let arr = ["D", "D", "R", "D"] arr.filter { $0 == "D" }.count // 3

10. `ForEach`  
 ```swift
// ✅ 
let ships = [
    "Star Trek": "Enterprise",
    "Firefly": "Serenity",
    "Aliens": "Sulaco"
]
ships.forEach {
    print("\($0.value) is from \($0.key)")
}
public func forEach(_ body: (Element) throws -> Void) rethrows {
    for element in self {
        try body(element)
    }
}
(1...100).forEach(calculateFactors)
```



#### <span style="color:orange">**1. 두 배열 비교**</span>
1. 중복 제거
```swift
// ✅ Set 없이 반복문으로 중복값 제거하기 
func removeDuplication(in array: [Int]) -> [Int]{
        var duplicationRemovedArray = array
        for index in 0 ... duplicationRemovedArray.count-1 {
            if index + 1 <= duplicationRemovedArray.count-1 {
                if duplicationRemovedArray[index] == duplicationRemovedArray[index+1] {
                    duplicationRemovedArray.remove(at: index)
                }
            }
        }
        return duplicationRemovedArray
   }
        
// ✅ Set 으로 중복값 제거하기
func removeDuplication(in array: [Int]) -> [Int]{
    let set = Set(array)
    let duplicationRemovedArray = Array(set)
    return duplicationRemovedArray
}
```
2. 요소 확인 `contains(element: )`
```swift
let seonho: String = "seonho"
print(seonho.contains("s"))
/* true */
print(seonho.contains("a"))
/* false */

let celebrity: [String] = ["seonho", "donghae", "hero"]
print(celebrity.contains("hero"))
/* true */
print(celebrity.contains("dindin"))
/* false */

//배열에 특정 조건을 만족하는 값이 있는지 확인
let numbers: [Int] = Array(1...12)

let numBool: Bool = numbers.contains { (number: Int) -> Bool in
    return number > 10
}
print(numBool)
/* true */
print(numbers.contains {$0 > 10})
/* true */
// ✅ 
```
3. 조건에 따라 거르기
```swift
// ✅ 
let oddArray = array.filter( { (value: Int) -> Bool in 
                              return (value % 2 == 0) } )
array.filter { $0 % 2 == 0 }
```
4. 중복 값만 담은 배열 만들기
```swift
// ✅ 
let newArr1 = array1.filter{array2.contains($0)}
let newArr2 = array2.filter{array1.contains($0)}
```
5. 중복되지 않은 값만 담은 배열 만들기
```swift
// ✅ 
let newArr1 = array1.filter{!array2.contains($0)}
let newArr2 = array2.filter{!array1.contains($0)}
```


#### <span style="color:orange">**2. Indices**</span>
```swift
// ✅ Indices
// Again, this is so similar to map(). I often quote Javier Soto, who said “map() allows us to express what we want to achieve, rather than how this is implemented.” The same is true of indices: rather than explicitly creating a range by hand, we’re relying on Swift to figure that out.
// 1. 배열을 안전하게 도는 것. Index 반환
let movies = [
    "The Phantom Menace",
    "Attack of the Clones",
    "Revenge of the Sith",
    "A New Hope",
    "The Empire Strikes Back",
    "Return of the Jedi"
]
for index in movies.indices {
    print("Episode \(index + 1): \(movies[index])")
}
// 이거랑 같은 코드
for i in 0 ..< movies.count {
    print("Episode \(index + 1): \(movies[index])")
}

// 2. 역순으로 도는 것
for i in movies.indices.reversed() { print(i) }
// 이거랑 같은 코드
for i in (0 ..< movies.count).reversed() { print(i) }

// Extension Array
// 만약 배열이 var a= [1,2,3,4]라면 a[6]을 넣는 순간 에러가 난다.
// 하지만 이렇게 Array를 확장해서 indices가 index범위라면 값을 반환하고 아니라면 nil을 반환하도록 할 수 있다.
extension Array {
    subscript (safe index: Int) -> Element? {
        return indices ~= index ? self[index] : nil    
    }
}
```
#### <span style="color:orange">**3. enumerated**</span>
```swift
// ✅ 
let ships = [
    "Star Trek": "Enterprise",
    "Firefly": "Serenity",
    "Aliens": "Sulaco"
]
for (index, ship) in ships.enumerated() {
    print("\(index + 1). \(ship.value) is from \(ship.key)")
}
// 이거랑 같은 코드
for ship in ships {
    print("\(ship.value) is from \(ship.key)")
}

// 참고로 Dictionary Reversed 가능
for ship in ships.reversed() {
    print("\(ship.value) is from \(ship.key)")
}


// You can even combine enumerated() and reversed(), 
// but be careful: enumerated().reversed() is not the same as reversed().enumerated():

// this will count down from 3 to 1
for (index, ship) in ships.enumerated().reversed() {
    print("\(index + 1). \(ship.value) is from \(ship.key)")
}

// this will count up from 1 to 3
for (index, ship) in ships.reversed().enumerated() {
    print("\(index + 1). \(ship.value) is from \(ship.key)")
}
```
#### <span style="color:orange">**4. stride**</span>
```swift
// ✅ To: 미만(포함X), Through: 이하(포함O)
// 0 1 2 3 4 
for i in stride(from: 0, to: 5, by: 1){} 
// 0 1 2 3 4 5 
for i in stride(from: 0, through: 5, by: 1){}
```

#### <span style="color:orange">**5. 절대값 + 진수 변환**</span>
```swift
// ✅ 절대값
abs(-11) // 11

// ✅ 진수 변환
let num = 45
// radix 뜻은 '근'
// String(var, radix: 3)의 경우 var의 타입은 Int
String(num, radix: 3) // 1200

// Int(var , radix: 3)의 경우 var의 타입은 String
// 45를 3진법으로 변환하고 앞뒤를 반전시킨 값을 10진법으로 표현한 것. 
Int(String(String(num, radix: 3).reversed()), radix: 3)! // 7
```
#### <span style="color:orange">**6. 소수점 처리**</span>
```swift
// ✅ 0. 원하는 소수점까지 반올림하고 싶을 때 
var value = 1.543 
value = round(value * 100) / 100 // 1.54
// ✅ 1. 반올림(round)
// 참고: 소수점이 5보다 크거나 같으면 올림. 소수점 자릿수에 대한 지정이 없는 경우 첫 번째 자리로 판단.
round(5.123)  // 5.0
round(5.456)  // 5.0
round(5.512)  // 6.0
round(5.567)  // 6.0
round(5.657)  // 6.0
round(5.765)  // 6.0
round(-3.453) // -3.0
round(-3.658) // -4.0
// ✅ 2. 올림(ceil)
// 참고: 소수점이 0보다 큰 경우 무조건 올림. 소수점을 올리면 정수부는 + 1. 소수점은 0이 됨.
ceil(5.123)  // 6.0
ceil(5.456)  // 6.0
ceil(5.512)  // 6.0
ceil(5.567)  // 6.0
ceil(5.657)  // 6.0
ceil(5.765)  // 6.0
ceil(-3.453) // -3.0
ceil(-3.658) // -3.0
// ✅ 3. 내림(floor)
//소수점을 무조건 내림.
floor(5.123)  // 5.0
floor(5.456)  // 5.0
floor(5.512)  // 5.0
floor(5.567)  // 5.0
floor(5.657)  // 5.0
floor(5.765)  // 5.0
floor(-3.453) // -4.0
floor(-3.658) // -4.0
// ✅ 4. 버림(trunc)
//trunc는 truncate의 약어. 정수부에는 아무런 영향을 주지 않고 소수점만 지움.
trunc(5.123)  // 5.0
trunc(5.456)  // 5.0
trunc(5.512)  // 5.0
trunc(5.567)  // 5.0
trunc(5.657)  // 5.0
trunc(5.765)  // 5.0
trunc(-3.453) // -3.0
trunc(-3.658) // -3.0
// ✅ 5. 원하는 자리에서 반올림, 올림, 내림
// 기본적으로 다른 메서드는 소수점 첫째자리 기준.
// Swift에서 기본으로 제공하는 함수가 없음. 따라서
// 소수점 4번째 자리에서 반올림할 경우 1,000을 곱하여 반올림 후 다시 1,000을 나눠주는 방식
// 소수점 6번째 자리에서 반올림할 경우 100,000을 곱하여 반올림 후 다시 100,000을 나눠주는 방식
// 4번째 자리에서 반올림 예시
let digit: Double = pow(10, 3) // 10의 3제곱
round(5.123123 * digit) / digit  // 5.123
round(5.456456 * digit) / digit  // 5.456
round(5.512512 * digit) / digit  // 5.513
round(5.567567 * digit) / digit  // 5.568
round(5.657657 * digit) / digit  // 5.658
round(5.765765 * digit) / digit  // 5.766
round(-3.453453 * digit) / digit // -3.453
round(-3.658658 * digit) / digit // -3.659
// ✅ 6. n번째 소수점 자르기 (소수점 아래 자리수 제한)
//원하는 자릿수만큼 필요한 경우를 위해 소수점을 n번째까지만 자르는 방법.String(format:_:...)을 사용. format "%.nf"이 중요. n번째까지 소수점을 제거한다는 format입니다. return값은 String.
let decimal = 5.1231283712893781293
String(format: "%.3f", decimal) // "5.123"
String(format: "%.6f", decimal) // "5.123128"
```

#### <span style="color:orange">**7. 제곱 관련**</span>
```swift
// ✅ pow(밑: Double, 지수: Double) 
let n = 2 
Int(pow(2.0, Double(n))) // 4
```

#### <span style="color:orange">**8. 입력값 처리 관련(문자열 쪼개기)**</span>
```swift
// ✅ 1. Input 값 받기
var input = readLine()!
var num = Int(readLine()!)!

// ✅ 2. 공백 있는 숫자 입력 받기
// "1 2 3 4 5"
// split - ReturnType: [SubString] 
var nums = readLine()!.split(separator: " ").map {Int($0)!} 
// components - ReturnType: [String] 
var nums = readLine()!.components(separatedBy: " "). map {Int($0)!}

// ✅ 3. 공백 없는 숫자를 배열로 입력 받기. 
// Int($0)! 보다 Int(String($0))! 가 속도가 더 빠름.
// "12345"
var nums = Array(readLine()!).map {Int(String($0))!} // String 변수 삽입해도 가능. 
var nums = Array(numbers).map {String($0)}

// ✅ 4. 문자열 자르기
var s = "HelloWorld" 
var firstIndex = s.index(s.startIndex, offsetBy: 0) 
var lastIndex = s.index(s.endIndex, offsetBy: -5) 
// "Hello" 
var v = s[firstIndex ..< lastIndex] 
// "Hello" 
var prefix = s.prefix(5)  //앞에서 5번째까지
// "World" 
var suffix = s.suffix(5) //뒤에서 5번째까지
// ✅ 5. 배열을 이용하여 문자열 자르고 합치기 (원하는 index까지)
var index = 4 
var hello = w[w.startIndex...index].map{String($0)} 
var world = w[index+1..<w.endIndex].map{String($0)} 
var helloWorld = hello.joined() + world.joined()

// ✅ 6. String 대문자 또는 소문자로 바꾸기
var s = "Hello, Swift"
s.lowercased() // hello, swift
s.uppercased() // HELLO, SWIFT
 
// ✅ 7. 문자열 치환 (원하는 문자열만 바꾸기)
import Foundation
let str = "Hello, Swift"
let replacedStr = str.replacingOccurrences(of: "Hello", with: "Hi") // Hi, Swift

// ✅ 8. 배열의 특정 구간만 새로운 배열에 넣기
// Subsequence(ArraySlice) -> Array
// nums 배열에서 특정 구간을 잘라서 res 배열에 넣는 상황
let nums = [0,1,2,3,4,5]
var res = [[Int]]()
// Subsequence 타입을 다시 Array로 캐스팅해야 res에 넣을 수 있음
res.append(Array(nums[nums.firstIndex(of: 3)!...])) 
// res -> [[3,4,5]]

```
#### <span style="color:orange">**9. 범위 체크**</span>
```swift
let n = 5, m = 6 
func checkBorder(_ r: Int, _ c: Int) -> Bool { 0..<n ~= r && 0..<m ~= c } print(checkBorder(3, 4)) // true 
print(checkBorder(4, 7)) // false
// ~= 연산자는 대상이 특정 범위에 속하는지 범위를 체크하는 연산자.
```

#### 참고자료
1. [[Swift] Date() 날짜구하기, DateFormatter() 다루기 오전, 오후 등등](https://formestory.tistory.com/6?category=820173)

참고
1. [[Swift] 소수점 다루기 반올림(round), 올림(ceil), 내림(floor) 등등](https://formestory.tistory.com/m/21?category=842283)
2. [[Swift] 알고리즘에 필요한 Tip 정리](https://thoonk.tistory.com/2)
3. [How to loop over arrays](https://www.hackingwithswift.com/articles/76/how-to-loop-over-arrays)