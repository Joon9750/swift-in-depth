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
                return [:]  // data를 json으로 변환 시 실패하면 빈 딕셔너리를 리턴합니다.
            }
            return jsonDictionary
          }
          .mapError { (networkError: NetworkError) -> SearchResultError in
            return SearchResultError.underlying(networkError)
          }
    completionHandler(convertedResult)
  }
}
```

위의 코드와 같이 map과 mapError를 통해 Result 타입 내의 value와 error를 변환해 마지막 Result를 completionHandler로 리턴하여 한 번의 completionHandler만 호출하도록 함수를 만들 수 있습니다.

Just like with optionals, you delay any error handling until you want to get the value out.

하지만 위의 코드에서 data를 json으로 변환 시 실패하면 빈 딕셔너리를 리턴하고 있습니다.
빈 딕셔너리를 리턴하는 대신 Error를 가지는 방식으로 개선하려 합니다.

이때 우리는 **faltMap**을 사용하여 개선할 수 있습니다.

**flatMapping over Result**

빈 딕셔너리를 리턴하는 대신 Error를 던지는 방식으로 개선할 수 있지만 Result 타입을 사용하는 것과 error throwing이 섞이면 이상할 수 있습니다. (뒤에서 Result 타입과 error throwing을 섞는 방식도 살펴볼 예정입니다.)

따라서 빈 딕셔너리를 리턴하는 대신 failure 케이스를 가진 Result를 리턴하는 방식으로 코드를 개선할 수 있습니다.

하지만 Result를 매핑한 map 클로저 내에서 빈 딕셔너리 대신 Result(failure)를 리턴하면 결과적으로 SearchResult<SearchResult<JSON>>과 같은 중첩 Result 타입이 만들어 집니다.

이때 중첩 Result를 flatMap을 통해 단일 Result로 평탄화 할 수 있습니다.
물론 Error 케이스를 가진 Result는 map과 동일하게 flatMap에서도 무시됩니다.

결과적으로 flatMap을 사용해 flatMap 클로저 내부에서 진행되는 data -> JSON 변환 과정에서 발생되는 에러를 Result 타입으로 리턴하여 중첩 Result 타입을 단일 Result 타입으로 리턴합니다.

flatMap이 success 케이스의 Result와 동작하는 과정을 살펴봅시다.
flatMap의 클로저에서 데이터를 변환하고 변환 실패 시 failure 케이스의 Result를 생성하고 성공 시 success 케이스의 Result를 생성합니다.

추가적으로 flatMap은 failure 케이스의 Result는 무시하게 됩니다.

1. You start with a successful result containing Data(x2) and with one result containing an error.
2. With flatMap, you apply a function to the value inside the result. This function will itself return a new result. (This new result could be successful and carry a value, or be a failure result containing an error. But if you start with a result containing an error, any flatMap action is ignored.)
3. You end up with a nested result. (If you start with an error, then nothing is transformed or nested.)
4. The nested result is flattened to a regular result. (If you start wih an error, nothing happened and the result remains the same.

이제는 위에서 데이터 변환 실패 시 빈 딕셔너리를 생성하는 코드를 빈 딕셔너리가 아닌 failure 케이스의 Result를 생성하도록 고쳐봅시다.

아래 코드는 데이터 변환 실패 시 failure 케이스의 Result를 생성하고 이후 중첩된 Result 타입을 flatMap으로 평탄화하는 코드입니다.

```swift
func search(term: String, completionHandler: @escaping (SearchResult<JSON>) -> Void) {
  // ... 생략

  callURL(with: url) { result in
    let convertedResult: SearchResult<JSON> =
      result
          .mapError { (networkError: NetworkError) -> SearchResultError in
            return SearchResultError.underlying(networkError)
          }          
          .flatMap { (data: Data) -> SearchResult<JSON> in  // 리턴되는 타입이 JSON에서 SearchResult<JSON>으로 수정됐습니다.
            guard let json = try? JSONSerialization.jsonObject(with: data, options: []),
              let jsonDictionary = json as? JSON else {
                // data를 json으로 변환 시 실패하면 빈 딕셔너리를 리턴하지 않고 failure 케이스를 가진 Result 타입을 리턴합니다.
                return SearchResult(.invalidData)  
            }
            return SearchResult(jsonDictionary)
          }
    completionHandler(convertedResult)
  }
}
```

위에서 이야기 했듯이 Result 타입의 Error는 flatMap으로 변환할 수 없습니다.
Result 타입의 Error는 mapError로 변환 가능합니다.

## Mixing Result with throwing functions

지금까지는 Result 타입의 mapping, flatmapping 연산 안에서(클로저 안에서) Error를 던지는 함수를 호출하는 방식을 피했습니다.
지금부터 Result 타입의 mapping, flatmapping 연산 안에서 Error를 던지는 함수를 추가해보고 마지막에는 Error를 던지지 않고 파이프라인 방식으로 Result 타입의 전달만으로 에러를 핸들링해봅시다.

search 함수의 flatMap 클로저 안에서 Data -> JSON으로 변환할 때 Error를 던지는 parseData 함수를 사용해봅시다.
parseData 함수는 JSON으로 데이터 변환에 실패 했을 때 ParsingError 에러를 던집니다.

먼저 ParsingError 열거형과 parseData 함수를 코드로 살펴봅시다.

```swift
enum ParsingError: Error {
  case couldNotParseJSON
}

// 함수에 throws 키워드를 추가해 에러를 던질 수 있음을 알립니다.
func parseData(_ data: Data) throws -> JSON {
  guard let json = try? JSONSerialization.jsonObject(with: data, options: []),
        let jsonDictionary = json as? JSON else {
          throw ParasingError.couldNotParseJSON
        }
  return jsonDictionay
}
```

error를 던지는 함수가 error를 던졌을 때 이를 failure 케이스의 Result 타입으로 변환하여 대응할 수 있습니다.

Result의 생성자(init)로 error를 던지는 함수를 try 키워드와 함께 넣습니다.
이때 함수가 error를 던지는 여부에 따라 Result가 success 케이스로 생성될지 failure 케이스로 생성될지 결정됩니다.
함수가 error를 던졌을 때 failure 케이스의 Result 타입을 생성합니다.

아래 코드로 살펴봅시다.

```swift
let searchResult: Result<JSON, SearchResultError> = Result(try parseData(data))
```

위의 코드는 한 가지 문제가 있습니다.

parseData 함수가 던질 에러 타입을 런타임에서야 알 수 있고 만약 parseData 함수가 SearchResultError 타입의 에러를 던지지 않는다면 다른 타입의 에러에 추가적으로 대응해야 합니다.

이를 해결하기 위해서 아래 코드와 같이 **do-catch** 구문이 필요합니다.

```swift
do {
  let searchResult: Result<JSON, SearchResultError> = Result(try parseData(data))
} catch {
  print(error)  // ParsingError.couldNotParseData
  // 결국 SearchResultError와 다른 에러 타입이 리턴되면 catch에서 Result(.invalidData(data)를 리턴해야 합니다. 
  let searchResult: Result<JSON, SearchResultError> = Result(.invalidData(data))
}
```

throwing function을 Result 타입으로 변환했다면 이 방법으로 search 함수를 완성해 봅시다.

아래 코드로 살펴봅시다.

```swift
func search(term: String, completionHandler: @escaping (SearchResult<JSON>) -> Void) {
  // ... 생략

  callURL(with: url) { result in
    let convertedResult: SearchResult<JSON> =
      result
          .mapError { SearchResultError.underlyingError($0) }          
          .flatMap { (data: Data) -> SearchResult<JSON> in
            do {
              let searchResult: SearchResultError<JSON> = Result(try parseData(data))
              return searchResult  
            } catch {
              // parseData 함수에서 던지는 에러를 SearchResultError로 변환합니다.
              return SearchResult(.invalidData(data))
            }
          }
    completionHandler(convertedResult)
  }
}
```

**Weaving errors through a pipeline**

위에서 살펴본 map, flatMap, mapError를 파이프라인 형식으로 구성하여 에러를 던지는 함수 없이도 에러를 핸들링 할 수 있습니다.
Result 타입이 파이프라인을 통과하고 나서 최종적으로 패턴 매칭을 통해 에러를 핸들링합니다.

flatMap은 에러가 발생한 상황에 프로그램의 흐름을 Error path로 바꾸지만, map은 항상 Happy path에 프로그램의 흐름을 유지시켜줍니다.
flatMap이 사용하게 된 이유가 Error를 던질 상황에 Result 타입을 리턴하기 위함으로 에러가 발생한 상황에 프로그램의 흐름을 Error path로 바꾸는 것입니다.

에러를 던지는 함수 없이 파이프라인 방식으로 에러를 핸들링한 아래 코드를 살펴봅시다.

```swift
func search(term: String, completionHandler: @escaping (SearchResult<JSON>) -> Void) {
  // ... 생략

  callURL(with: url) { result in
    let convertedResult: SearchResult<JSON> =
      result
          // Transform error type to SearchResultError
          .mapError { (networkError: NetworkError) -> SearchResultError in
            return SearchResultError.underlying(networkError)
          }
          // Parse Data to JSON, or return SearchResultError              
          .flatMap { (data: Data) -> SearchResult<JSON> in
            // 생략
          }
          // validate Data
          .flatMap { (json: JSON) -> SearchResult<JSON> in
            // 생략
          }
          // filter value
          .map { (json: JSON) -> [JSON] in
            // 생략
          }
          // Save to database
          .flatMap { (mediaItems: [JSON]) -> SearchResult<JSON> in
            // 생략
            database.store(mediaItems)
          }
    completionHandler(convertedResult)
  }
}
```

map과 flatMap은 Result 타입이 failure일 경우 무시합니다.
flatMap에서 특정 에러로 인해 failure인 Result가 리턴된다면 이후의 mapping이나 flatmapping은 무시됩니다.

## Multiple errors inside of Result

지금까지는 에러가 발생했을 때 Result 타입의 failure 케이스가 단일 에러 타입을 가지도록 했습니다.
위에서 다뤘던 Result 타입의 단일 에러 타입은 SearchResultError입니다.

에러 타입을 하나로 특정하는 방법은 좋습니다. 
하지만 다뤄야할 에러 타입이 너무 다양하다면 초기 프로젝트 시기에는 모든 에러 타입을 정확한 단일 에러 타입으로 확정하기 부담스러울 수 있습니다.

이때 우리는 SPM(Swift Package Manager)에서 제공하는 제네릭 타입인 **AnyError**를 사용하여 에러의 타입을 런타임에 알 수 있도록 하여 정확한 에러 타입을 선언할 부담을 덜어줍니다.
AnyError는 Result 내부의 Error에 들어가는 모든 에러 타입에 대응합니다. 
Result 내부의 에러는 Error 타입을 따르기 때문에 AnyError 또한 Error를 따르는 모든 에러에 대응할 수 있습니다.(?)

예를 들어 Result<String, AnyError>와 같이 사용될 수 있습니다.

AnyError를 가진 Result 타입을 생성하는 방법은 두 가지가 있습니다.

첫 번째로 일반적인 Error를 AnyError로 변환 후 Result 타입에 변환된 AnyError를 넣는 방식입니다.

이때 AnyError(일반적인 Error)와 같은 형식으로 AnyError의 생성자에 일반적인 Error를 넣어 AnyError를 생성할 수 있습니다.
아래 코드와 같습니다.

```swift
enum PaymentError: Error {
  case amountTooLow
  case insufficientFunds
}

let error: AnyError = AnyError(PaymentError.amountTooLow)
let result: Result<String, AnyError> = Result(error)

let directResult: Result<String, AnyError> = Result(PaymentError.amountTooLow)
```

result와 같이 AnyError를 Result 생성자에 넣을 수 있고 directResult와 같이 이미 자료형을 Result<String, AnyError>로 선언했다면 일반적인 Error를 Result 생성자에 바로 넣을 수 있습니다.
자료형이 Result<String, AnyError>이기 때문에 Result의 생성자로 들어온 일반적인 Error도 자동으로 AnyError 타입으로 형변환됩니다.

두 번째 방법은 에러를 던지는 함수를 Result의 생성자에 넣는 방법입니다.
아래 코드로 살펴봅시다.

```swift
let otherResult: Result<String, AnyError> = Result(anyError: { () throws -> String in
  throw PaymentError.insufficientFunds
}
```

AnyError는 에러 타입을 런타임에 알게 됩니다. 따라서 정확한 에러 타입을 알 필요 없을 때 AnyError를 Result 타입과 함께 사용할 수 있습니다.

파이프라인에서 각 단계별로 리턴되는 에러 타입이 다양할 경우, AnyError를 사용해 리턴되는 에러들을 특정 에러 타입으로 변환할 부담을 줄여줍니다.

아래 코드로 AnyError의 쓰임을 살펴봅시다.

Imagine that you're creating a function to transfer money, called processPayment. 
You can return different types of errors in each step, which relieves you of the burden of translating different errors to one specific type.

```swift
func processPayment(fromAccount: Account, toAccount: Account, amoutInCents: Int, completion: @escaping (Result<String, AnyError>) -> Void) {
  guard amountInCents > 0 else {
    completion(Result(PaymentError.amountTooLow))
    return
  }

  guard isValid(toAccount) && isValid(fromAccount) else {
    completion(Result(AccountError.invalidAccount))
    return
  }

  // Process payment

  moneyAPI.transfer(amountInCents, from: fromAccount, to: toAccount) { (result: Result<Data, AnyError>) in
    let response = result.mapAny(parseResponse)  // mapAny!!
    completion(response)
  }
}
```

AnyError를 가진 Result 타입을 사용하면 Result 타입에 mapAny를 사용 가능합니다.

The **mapAny** method works similarly to map, except that it can accept any throwing function.

mapAny 안의 함수에서 error를 던지면 해당 에러를 AnyError로 감싸서 리턴하게 됩니다.

mapAny를 통해 에러 핸들링(catch) 없이 map으로 에러를 던지는 함수를 넘길 수 있습니다.

mapAny는 flatMap 처럼 클로저 안에서 새로운 Result 타입을 리턴하여 Result의 에러 타입을 변환할 수는 없습니다.
그저 mapAny에 속하는 함수가 에러를 리턴하면 해당 에러를 AnyError로 감싸서 리턴 시켜줍니다.

따라서 map은 연산에 속하는 함수가 에러를 던지지 못하는 함수임을 나타내고, mapAny는 에러를 던질 수 있는 함수임을 나타냅니다!

The difference between map and mapAny is that map works on all Result types, but it doesn't catch errors from throwing functions.
In contrast, mapAny works on both throwing and nonthrowing functions, but it's available only on Result types containing AnyError.

결과적으로 mapAny를 통해 Result의 value를 매핑하고 에러가 발생할 경우 AnyError로 감싼 에러를 가진 Result 타입을 얻을 수 있습니다.

**Matching with AnyError**

AnyError 속 실제 에러 타입을 꺼내기 위해서는 underlyingError 프로퍼티를 사용해야 합니다.
AnyError 속 에러에 따라 failure 케이스를 매칭하는 코드를 살펴봅시다.

```swift
processPayment(fromAccount: from, toAccount: to, amountInCents: 100) { (result: Result<String, AnyError>) in
  switch result {
  case .success(let value): print(value)
  case .failure(let error) where error.underlyingError is AccountError:
    print("Account error")
  case .failure(let error):
    print(error)
  }
}
```

프로젝트 초기가 아니고 시간적 여유가 있다면 AnyError가 아니라 일반적 error를 사용해 컴파일 타임에 이점을 얻을 수 있도록 합시다.

AnyError를 사용하면 더욱 유연성있는 코드를 만듭니다.

하지만 Result의 error 타입을 컴파일 타임에 알 수 있도록 했던 장점을 AnyError의 사용으로 인해 런타임에 알게 된다는 단점도 존재합니다.

## Impossible failure and Result

Result 타입을 가지는 프로토콜을 따를 때, 프로토콜을 따르는 타입에서 Result의 failure가 발생하는 상황이 절대 일어나지 않는 경우가 있습니다. (never fail)

Result 타입은 success와 failure 케이스를 갖지만, failure 케이스가 발생할 가능성이 없을 때 빈 Error 타입을 만들어서 대응하거나 Never 타입으로 대응할 수 있습니다.

먼저 Result 타입을 가지는 프로토콜을 코드로 구현해 봅시다.

```swift
protocol Service {
  associatedtype Value
  associatedtype Err: Error
  func load(complete: @escaping (Result<Value, Err>) -> Void)
}
```

위의 Service 프로토콜을 따르는 SubscriptionsLoader 타입에서 load 함수는 항상 성공합니다.

이때 Error 타입을 따르는 빈 열거형을 만들어 Result 속 Error로 넣어야 합니다.

아래 코드는 Service 프로토콜을 따르는 SubscriptionsLoader 클래스를 구현한 코드입니다. 

이때 SubscriptionsLoader 클래스의 load 함수는 항상 성공하는 함수이기 때문에 Result<Value, Err> 타입으로 리턴되는 Error에는 빈 열거형을 넣게 됩니다.

```swift
struct Subscription {
  // ...details omitted
}

enum BogusError: Error {}  // 빈 열거형

final class SubscriptionsLoader: Service {
  func load(complete: @escaping (Result<[Subscription], BogusError>) -> Void) {
    // ...load data. Always succeeds
    let subscriptions = [Subscription(), Subscription()]
    complete(Result(subscriptions))
  }
}
```

BogusError는 빈 열거형이기 때문에 인스턴스화 할 수 없습니다.

컴파일러는 Error 케이스에 빈 열거형(BogusError)을 넣으면 Result 타입의 failure 케이스를 switch case 매칭할 수 없습니다.

물론 아래 코드와 같이 failure 케이스 매칭 없이, success 케이스만 case 매칭할 수 있습니다. 

```swift
let subscriptionsLoader = SubscriptionsLoader()
subscriptionsLoader.load { (result: Result<[Subscription], BogusError>) in
  switch result {
  case .success(let subscriptions): print(subscriptions)
  // You don't need .failure
  }
}
```

이처럼 에러가 발생하지 않는 상황에서 빈 열거형을 Error 타입으로 넣는 방식의 장점은 열거형의 케이스를 줄일 수 있고 코드를 깔끔하게 만듭니다.

하지만 빈 열거형을 Error 타입으로 넣는 방식은 공식적인 방식은 아닙니다.

**Never** 타입이 빈 열거형을 대신하는 공식적인 방식입니다.

Never 타입은 컴파일러에게 특정 경로(케이스)로 연결되지 않는다는 사실을 알립니다.
다시 말해 불가능한 경로를 나타냅니다.

아래 코드로 Never 타입을 살펴봅시다.

```swift
func crashAndBurn() -> Never {
  fatalError("Something very, very bad happened")
}
```

위의 crashAndBurn 함수는 Never 타입을 리턴하기 때문에 절대 값을 리턴하지 않는 함수임을 보장합니다.

Never 타입의 구현부는 아래 코드와 같습니다.

```swift
public enum Never {}
```

앞에서 봤던 빈 열거형 BogusError를 Never 타입으로 대체 가능합니다.
물론 Result 타입의 Error로 Never 타입을 사용하려면 Never 타입을 Error 타입을 따르도록 해야 합니다.

아래 코드는 BogusError를 Never 타입으로 고친 코드입니다.

```swift
extension Never: Error {}

final class SubscriptionsLoader: Service {
  func load(complete: @escaping (Result<[Subscription], BogusError>) -> Void) {
    // ...load data. Always succeeds
    let subscriptions = [Subscription(), Subscription()]
    complete(Result(subscriptions))
  }
}
```














