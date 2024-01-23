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
chapter 6에서 동기적 상황의 에러를 핸들링하는 방법을 살펴봤다면 앞으로는 비동기 상황에서의 에러 핸들링을 살펴볼 것입니다.

스위프트에서는 공식적으로 비동기 상황에서의 에러 핸들링 기법을 제공하지 않습니다.

대신 Swift Package Manager에서 제공하는 Result 타입을 사용하게 됩니다.

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

열거형이기 때문에 success **또는** failure 중 하나의 상태만 가지게 됩니다.
다시 말해 success와 failure가 동시에 참이거나 거짓일 가능성을 차단합니다.

또한 Result의 ErrorType은 Swift.Error 타입으로 제약되어 있습니다.
따라서 ErrorType에 들어오는 타입은 반드시 Error 타입을 채택해야 합니다.

그에 반해 success의 Value 타입에 경우 타입 제약이 없기 때문에 모든 타입이 Value로 들어올 수 있습니다.

옵셔널은 Value 또는 nil을 가지지만, Result는 Value 또는 Error를 가지게 됩니다.

failure의 경우 단순히 빈 값(nil)을 가지는 옵셔널과 달리 Error를 가지기 때문에 실패에 대한 맥락을 제공할 수 있습니다.

결과적으로 Result 타입은 이후 패턴 매칭을 통해 success와 failure에 대한 대응을 모두 구현해야 합니다.

에러가 발생할 때 Result 타입에 ErrorType을 넘겨 대응하고 정상적으로 동작한다면 Value를 넘겨 에러를 핸들링할 수 있습니다.

**Understanding the benefits of Result**

Result 타입의 이점을 느끼기 위해 먼저 Cocoa Touch-style의 에러 핸들링이 비동기 상황에서 보이는 문제점을 살펴봅시다.

Cocoa Toach-style 에러 핸들링의 문제점을 Result 타입으로 해결해 봅시다.

지금부터 구현할 API는 iTunes Store에서 특정 url로 검색하는 기능을 하는 API입니다.

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
    // error도 없고 data도 없는 상황 - 말도 안되는 상황에 대한 처리를 해야합니다.
    // What goes here?
  }
}
```

위의 callURL 함수의 URLSession.dataTask의 작업이 끝났을 때 completionHandler가 호출됩니다.
URLSession.dataTask 작업에 시간이 걸리기 때문에 @escaping 클로저로 completionHandler를 선언했습니다. 

callURL 함수의 호출부를 보면 error와 data를 모두 체크해야 합니다. 

Cocoa Touch-style 방식은 callURL 함수에서 이론적으로 error와 data를 둘 다 받을 수 있고 못받을 수도 있습니다. error와 data 모두 없는 말도 안되는 상황까지 대응해야 합니다.

또한 Cocoa Touch-style 방식은 에러 핸들링에 있어 컴파일 타임 이점을 얻지 못합니다.

하지만 Result 타입은 열거형으로 error **또는** data를 가집니다.

열거형의 성격으로 success와 failure가 동시에 참이거나 거짓인 상황을 success와 failure 중 한 가지로 줄일 수 있습니다.

Result 타입을 사용해 컴파일 타임에 response를 success(with a value) 또는 failure(with an error)로 강제할 수 있습니다. 

아래 코드는 위 Cocoa Touch-style API 호출을 Result 타입을 사용한 방식으로 고친 코드입니다.

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

callURL(with: url) { (result: Result<Data, NetworkError) in
  // 컴파일 타임에 failure case가 가진 error의 타입을 알 수 있습니다.
  switch result {
  case .success(let data):
    let value = String(data: data, encoding: .utf8)
    print(value)
  case .failure(let error):  // Result 열거형의 패턴 매칭을 통해 에러를 처리합니다. 
    print(error)
  }
}
```

아직 callURL 함수 안의 URLSession API 호출 코드는 고치지 않았습니다. 
해당 부분은 이후에 살펴보고 지금은 Result 타입의 이점에 집중합시다.

위의 코드와 같이 Result 타입을 사용하면 success와 failure의 패턴 매칭으로 컴파일 타임 안전성을 얻게 됩니다.

더 이상 data와 error가 동시에 존재하거나 존재하지 않는 이상한 상황에 대응하지 않아도 됩니다.

Result 타입의 중요한 부분은 Result의 value를 얻기 위해서는 error에 대한 처리까지 필수적으로 컴파일 단계에서 강제하여 컴파일 타임 안전성을 얻게 됩니다.

물론 Result의 failure 상황의 error를 핸들링하지 않고 무시할 수 있습니다.

if case let을 사용해 Result 타입의 success 케이스에만 대응하면 가능합니다.
하지만 Result의 value를 얻고 싶다면 error에 대한 처리까지 구현하는 것이 올바른 방법입니다.

if case let 구문 외에도 Result 타입의 failure(error)를 핸들링하고 싶지 않다면 **get** 함수를 사용할 수 있습니다.
get 함수는 Result 타입이 failure 일 때 failure의 이유를 무시할 수 있습니다.

아래 코드와 같이 get 함수를 사용해 Result 타입을 패턴 매칭 없이 value 값을 얻을 수 있습니다.

```swift
let integerResult: Result<Int, Error> = .success(5)
do {  
    let value = try integerResult.get()
    print("The value is \(value).")
} catch {
    print("Error retrieving the value: \(error)")
}
// Prints "The value is 5."
```

**Bridging from Cocoa Touch to Result**

이제는 callURL 함수 안의 URLSession API 호출 코드에 Result 타입을 사용해 코드를 개선해봅시다.

```swift
URLSession.shared.dataTask(with: url, completionHandler: { (data, response, error) -> Void in ... }
```

URLSession.shared.dataTask가 리턴하는 data, response, error 세 가지 데이터를 Result 타입으로 변환해야 합니다.

세 가지 데이터를 Result 타입으로 변환하기 위해 Result 타입에 custom init을 추가해야 합니다.
URLSession.shared.dataTask가 리턴하는 세 가지 데이터를 Result 타입으로 변환해야 아래와 같은 callURL 함수의 completionHandler를 만족할 수 있습니다.

```swift
func callURL(with url: URL, completionHandler: @escaping (Result<Data, NetworkError>) -> Void)
```

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

func callURL(with url: URL, completionHandler: @escaping (Result<Data, NetworkError>) -> Void) {
  let task = URLSession.shared.dataTask(with: url, completionHandler: { (data, response, error) -> Void in
    let dataTaskError = error.map { NetworkError.fetchFailed($0) }
    let result = Result<Data, NetworkError>(value: data, error: dataTaskError)  // Result enum의 custom init으로 Result 타입 생성
    completionHandler(result)
  })
  task.resume()
}
```

위의 URLSession.shared.dataTask에서는 data를 리턴하고 있지만, 모든 API가 항상 value를 리턴하는것은 아닙니다.

이때 Result<(), MyError> 또는 Result<Void, MyError>와 같이 () 또는 Void를 사용해 value 값을 가지지 않는 Result 타입을 만들 수 있습니다.
만약 URLSession.share.dataTask가 data를 리턴하지 않는 API라면 callURL 함수의 completionHandler에서 @escaping(Result<Data, NetworkError>) -> Void가 아닌 @escaping(Result<(), NetworkError>) -> Void로 선언할 수 있습니다.

## Propagating Result

URL을 전달하는 callURL 함수 대신 문자열을 전달하여 iTunes Store에서 항목을 검색할 수 있도록 API를 좀 더 높은 수준으로 만들어 보겠습니다.

지금까지 다룬 NetworkError와 같이 저차원 에러가 아닌 고차원 에러(SearchResultError)를 다룰 것입니다. 
NetworkError와 같은 저차원 에러보다 SearchResultError가 검색 기능의 추상적인 개념과 보다 적합하게 때문입니다.

고차원 에러인 SearchResultError 코드를 살펴봅시다.

```swift
enum SearchResultError: Error {
  case invalidTerm(String)  // when an URL can't be created
  case underlyingError(NetworkError)  // underlyingError can carries the lower-level NetworkError for troubleshooting
  case invalidData  // when the raw data could not be parsed to JSON
}

search(term: "Iron man") { result: Result<[String: Any]>, SearchResultError> in
  print(result)
}
```

**Typealiasing for convenience**

search() 함수를 구현하기 전에 **typealias** 키워드로 Result 타입을 축약해 편리하게 사용합시다.

typealias 키워드를 통해 Result 타입의 data나 error의 타입을 고정할 수 있습니다.

예를 들어 Result<Value, SearchResultError> 타입을 SearchResult<Value>로 typealias 한다면 에러 타입을 SearchResultError 타입으로 고정하게 됩니다.
SearchResult<Value> 타입은 Value와 Error 타입 모두 제네릭 타입이었지만 이제는 Value만 제네릭 타입이 됩니다.

아래 코드로 살펴봅시다.

```swift
// 에러 타입이 SearchResultError로 고정됩니다.
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

결과적으로 typealias 키워드를 통해 SearchResult<JSON> 타입을 만들었습니다.

SearchResult<JSON> 타입의 실제 타입은 Result<[String: Any], SearchResultError> 타입인 사실을 기억해야 합니다.

**The search function**

이제 본격적으로 문자열을 전달하여 iTunes Store에서 항목을 검색하는 search 함수를 구현해봅시다.

search() 함수에서는 completionHandler로 SearchResult<JSON> 타입을 리턴하기 위해, data를 JSON으로 파싱하고 저차원 에러 NetworkError를 고차원 에러 SearchResultError로 변환하는 과정을 포함하고 있습니다.

아래 코드로 살펴봅시다.

```swift
func search(term: String, completionHandler: @escaping (SearchResult<JSON>) -> Void) {
  // encodedString 변수는 옵셔널 타입입니다.
  let encodedString = term.addingPercentEncoding(withAllowedCharacters: .urlHostAllowed)
  // map을 사용해 옵셔널 언래핑을 딜레이합니다.
  let path = encodedString.map { "https://itunes.apple.com/search?term=" + $0 }

  guard let url = path.flatMap(URL.init) else {
    completionHandler(SearchResult(.invalidTerm(term)))  // 1. CompletionHandler 호출
    return
  }

  callURL(with: url) { result in
    switch result {
    case .success(let data):
      if let json = try? JSONSerialization.jsonObject(with: data, options: []),
        let jsonDictionary = json as? JSON {
          let result = SearchResult<JSON>(jsonDictionary)  
          completionHandler(result)  // 2. CompletionHandler 호출
        } else {
          let result = SearchResult<JSON>(.invalidData)
          completionHandler(result)  // 3. CompletionHandler 호출
        }
    case .failure(let error):
      // lower-level error를 higher-level error로 변환합니다.
      let result = SearchResult<JSON>(.underlyingError(error))
      completionHandler(result)  // 4. CompletionHandler 호출
    }
  }
}
```

위의 코드에서는 네 번의 CompletionHandler를 호출합니다. 
네 번의 CompletionHandler 호출에 필요한 Result 타입도 네 개가 필요합니다.

이런 점이 위의 코드의 boilerplate code라고 할 수 있습니다.

이때 **map, flatMap, flatMap**을 사용해 단일 Result 타입의 조작으로(Value와 Error를 변환) 한 번의 CompletionHandler 호출을 가진 함수로 구현할 수 있습니다.

## Transforming values inside Result

옵셔널에 map을 사용해 옵셔널 언래핑을 미뤘던것처럼 Result 타입과 map도 함께 사용할 수 있습니다.
옵셔널을 매핑하듯이 Result를 매핑하여 변형할 수 있습니다.

매핑 없이 Result를 사용할 때, Result 타입을 전달하며 변형한 이후 switch(패턴 매칭)를 통해 Result 내부의 값을 추출할 수 있습니다.

하지만 map을 사용하여 옵셔널을 매핑할 때 옵셔널 언래핑 없이 내부 값을 다루고 다시 옵셔널로 감쌌듯이, Result를 매핑하면 success 케이스의 경우 내부 value가 클로저로 들어가고 failure 케이스 경우 map 연산이 무시됩니다. 
이후 내부 value는 다시 Result으로 감싸서 리턴됩니다.

위의 search 함수에서 Result 타입의 data를 JSON으로 변환할 때 map을 사용해 여러 번의 CompletionHandler 호출을 단일 호출 방식으로 고쳐봅시다.

success 케이스인 Result 타입을 매핑할 때의 과정을 먼저 살펴봅시다.

1. **You have result: with a value.**
2. **With map, you apply a function to the value inside a result.**
3. **Map rewraps the transformed value in a result.**

두 번째로 failure 케이스인 Result 타입을 매핑할 때의 과정을 살펴봅시다.

1. **You have result: with an error.**
2. **Map does nothing with a failing result.**
3. **The failing result is still the same old failling result.**

map은 failure 케이스의 Result 타입에는 동작하지 않고 success 케이스의 Result 타입에만 동작합니다.

하지만 **mapError**을 사용하면 map과 반대로 Result 타입이 failure 케이스일 때 에러를 매핑하고 success 케이스의 경우 mapError가 동작하지 않습니다.

success 케이스인 Result 타입에 mapError가 동작하는 과정을 살펴봅시다.

1. **You have success result: with a value.**
2. **mapError does nothing with a successful result.**
3. **The successful result is still the same.**

두 번째로 failure 케이스인 Result 타입에 mapError가 동작하는 과정을 살펴봅시다. 

1. **You have failure result: with a error.**
2. **With mapError, you apply a function to the error inside a result.**
3. **mapError rewraps the transformed error in a result.**

**다시 말해 map을 통해 Result 타입의 Value를 매핑하여 값을 변환하고, mapError로 Result 타입의 Error를 매핑하여 값을 변환할 수 있습니다.**

map과 mapError의 기능을 결합하면 Result<Data, NetworkError>를 SearchResult<JSON>이라고도 불리는 Result<JSON, SearchResultError>로 바꿀 수 있습니다.

지금부터 map과 mapError의 기능을 결합해 Result 타입의 데이터 변환하여 위의 보일러 플레이트 코드를 개선합시다.

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
              // data를 json으로 변환 시 실패하면 빈 딕셔너리를 리턴합니다.
              return [:] 
            }
            return jsonDictionary
          }
          .mapError { (networkError: NetworkError) -> SearchResultError in
            return SearchResultError.underlying(networkError)
          }
    // 단일 Result를 조작하여 completionHandler를 한 번만 호출하도록 개선합니다.
    completionHandler(convertedResult)
  }
}
```

위의 코드와 같이 map과 mapError를 결합해 Result 타입 내의 value와 error를 변환하여, 단일 Result를 completionHandler로 리턴하여 한 번의 completionHandler만 호출하도록 함수를 개선했습니다.

**매핑을 통해 옵셔널의 내부값을 얻기 전까지 언래핑을 연기했듯이, Result의 내부 값을 얻고 싶을 때까지 오류 처리를 연기합니다.**

하지만 위의 코드에서 data를 json으로 변환에 실패하면 빈 딕셔너리를 리턴하고 있습니다.

빈 딕셔너리를 리턴하는 대신 Error를 표현하는 방식으로 개선하려 합니다.
이때 failure 케이스인 Result를 통해 Error를 표현할 수 있습니다.

이때 우리는 **faltMap**을 사용하게 됩니다.

**flatMapping over Result**

Result 타입 없이, 빈 딕셔너리 대신 Error를 던지는 방식으로 개선할 수 있지만 Result 타입을 사용하는 것과 error throwing이 섞이면 혼란스러울 수 있습니다. (뒤에서 Result 타입과 error throwing을 섞는 방식도 살펴볼 예정입니다.)

따라서 빈 딕셔너리를 리턴하는 대신 failure 케이스를 가진 Result를 리턴하는 방식으로 코드를 개선할 것입니다.

하지만 Result를 매핑한 map 클로저 내에서 빈 딕셔너리 대신 Result(failure)를 리턴하면, 결과적으로 중첩 Result 타입인 SearchResult<SearchResult<JSON>>가 리턴됩니다.

이때 중첩 Result를 flatMap을 통해 단일 Result로 평탄화 할 수 있습니다.
물론 failure 케이스인 Result는 map에서와 동일하게 flatMap에서도 무시됩니다.

결과적으로 flatMap을 사용해 flatMap 클로저 내부에서 진행되는 data -> JSON 변환 과정에서 발생되는 에러를 failure의 Result 타입으로 리턴하여 중첩 Result 타입을 단일 Result 타입으로 리턴합니다.

flatMap이 Result와 동작하는 과정을 살펴봅시다.
flatMap의 클로저에서 데이터를 변환하고 변환 실패 시 failure 케이스의 Result를 생성하고 성공 시 success 케이스의 Result를 생성하게 됩니다.

1. **You start with a successful result containing Data(x2) and with one result containing an error.**
2. **With flatMap, you apply a function to the value inside the result. This function will itself return a new result. (This new result could be successful and carry a value, or be a failure result containing an error. But if you start with a result containing an error, any flatMap action is ignored.)**
3. **You end up with a nested result. (If you start with an error, then nothing is transformed or nested.)**
4. **The nested result is flattened to a regular result. (If you start wih an error, nothing happened and the result remains the same.**

위에서 데이터 변환 실패 시 빈 딕셔너리를 리턴하는 코드를 failure 케이스의 Result를 리턴하도록 고쳐봅시다.

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

지금까지는 Result 타입의 mapping, flatmapping 연산 안에서(클로저 안에서) 에러를 던지는 함수를 호출하는 방식을 혼란스럽다는 이유로 피했습니다.

지금부터 Result 타입의 mapping, flatmapping 연산 안에서 에러를 던지는 함수를 추가해보고, 마지막에는 에러를 던지지 않고 파이프라인 방식으로 Result 타입의 전달만으로 에러를 핸들링하는 방법을 살펴봅시다.

search 함수의 flatMap 클로저 안에서 data를 JSON으로 변환할 때 실패 시 에러를 던지는 parseData 함수를 추가해봅시다.
parseData 함수는 JSON으로 데이터 변환에 실패 했을 때 ParsingError 타입 에러를 던집니다.

ParsingError 열거형과 parseData 함수를 코드로 살펴봅시다.

```swift
enum ParsingError: Error {
  case couldNotParseJSON
}

// data를 JSON으로 변환하며 실패 시 에러를 던집니다.
func parseData(_ data: Data) throws -> JSON {
  guard let json = try? JSONSerialization.jsonObject(with: data, options: []),
        let jsonDictionary = json as? JSON else {
          throw ParasingError.couldNotParseJSON
        }
  return jsonDictionay
}
```

에러를 던지는 함수가 에러를 던졌을 때 이를 failure 케이스의 Result 타입으로 변환하여 대응할 수 있습니다.

Result의 생성자(init)에 에러를 던지는 함수를 try 키워드와 함께 넣습니다.

이때 함수가 에러를 던지는 여부에 따라 Result가 success 케이스로 생성될지 failure 케이스로 생성될지 결정됩니다.
함수가 에러를 던졌을 때 failure 케이스의 Result 타입을 생성하고 에러를 던지지 않을 때 success 케이스의 Result 타입이 생성됩니다.

아래 코드로 살펴봅시다.

```swift
// parseData 함수가 에러를 던지면 Result failure, 던지지 않으면 Result success로 Result 타입이 생성됩니다.
let searchResult: Result<JSON, SearchResultError> = Result(try parseData(data))
```

하지만 위의 코드는 한 가지 문제가 있습니다.

parseData 함수가 던질 에러 타입을 런타임에서야 알 수 있고, 만약 parseData 함수가 SearchResultError 타입의 에러를 던지지 않는다면 다른 타입의 에러에는 추가적으로 대응해야 합니다.

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

throwing function을 Result 타입으로 변환했다면, 위의 do-catch 구문으로 search 함수를 완성해 봅시다.

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

flatMap이 사용하게 된 이유가 에러를 던질 상황에 Result 타입을 리턴하기 위함으로 에러가 발생한 상황에 프로그램의 흐름을 Error path로 바꾸는 것입니다.

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

map과 flatMap은 Result 타입이 failure 케이스일 경우 무시됩니다.
flatMap에서 특정 에러로 인해 failure 케이스인 Result가 리턴된다면 이후의 mapping이나 flatmapping은 무시됩니다.

## Multiple errors inside of Result

지금까지는 에러가 발생했을 때 Result 타입의 failure 케이스가 단일 에러 타입을 가지도록 했습니다.
위에서 다뤘던 Result 타입의 단일 에러 타입은 SearchResultError입니다.

에러 타입을 하나로 특정하는 방법은 좋은 방법입니다. 

하지만 다뤄야할 에러 타입이 너무 다양하다면, 초기 프로젝트 시기에는 모든 에러 타입을 정확한 단일 에러 타입으로 확정하기에 부담스러울 수 있습니다.

이때 우리는 SPM(Swift Package Manager)에서 제공하는 제네릭 타입인 **AnyError**를 사용하여 에러의 타입을 런타임에 알 수 있도록하여 정확한 에러 타입을 선언할 부담을 덜어줍니다.

**AnyError는 Result 내부의 Error에 들어가는 모든 에러 타입에 대응할 수 있습니다.**
Result 내부의 에러는 Error 타입을 따르기 때문에 AnyError 또한 Error 타입을 따르는 모든 에러에 대응할 수 있습니다.

예를 들어 Result<String, AnyError>와 같이 사용될 수 있습니다.

AnyError를 가진 Result 타입을 생성하는 방법은 두 가지가 있습니다.

첫 번째로 Error 타입을 AnyError로 변환 후 Result 타입에 변환된 AnyError를 넣는 방식입니다.
이때 AnyError(Error)와 같은 형식으로 AnyError의 생성자에 Error 타입을 넣어 AnyError를 생성할 수 있습니다.

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
자료형이 Result<String, AnyError>이기 때문에 Result의 생성자로 들어온 Error 타입도 자동으로 AnyError 타입으로 형변환됩니다.

두 번째 방법은 에러를 던지는 함수를 Result의 생성자에 넣는 방법입니다.

아래 코드로 살펴봅시다.

```swift
let otherResult: Result<String, AnyError> = Result(anyError: { () throws -> String in
  throw PaymentError.insufficientFunds
}
```

AnyError는 에러 타입을 런타임에 알게 됩니다. 따라서 정확한 에러 타입을 알 필요 없을 때 AnyError를 Result 타입과 함께 사용할 수 있습니다.
파이프라인에서 각 단계별로 리턴되는 에러 타입이 다양할 경우, AnyError를 사용해 리턴되는 에러들을 특정 에러 타입으로 변환할 부담을 줄여줍니다.

AnyError를 활용하여 processPayment라는 돈을 이체하는 함수를 만들어봅시다.
각 단계에서 다양한 유형의 오류를 반환할 수 있으므로 AnyError를 통해 다양한 오류를 하나의 특정 유형으로 변환해야 하는 부담을 줄일 수 있습니다.

아래 코드로 살펴봅시다.

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

mapAny 메소드는 에러를 던지는 함수를 허용한다는 점을 제외하면 map과 유사하게 작동합니다.

mapAny 안의 함수에서 error를 던지면 해당 에러를 AnyError로 감싸서 리턴하게 됩니다.
즉 AnyError를 가진 Result failure 케이스를 리턴합니다. 

mapAny를 통해 에러 핸들링(catch) 없이 map으로 에러를 던지는 함수를 넘길 수 있습니다.

mapAny는 flatMap 처럼 클로저 안에서 새로운 Result 타입을 리턴하여 Result의 에러 타입을 변환할 수는 없습니다.
그저 mapAny에 속하는 함수가 에러를 리턴하면 해당 에러를 AnyError로 감싸서 리턴 시켜줍니다.

따라서 map은 연산에 속하는 함수가 에러를 던지지 못하는 함수임을 나타내고, mapAny는 에러를 던질 수 있는 함수임을 나타냅니다!

map과 mapAny의 차이점은 map이 모든 Result 유형에서 작동하지만 함수 발생 시 오류를 포착하지 못한다는 것입니다.
반대로, mapAny는 에러를 던지는 함수와 던지지 않는 함수 모두에서 작동하지만 AnyError를 포함하는 Result 유형에서만 사용할 수 있습니다.

**결과적으로 mapAny를 통해 Result의 value를 매핑하고 에러가 발생할 경우 AnyError로 감싼 에러를 가진 Result 타입을 얻을 수 있습니다.**

**Matching with AnyError**

AnyError 속 실제 에러 타입을 꺼내기 위해서는 **underlyingError** 프로퍼티를 사용해야 합니다.

AnyError 속 에러에 따라 failure 케이스를 매칭하는 아래 코드를 살펴봅시다.

```swift
processPayment(fromAccount: from, toAccount: to, amountInCents: 100) { (result: Result<String, AnyError>) in
  switch result {
  case .success(let value): print(value)
  case .failure(let error) where error.underlyingError is AccountError:
    // Result 타입이 가진 AnyError의 타입이 AccountError 타입일 경우를 나타냅니다.
    print("Account error")
  case .failure(let error):
    print(error)
  }
}
```

AnyError를 사용하면 더욱 유연성있는 코드를 만듭니다.

하지만 AnyError는 Result의 error 타입을 컴파일 타임에 알 수 있도록 했던 장점을 잃게 됩니다.
따라서 프로젝트 초기가 아니고 시간적 여유가 있다면 AnyError가 아니라 일반적 error를 사용해 컴파일 타임에 이점을 얻을 수 있도록 합시다.

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

위의 Service 프로토콜을 따르는 SubscriptionsLoader 타입에서 load 함수를 항상 성공하는 함수라고 가정하겠습니다.
항상 성공하는 함수의 리턴이 Result 타입이기 때문에 Error 타입을 따르는 빈 열거형을 만들어 Result 속 Error로 넣어야 합니다.

아래 코드는 Service 프로토콜을 따르는 SubscriptionsLoader 클래스를 구현한 코드입니다. 
SubscriptionsLoader 클래스의 load 함수는 항상 성공하는 함수이기 때문에 Result<Value, Err> 타입으로 리턴되는 Error에는 빈 열거형을 넣게 됩니다.

```swift
struct Subscription {
  // ...details omitted
}

// 빈 열거형
enum BogusError: Error {} 

final class SubscriptionsLoader: Service {
  func load(complete: @escaping (Result<[Subscription], BogusError>) -> Void) {
    // ...load data. Always succeeds
    let subscriptions = [Subscription(), Subscription()]
    complete(Result(subscriptions))
  }
}
```

BogusError는 빈 열거형이기 때문에 인스턴스화 할 필요도 할 수 없습니다.
또한 Error 케이스에 빈 열거형(BogusError)을 넣으면, Result 타입의 failure 케이스를 switch case 매칭할 수 없습니다.

물론 아래 코드와 같이 failure 케이스 매칭 없이 success 케이스만 case 매칭할 수 있습니다. 

```swift
let subscriptionsLoader = SubscriptionsLoader()
subscriptionsLoader.load { (result: Result<[Subscription], BogusError>) in
  switch result {
  case .success(let subscriptions): print(subscriptions)
  // You don't need .failure
  }
}
```

이처럼 에러가 발생하지 않는 상황에서 빈 열거형을 Error 타입으로 넣는 방식의 장점은 열거형의 케이스를 줄여 패턴 매칭 코드를 줄이고 코드를 깔끔하게 만듭니다.

하지만 빈 열거형을 Error 타입으로 넣는 방식은 공식적인 방식은 아닙니다.

**Never** 타입이 빈 열거형을 대신하는 공식적인 방식입니다.

Never 타입은 컴파일러에게 특정 경로(케이스)로 프로그램의 흐름이 가지 않는다는 사실을 알립니다.
다시 말해, 불가능한 경로를 나타냅니다.

아래 코드로 Never 타입을 살펴봅시다.

```swift
func crashAndBurn() -> Never {
  fatalError("Something very, very bad happened")
}
```

위의 crashAndBurn 함수는 Never 타입을 리턴하기 때문에 절대 값을 리턴하지 않는 함수임을 알 수 있습니다.

Never 타입의 구현부는 아래 코드와 같습니다. Never 타입도 빈 열거형입니다.
이미 스위프트에서 지원하는 빈 열거형인 Never 타입이 있는데 굳이 추가적인 빈 열거형을 사용할 필요는 없습니다.

```swift
public enum Never {}
```

앞에서 봤던 빈 열거형 BogusError를 Never 타입으로 대체 가능합니다.
물론 Result 타입의 에러로 Never 타입을 사용하려면 Never 타입을 Error 타입을 따르도록 해야 합니다.
Result 타입의 에러는 Error 타입을 따르도록 타입 제약을 가지고 있기 때문입니다.

아래 코드는 BogusError를 Never 타입으로 고친 코드입니다.

```swift
// Never 타입이 Error 타입을 따르도록 합니다.
extension Never: Error {}

final class SubscriptionsLoader: Service {
  func load(complete: @escaping (Result<[Subscription], Never>) -> Void) {
    // ...load data. Always succeeds
    let subscriptions = [Subscription(), Subscription()]
    complete(Result(subscriptions))
  }
}
```

Result 타입의 Error에 Never을 넣을 수 있듯이 Success 케이스의 Value에도 Never를 넣을 수 있습니다.
Success 케이스의 Value에 Never를 넣게 되면 절대 Success 되지 않는 동작을 의미합니다.

## Summary
- Using the default way of URLSession's data tasks is an error-prone way of error handling.
- Result is offered by the Swift Package Manager and is a good way to handle asynchronous error handling.
- Result has two generics and is a lot like Optional, but has a context of why something failed.
- Result is a compile-time safe way of error handling, and you can see which error to expect before running a program.
- By using map and flatMap and mapError, you can cleanly chain transformations of your data while carrying an error context.
- Throwing functions can be converted to a Result via a special throwing initializer. This initializer allows you to mix and match two error throwing idioms.
- You can postpone strict error handling with the use of AnyError.
- With AnyError, multiple errors can live inside Result.
- If you're working with many types of errors, working with AnyError can be faster, at the expense of not knowing which errors to expect at compile time.
- AnyError can be a good alternative to NSError so that you reap the benefits of Swift error types.
- You can use the Never type to indicate that a Result can't have a failure case, or a success case.
