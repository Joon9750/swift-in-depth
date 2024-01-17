# Asynchronous error handling with Result

## This chapter covers
- Learning about the problems with Cocoa style error handling
- Getting an introduction to Apple's Result type
- Seeing how Result provides compite-time safety
- Preventing bugs involving forgotten callbacks
- Transforming data robustly with map, mapError, and flatMap
- Focusing on the happy path when handling errors
- Mixing throwing functions with Result
- Learning how AnyError makes Result less restrictive
- How to show intent with the Never type

## Why use the Result type?
chapter 6에서는 동기적 상황에서 에러를 핸들링하는 방법을 살펴봤다면 앞으로는 비동기 상황에서의 에러 핸들링을 살펴볼 것입니다.

스위프트에서 공식적으로 비동기 상황에서의 에러 핸들링 기법을 제공하지 않습니다.
Swift Package Manager에서 제공하는 Result 타입을 사용하게 됩니다.

**Result is like Optional, with a twist**

Result 타입은 옵셔널과 같은 열거형으로 구현부는 아래 코드와 같습니다.

```swift
public enum Result<Value, ErrorType: Swift.Error> {
  /// Indicates success with value in the associated object.
  case success(Value)

  /// Indicates failure with error inside the associated object.
  case failure(ErrorType)

  // ... The rest if left out for later
}
```

Result는 success(Value)와 failure(ErrorType) 두 개의 케이스를 가진 열거형입니다.
열거형이기 때문에 success 또는 failure 중 하나의 상태만 가지게 됩니다.
다시 말해 success와 failure가 동시에 참이거나 거짓일 경우는 없습니다.

또한 ErrorType은 Swift.Error 타입으로 제약되어 있습니다.
따라서 ErrorType에 들어오는 타입은 Error 타입을 반드시 채택해야 합니다.
그에 반해 success의 Value 타입에 경우 타입 제약이 없기 때문에 모든 타입이 Value로 들어올 수 있습니다.

옵셔널은 Value 또는 nil을 가지지만 Result는 Value 또는 Error를 가지게 됩니다.
failure의 경우 단순히 빈 값(nil)을 가지는 옵셔널과 달리 Error를 가지기 때문에 실패에 대한 맥락을 제공할 수 있습니다.

결과적으로 Result 타입은 success와 failure의 값(Value, ErrorType)을 모두 요구합니다.
우리는 에러가 발생할 때 Result 타입에 ErrorType을 넘겨 대응하고 정상적으로 동작한다면 Value를 넘겨 에러를 핸들링할 수 있습니다.

**Understanding the benefits of Result**

Result 타입의 이점을 느끼기 위해 먼저 Cocoa Touch-style의 에러 핸들링이 비동기 상황에서 보이는 문제점을 살펴봅시다.
해당 문제점을 Result 타입으로 해결해 봅시다.

아래에서 예시로 구현할 API는 iTunes Store에서 검색하는 기능을 가졌습니다.

먼저 Cocoa Touch-style의 코드를 살펴봅시다.

```swift
func callURL(with url: URL, completionHandler: @escaping (Data?, Error?) -> Void) {
  let task = URLSession.shared.dataTask(with: url, completionHandler: { (data, response, error) -> Void in
    completionHandler(data, error)
  })
  task.resume()
}

let url = URL(string: "https://itunes.apple.com/search?term=iron%20man")
```












