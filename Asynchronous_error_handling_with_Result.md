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

callURL(with: url) { (data, error) in
  if let error = error {
    print(error)
  } else if let data = data {
    let value = String(data: data, encoding: .utf8)
    print(value)
  } else {
    // error도 없고 data도 없는 상황
    // What goes here?
  }
}
```

위의 callURL 함수의 URLSession.dataTask의 작업이 끝났을 때 completionHandler가 호출됩니다.

작업에 시간이 걸리기 때문에 @escaping 클로저로 completionHandler를 선언했습니다. 
일반적인 클로저의 경우 함수의 실행 흐름을 탈출하지 않아, 함수가 종료되기 전에 무조건 실행 되어야 합니다.
하지만 @escaping 키워드가 붙은 클로저의 경우 비동기 상황에서 함수의 실행 흐름에 상관 없이 실행되는 클로저입니다.

추가적으로 파라미터로 받을 클로저가 있을 수도, 없을 때 @escaping 클로저에 옵셔널을 추가하려면 아래와 같이 ?를 추가하는 대신 @escaping 키워드를 지워야 합니다.
@escaping 키워드와 ?을 클로저에 함께 사용할 경우 @escaping 키워드를 지우라는 경고가 뜹니다.

```swift
@escaping ((Data?, Error?)-> Void)?  // 에러 발생

((Data?, Error?)-> Void)?
```

위와 같이 @escaping 키워드를 지우고 클로저에 ?(옵셔널) 선언만 해도 @escaping 클로저와 동일한 동작을 합니다.
결론적으로 함수 파라미터의 클로저가 Optional Type인 경우에는 자동으로 escaping으로 동작하기 때문에 추가적인 @escaping 키워드를 지워야 하는것입니다.

하지만 위의 옵셔널 클로저는 더 이상 클로저 타입이 아닙니다.
옵셔널을 붙이기 전의 @escaping (Data?, Error?) -> Void 클로저는 클로저 타입입니다.
하지만 클로저 타입에 옵셔널을 붙이면 더 이상 해당 클로저는 클로저 타입이 아닌 옵셔널 타입이 됩니다.
Int와 Optional(Int)가 타입이 다른 것과 같은 맥락입니다.

아래 링크을 참고해 봅시다.

https://babbab2.tistory.com/164

다시 callURL 함수의 코드를 살펴봅시다.

**Creating an API using Result**

callURL 함수의 호출부를 보면 error와 data를 모두 체크해야합니다. 심지어 error와 data 모두 없는 상황이 존재합니다. 
이론적으로 error와 data를 둘 다 받을 수 있고 못받을 수 있습니다.
여러 가능성이 존재하고 error와 data가 모두 들어오거나 들어오지 않는 이상한 상황을 제어하지 못합니다.
또한 위 코드에서는 error 핸들링에 있어 compile-time guarantee를 얻지 못합니다.

Result 타입은 열거형으로 error **또는** data를 가집니다.
열거형의 성격으로 이상한 상황을 포함한 여러 가능성을 success와 failure 중 한 가지로 줄일 수 있습니다.

또한 Result 타입을 사용하면 compile-time guarantee를 얻습니다.
Result 타입을 사용해 컴파일 타임에 response를 success(with a value) 또는 failure(with an error)로 강제할 수 있습니다. 

아래 코드는 위 Cocoa Touch-style API 호출을 Result 타입을 사용한 방식으로 고친 코드입니다.
아직 callURL 함수 안의 URLSession API 호출 코드는 고치지 않았습니다. 해당 부분은 아래에서 살펴봅시다.

```swift
enum NetworkError: Error {
  case fetchFailed(Error)
}

func callURL(with url: URL, completionHandler: @escaping (Result<Data, NetworkError>) -> Void) {
  let task = URLSession.shared.dataTask(with: url, completionHandler: { (data, response, error) -> Void in
    // ... details will be filled in shortly. 아래에서 살펴볼 예정입니다.
  })
  task.resume()
}

let url = URL(string: "https://itunes.apple.com/search?term=iron%20man")

callURL(with: url) { (result: Result<Data, NetworkError) in  // 컴파일 타임에 failure case가 가진 error의 타입을 알 수 있습니다.
  switch result {
  case .success(let data):
    let value = String(data: data, encoding: .utf8)
    print(value)
  case .failure(let error):  // Result 열거형의 패턴 매칭을 통해 에러를 처리합니다. 
    print(error)
  }
}
```

위의 코드와 같이 Result 타입을 사용하면 success와 failure의 패턴 매칭으로 compile-time safety를 얻게 됩니다.
더 이상 data와 error가 동시에 존재하거나 존재하지 않는 이상한 상황에 대응하지 않아도 됩니다.

Result 타입을 사용하며 중요한 부분은 Result의 value를 얻기 위해서는 error에 대한 처리도 필수적이고 반대 상황에서도 value에 대한 처리가 필수적입니다.

물론 Result의 failure 상황의 error를 핸들링하지 않고 무시할 수 있습니다.
if case let을 사용해 Result 타입의 success 케이스에만 대응하면 가능합니다.
하지만 Result의 value를 얻고 싶다면 컴파일러에게 error에 대한 처리도 알리는 것이 올바른 방법입니다.

if case let 구문 외에도 Result 타입의 failure(error)를 핸들링하고 싶지 않다면 **dematerialize** 함수를 사용할 수 있습니다.
dematerialize 함수는 Result 타입이 failure 일 때 failure의 이유를 무시할 수 있습니다.

아래 코드와 같이 dematerialize 함수를 사용해 Result 타입을 패턴 매칭 없이 value 값을 얻을 수 있습니다.

```swift
let value: Data? = try? result.dematerialize()
```

**Bridging from Cocoa Touch to Result**

이제는 callURL 함수 안의 URLSession API 호출 코드에 Result 타입을 사용해 봅시다.

```swift
URLSession.shared.dataTask(with: url, completionHandler: { (data, response, error) -> Void in ... }
```

URLSession.shared.dataTask가 리턴하는 data, response, error 데이터를 Result 타입으로 변환해야 합니다.

세 가지 데이터를 Result 타입으로 변환하기 위해 Result 타입에 custom init을 추가해야 합니다.
URLSession.shared.dataTask가 리턴하는 세 가지 데이터를 Result 타입으로 변환해야 

```swift
func callURL(with url: URL, completionHandler: @escaping (Result<Data, NetworkError>) -> Void)
```

위와 같은 callURL 함수의 선언부를 만족할 수 있습니다.

아래 코드는 Result 타입에 custom init을 추가하고 callURL 함수의 URLSession.shared.dataTask에서 리턴되는 세 가지 데이터를 Result 타입으로 변환하는 코드입니다.

```swift
publice enum Result<Value, ErrorType> {
  // ... 생략

  // custom init
  init(value: Value?, error: ErrorType?) {
    if let error = error {
      self = .failure(error)
    } else if let value = value {
      self = .success(value)
    } else {
      fatalError("Could not create Result")
    }
  }
}

func callURL(with url: URL, completionHandler: @escaping (Result<Data, NetworkErrork>) -> Void) {
  let task = URLSession.shared.dataTask(with: url, completionHandler: { (data, response, error) -> Void in
    let dataTaskError = error.map { NetworkError.fetchFailed($0) }
    let result = Result<Data, NetworkError>(value: data, error: dataTaskError)  // Result enum의 custom init으로 Result 타입 생성
    completionHandler(result)
  })
  task.resume()
}
```

모든 API가 value를 리턴하지 않을 수 있습니다.
Result<(), MyError> 또는 Result<Void, MyError>와 같이 () 또는 Void를 사용해 value 값을 가지지 않는 Result 타입을 만들 수 있습니다.

## Propagating Result

Let's make your API a bit higher-level so that instead of manually creating URLs, you can search for items in the iTunes Store by passing strings.

지금까지 위에서 다룬 NetworkError와 같이 lower-level 에러가 아닌 아래와 같은 higher-level 에러 SearchResultError를 다룰 것입니다. SearchResultError가 검색 기능의 추상적인 개념과 적합합니다.

SearchResultError 코드를 살펴봅시다.

```swift
enum SearchResultError: Error {
  case invalidTerm(String)  // when an URL can't be created
  case underlyingError(NetworkError)  // underlyingError cse carries the lower-level NetworkError for troubleshooting
  case invalidData  // when the raw data could not be parsed to JSON
}

search(term: "Iron man") { result: Result<[String: Any]>, SearchResultError> in
  print(result)
}
```

**Typealiasing for convenience**

search() 함수를 구현하기 전에 **typealias** 키워드를 사용해 Result 타입을 축약해 편리하게 사용할 수 있습니다.
typealias 키워드를 통해 Result 타입의 data나 error 타입을 고정할 수 있습니다.

예를 들어 Result<Value, SearchResultError> 타입을 SearchResult<Value>로 typealias 한다면 에러 타입을 SearchResultError 타입으로 고정하게 됩니다.
SearchResult<Value> 타입은 Value와 Error 타입 모두 제네릭 타입이었지만 이제는 Value만 제네릭 타입이 됩니다.

아래 코드로 살펴봅시다.

```swift
typealias SearchResult<Value> = Result<Value, SearchResultError>

let searchResult = SearchResult("Tony Stark")
print(searchResult)  // success("Tony Stark")
```

위 코드에서 SearchResult 타입의 에러 타입은 SearchResultError로 고정되었고 Value 타입은 고정되지 않았습니다.

JSON 타입도 typealias 키워드를 통해 만들어 봅시다.
[String: Any] 타입보다 JSON 타입이 읽기 쉬운 코드를 만듭니다.

```swift
typealias JSON = [String: Any]
```

typealias 키워드를 통해 SearchResult<JSON> 타입을 만들었습니다.
SearchResult<JSON> 타입의 실제 타입은 Result<[String: Any], SearchResultError> 타입입니다.

**The search function**

지금부터 iTunes Store에서 문자열로 검색하는 search() 함수를 구현해 봅시다.

search() 함수에서는 completionHandler로 최종적으로 SearchResult<JSON> 타입을 리턴하기 위해, data를 JSON으로 파싱하고 lower-level error인 NetworkError를 SearchResultError로 변환하는 과정까지 포함하고 있습니다. 

아래 코드로 살펴봅시다.

```swift
func search(term: String, completionHandler: @escaping (SearchResult<JSON>) -> Void) {
  // encodedString 변수는 옵셔널 타입입니다.
  let encodedString = term.addingPercentEncoding(withAllowedCharacters: .urlHostAllowed)
  // map을 사용해 옵셔널 언래핑을 미룰 수 있습니다.
  let path = encodedString.map { "https://itunes.apple.com/search?term=" + $0 }

  guard let url = path.flatMap(URL.init) else {
    completionHandler(SearchResult(.invalidTerm(term)))  // 1.
    return
  }

  callURL(with: url) { result in
    switch result {
    case .success(let data):
      if let json = try? JSONSerialization.jsonObject(with: data, options: []),
        let jsonDictionary = json as? JSON {
          let result = SearchResult<JSON>(jsonDictionary)  
          completionHandler(result)  // 2.
        } else {
          let result = SearchResult<JSON>(.invalidData)
          completionHandler(result)  // 3.
        }
    case .failure(let error):
      // lower-level error를 higher-level error로 변환합니다.
      let result = SearchResult<JSON>(.underlyingError(error))
      completionHandler(result)  // 4.
    }
  }
}
```

위의 코드에서는 여러 번의 CompletionHandler를 호출합니다. 
여러 번의 CompletionHandler 호출에 필요한 Result 타입 또한 여러 개를 생성해야 합니다.
이런 점에서 위의 코드는 boilerplate code를 가집니다.

이때 map, flatMap, flatMap을 사용해 단일 Result 타입의 Value와 Error를 변환해 한 번의 CompletionHandler로 search() 함수를 구현할 수 있습니다.

## Transforming values inside Result

옵셔널에 map을 사용해 옵셔널 언래핑을 미뤘던것처럼 Result 타입과 map도 함께 사용할 수 있습니다.
옵셔널을 매핑하듯이 Result를 매핑하여 변형할 수 있습니다.

매핑 없이 일반적인 경우 Result를 가졌을 때 Result 타입을 넘기고 변형한 이후 switch(패턴 매칭)를 통해 Result 내부의 값을 추출할 수 있습니다.

하지만 map을 사용하면 옵셔널을 매핑할 때 옵셔널 언래핑을 할 필요 없이 내부 값을 다루고 다시 옵셔널로 감쌌듯이, Result를 매핑하면 success 케이스의 경우 내부 value가 클로저로 들어가고 failure 케이스 경우 map 연산이 무시됩니다. 

따라서 위의 search() 함수에서 Result 타입의 String data를 JSON으로 변형할 때 map을 사용하면 효과적입니다.

map에 의해 Result 타입이 success 케이스의 경우 Result 타입의 Value를 변환하는 과정을 살펴봅시다.

1. You have result: with a value.
2. With map, you apply a function to the value inside a result.
3. Map rewraps the transformed value in a result.

이제는 Result 타입이 failure 케이스의 경우 map이 동작하는 과정을 살펴봅시다.

1. You have result: with an error.
2. Map does nothing with a failing result.
3. The failing result is still the same old failling result.

map은 failure 케이스의 Result 타입에는 동작하지 않고 success 케이스의 Result 타입에만 동작합니다.
하지만 **mapError**을 사용하면 map과 반대로 Result 타입이 failure 케이스일 때 error를 매핑하고 success 케이스의 경우 mapError가 동작하지 않습니다.

mapError가 success 케이스를 가진 Result 타입에 동작하는 과정을 살펴봅시다.

1. You have success result: with a value.
2. mapError does nothing with a successful result.
3. The successful result is still the same.

이번에는 mapError가 failure 케이스를 가진 Result 타입에 동작하는 과정을 살펴봅시다.

1. You have failure result: with a error.
2. With mapError, you apply a function to the error inside a result.
3. mapError rewraps the transformed error in a result.

다시 말해 map을 통해 Result 타입의 Value를 매핑하여 값을 변환하고 mapError로 Result 타입의 Error를 매핑하여 값을 변환할 수 있습니다.

With the power of both map and mapError combined, you can turn a Result<Data, NetworkError> into a Result<JSON, SearchResultError>, aka SearchResult<JSON>.

지금부터 map과 mapError를 통해 Result 타입의 데이터 변환하여 위의 보일러 플레이트 코드를 개선할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
func search(term: String, completionHandler: @escaping (SearchResult<JSON>) -> Void) {
  // ... 생략

  callURL(with: url) { result in
    let convertedResult: SearchResult<JSON> =
      result
          .map { (data: Data) -> JSON in
              guard let json = try? JSONSerialization.jsonObject(with: data, options: []),
                    let jsonDictionary = json as? JSON else {
                      return [:]
                  }
                return jsonDictionary
              }
              .mapError { (netwrokError
  }
}
```















