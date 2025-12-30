+++
title = "Fundamentals of type-driven code part1"
date = 2024-10-25
categories = ["type"]

[taxonomies]
tags = ["swift","type", "series"]

+++
Type Driven Code 의 기본 원리
<!-- more -->


# Fundamentals of type-driven code

타입 주도 코드의 기본 원리
- 원문  [Part 1 Fundamentals of type-driven code](https://swiftology.io/articles/tydd-part-1-fundamentals)


## 타입 주도 코드의 기본 원리

- 타입 주도 코드에 대한 직관을 개발하려면 익숙한 것에서 시작해야 합니다.
- Swift 프로그래머로서 이 코드를 문제가 있다고 즉시 인식할 수 있을 것입니다.

```swift
func main(number: Int?) {
  if number != nil {
    print("double:", 2 * number!) // 요기서 강제 언래핑
  } 
}
```

- 이 코드의 문제는 무엇일까요? 맞습니다, 여기서는 if let 옵셔널 언래핑 구문을 사용해야 합니다. 하지만 그 전에 이 코드를 면밀히 검토해 봅시다.
- 이 코드가 _정확히_ 왜 문제가 될까요? 옵셔널 값을 강제로 언래핑해야 했기 때문입니다. 
- 하지만 정말 그렇게 큰 문제일까요? 
- 이 구문은 Swift답지 않지만, 논리는 완벽히 타당하고 안전합니다. 
- 왜냐하면 `number`가 `nil`이 아님을 확인한 후에 접근하기 때문입니다. 
- 우리는 Objective-C에서도 오랫동안 비슷한 코드를 작성해 왔습니다!
- 문제는 강제 언래핑 자체가 아니라, `nil` 체크 후에도 강제 언래핑을 해야 하는 _이유_ 입니다. 
- 기본 문제를 더 명확히 하기 위해 지역성을 깨고 옵셔널 인자를 받는 외부 함수를 소개해 봅시다:

```swift
// Main.swift
func main(number: Int?) {
  if number != nil {
    save(number)		 
  }
}
```

```swift
// Save.swift
func save(_ number: Int?) {
  // ⚠️ We don't know 
  // if number is nil or not,
  // we must check ourselves!
if number != nil {
  UserDefaults.standard.set(
    number!, 
    foKey: "num"
  ) 
} else {
    print("Err: We don't want to store nils")
  }
}
```

> **통찰 💡:** 이 코드의 근본적인 문제는 _정보 손실_ 입니다. `number`가 `nil`이 아님을 확인하고 `main`의 `if` 문을 벗어나는 순간, 이 정보를 버리게 됩니다. 그리고 나중에 `save` 함수에서 다시 확인해야 합니다. 왜냐하면 `main`이 이미 수행한 확인에 대한 정보가 없기 때문입니다.

- 우리는 `number`가 `nil`이 아님을 나타내는 정보를 유지하고 이를 앞으로 전달하고자 합니다. 
- 그래서 `number`를 언랩하여 선택적이지 않은 값을 다음 함수에 전달함으로써 반복적인 확인의 필요성을 없앱니다.

```swift
func main(number: Int?) {
  if let number = number {
    save(number)
  }
}

func save(_ num: Int) {
  UserDefaults.standard.set(number, forKey: "num") 
}
```

- 물론 이걸 보여드리는 건 선택적 언래핑(optional unwrapping ) 과 non-optional types을 상기시키기 위해서만은 아닙니다
- 이 코드는 모든 Swift 프로그래머가 즉시 인식하고 수정할 수 있는 정보 손실의 예입니다.
-  그러나 이는 우리가 `if let` 구문을 선택적 값에 사용하도록 조건화되었기 때문입니다. 
- 동시에, 많은 Swift 프로그래머들은 화려한 구문 지원이 없는 타입을 다룰 때 다른 정보 손실의 사례를 인식하지 못합니다.
- 두 번째 예제를 고려해 봅시다.

```swift
func main(numbers: [Int]) {
  guard !numbers.isEmpty else {
    print("No numbers, sad")
    return
  }
  print("Cool numbers:", numbers)
}
```


- 이 코드는 전혀 문제가 없고 Swift답습니다. 
- 강제 언래핑 같은 의심스러운 것이 없으며, Swift 프로그래머의 머릿속에 경고를 울릴 만한 것도 없습니다. 
- 그러나 이 코드는 이전 것과 동일한 종류의 정보 손실을 가지고 있습니다. 
- 외부 함수의 도움으로 다시 이를 드러낼 수 있습니다

```swift
// Main.swift
func main(numbers: [Int]) {
  if !numbers.isEmpty {
    save(numbers)			 
  } else {
    print("No numbers, nothing to save...")
  }
}
```

```swift
// Save.swift
func save(_ numbers: [Int]) {
  // ⚠️ We don't know 
  // if numbers are empty or not,
  // and must check ourselves!
	if !numbers.isEmpty {
	  UserDefaults.standard.set(numbers, forKey: "numbers")
	} else {
	  print("No numbers, nothing to save...")
	}
}
```

- 배열이 비어 있지 않다는 것을 쉽게 기억할 수 있게 해주는 멋진 내장 구문이 없기 때문에, 
- 프로그래머들이 이러한 불편함을 무시하고 저품질의 코드를 확산시키는 경우가 매우 흔합니다.
- 이 예에서 정보 손실을 방지하기 위해, 배열의 첫 번째 요소를 명시적으로 캡처하여 비어 있지 않다는 증거로 삼고 다음 함수로 전달해야 합니다

```swift
func main(numbers: [Int]) {
  guard let first = numbers.first else { 
    print("No numbers, skipping...")
    return
  }
  let remaining = Array(numbers.dropFirst()) 
    save(first, remaining)	 
  }

func save(
  _ first: Int,
_ remaining: [Int]
) {
  UserDefaults.standard.set(
    CollectionOfOne(first) + remaining, 
    forKey: "numbers"
  )
}
```

- 이제 isEmpty 검사를 반복할 필요가 없습니다.
- 물론, 이렇게 모든 곳에서 배열을 구조 분해하고 재구성하는 것은 완전한 번거로움이기 때문에, 
- 우리는 이 과정을 `NonEmptyArray` 타입 내에 캡슐화할 수 있습니다:

```swift
struct NonEmptyArray<Element> {
  var first: Element
  var remaining: [Element]
  
  var arrayValue: [Element] { 
    CollectionOfOne(first) + remaining
  }
  
  init?(_ arrayValue: [Element]) {
    guard let first = arrayValue.first else {
      return nil
    }
    self.first = first
    remaining = Array(arrayValue.dropFirst())
  }
}
```


- NonEmptyArray 타입은 내부 구조를 활용해 비어있지 않다는 **증거**를 타입에 담습니다.. 
- 이 기술에 대한 자세한 내용은 제 기사 '[Greater type safety with Structural Typing in Swift](https://swiftology.io/articles/structural-typing/).  에서 배울 수 있습니다.
- 코드를 업데이트합시다:

```swift
func main(numbers: [Int]) {
  guard let nonEmptyNumbers = NonEmptyArray(numbers) else { 
  print("No numbers, skipping...")
  return
} 
  save(nonEmptyNumbers)	 
}

func save(_ numbers: NonEmptyArray<Int>) {
  UserDefaults.standard.set(
    numbers.arrayValue, 
    forKey: "numbers"
  )
}
```
   
>통찰💡: 정보의 보존과 전파를 위해 타입을 사용하세요. 예를 들어, 검증의 증거와 도메인 특화 불변성을 포함합니다

- Haskell, PureScript, Elm과 같은 언어는 표준 라이브러리에서 `NonEmpty` 컬렉션 타입을 제공.
- 이는 정말로 필수적이며, 이 시리즈를 다 읽고 나면 코드베이스 전반에서 비어 있지 않은 배열의 사용 사례를 발견하게 될 것입니다.
- Swift를 위해서는 Point-Free의 검증된 오픈 소스 라이브러리인 [swift-nonempty](https://github.com/pointfreeco/swift-nonempty).  를 추천합니다
