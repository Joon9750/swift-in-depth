# Swift patterns

## This chapter covers
- Mocking types with protocols and associated types
- Understanding conditional conformance and its benefits
- Applying conditional conformance with custom protocols and types
- Discovering shortcomings that come with protocols
- Understanding type erasure
- Using alternatives to protocols

## Dependency injection

앞으로 스위프트 개발에 유용한 패턴들을 살펴보겠습니다.

먼저 의존성 주입(Dependency injection)을 살펴봅시다.

의존성 주입에서 말하는 의존성이란 서로 다른 객체 사이에 의존 관계를 말합니다.
A 클래스에 B 클래스 객체를 인스턴스로 사용한다면 A 클래스와 B 클래스 사이에 의존 관계가 있는것입니다.

다시 말해, 의존하는 객체가 수정되면 다른 객체도 영향을 받게 됩니다.

또한 의존성 주입에서 주입이란 생성자를 활용한 방법 등으로 외부에서 생성한 객체를 넣는것을 의미합니다.

의존성을 가진 코드가 많을수록 코드의 재사용성이 떨어집니다.
재사용을 위해 코드를 수정하면 매번 의존성을 가진 객체들을 함께 수정해야하기 때문입니다.

의존성을 강하게 가지는 코드를 의존성 주입으로 의존 관계를 끊을 수 있습니다.

또한 의존성 주입을 통해 아래와 같은 이점을 얻을 수 있습니다.

1. Unit Test가 용이해집니다.
2. 코드의 재활용성을 높여줍니다.
3. 객체 간의 의존성(종속성)을 줄이거나 없엘 수 있습니다.
4. 객체 간의 결합도이 낮추면서 유연한 코드를 작성할 수 있습니다.

의존성 주입은 DIP(의존 관계 역전 법칙)와도 관련이 깊습니다.

의존 관계 역전 법칙은 객체 지향 프로그래밍 설계의 다섯가지 기본 원칙(SOLID) 중 하나입니다. 

추상화 된 것은 구체적인 것에 의존하면 안되고, 구체적인 것이 추상화된 것에 의존 해야합니다.
즉, 구체적인 객체는 추상화된 객체에 의존 해야 한다는 것이 핵심입니다.

스위프트에서 말하는 추상화는 프로토콜로 구현할 수 있습니다.

**Swapping an implementation**

구현부를 교환할 수 있다는 특징은 의존성 주입에 의해 만들어집니다.
지금부터 의존성 주입을 활용하여 구현부를 교환할 수 있는 WeatherAPI를 구현할 것입니다.

구체적 객체에 의존하지 않고 추상화된 객체에 의존하기 때문에 추상화된 객체를 따르는 여러 구현부를 교환할 수 있게 됩니다.

WeatherAPI는 실제 네트워크 세션(URLSession), 오프라인 네트워크 세션(OfflineURLSession) 그리고 테스트 세션(MockSession) 구현부에 모두 대응하도록 구현합니다.
세 가지 세션은 WeaterAPI의 생성자로 주입 받는 객체(구현부)가 됩니다.

WeatherAPI가 URLSession과 같은 구체적인 타입에 의존하면 강한 의존 관계가 형성되며 OfflineURLSession이나 MockSession이 주입되었을 때 대응하지 못합니다.
따라서 세 가지 세션을 추상화한 Session 프로토콜을 WeatherAPI에서 의존해야 합니다.

추상화된 Session 프로토콜에 의존하기 때문에 주입 받는 세션의 타입과 상관없이 Session 프로토콜의 함수를 사용할 수 있습니다. 

의존성 주입을 통해 WeatherAPI는 URLSession, OfflineURLSession 그리고 MockSession 타입과의 강한 의존 관계를 만들지 않게 됩니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/c0e48dae-9db6-48a0-a787-0575b6ed8b96)

WeatherAPI가 가진 Session 프로토콜에 의해 Session 프로토콜을 따르는 URLSession, OfflineSession, MockSession 타입을 주입 받을 수 있습니다.
의존성 주입으로 구현부를 교환이 가능해졌습니다.

Session 프로토콜을 코드로 구현해봅시다.

```swift
protocol Session {
  associatedType Task

  func dataTask(
    with url: URL,
    completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void
  ) -> Task
}
```

dataTask 함수에서 URLSessionDataTask나 다른 프로토콜을 리턴하기 보다, 연관 값 Task를 리턴하도록 구현했습니다.
프로토콜의 연관 값은 컴파일 타임에 타입이 결정되기 때문에 URLSessionDataTask와 같은 타입과 함께 여러 타입에 대응하게 됩니다.

WeatherAPI에 주입할 URLSession 타입은 Session 프로토콜을 따라야 합니다.

```swift
extension URLSession: Session {}
```

이제는 본격적으로 WeatherAPI의 구현부를 살펴봅시다.

```swift
final class WeatherAPI<S: Session> {
  let session: S

  init(session: S) {
    self.session = session
  }

  func run() {
    guard let url = URL(string: "https://www.someweatherstartup.com") else {
      fatalError("Could not create url")
    }
    let task = session.dataTask(with: url) { (data, response, error) in
      // Work with retrieved data.
    }
    task.resume()
  }
}

let weatherAPI = WeatherAPI(session: URLSession.shared)
weatherAPI.run()
```

WeatherAPI는 제네릭 타입을 활용하여 Session 프로토콜을 따르는 구현부를 교환하여 입력 받도록 만들었습니다.

위에서 task.resume() 함수를 호출하고 있지만, 아직 Session 프로토콜의 연관 값인 Task에 resume 함수를 구현하지 않았습니다.

resume 함수를 가진 DataTask 프로토콜을 만들고 Session 프로토콜의 연관 값인 Task를 DataTask로 타입 제약한다면, Task의 resume 함수를 호출할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
protocol DataTask {
  func resume()
}

protocol Session {
  associatedtype Task: DataTask

  func dataTask(
    with url: URL,
    completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void
  ) -> Task
}
```

Session 프로토콜의 연관 값 Task에 DataTask 프로토콜로 타입 제약을 걸어 dataTask 함수가 리턴하는 Task 타입의 객체에 resume 함수를 호출할 수 있게 되었습니다.

지금까지의 추상화 과정을 그림으로 살펴봅시다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/ab8ce8de-e1c1-4388-b25b-a3ec806ec9ee)

구체적인 객체인 URLSession과 URLSessionDataTask 모두 추상화(protocol)에 의존하고 있습니다.

위에서 만든 DataTask 프로토콜을 따르는 URLSession에서 쓰일 URLSessionDataTask도 만들어주면 URLSession이 완성됩니다.

```swift
extension URLSessionDataTask: DataTask {}
```

이제 URLSession을 생성하여 WeatherAPI에 주입할 수 있게 되었습니다.
물론 URLSession 이외에도 Session 프로토콜을 따르는 타입이라면 WeatherAPI의 생성자로 주입될 수 있습니다.

지금부터 URLSession 이외에 OfflineURLSession과 MockSession을 생성하여 WeatherAPI가 구현부를 교체(Swapping an implementation)해 보겠습니다.
의존성 주입으로 WeatherAPI가 구체적 타입이 아닌 추상화에 의존하기 때문에 구현부를 URLSession, OfflineURLSession, MockSession 등으로 교체해도 WeatherAPI에 수정될 부분은 없습니다.

먼저 OfflineURLSession과 OfflineTask를 구현하겠습니다.

```swift
final class OfflineURLSession: Session {
  var sessions = [URL: OfflineTask]()

  func dataTask(
    with url: URL,
    completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void
  ) -> OfflineTask {
    let task = OfflineTask(completionHandler: completionHandler)
    sessions[url] = task
    return task
  }
}

enum ApiError: Error {
  case couldNotLoasData
}

struct OfflineTask: DataTask {
  typealias Completion = (Data?, URLResponse?, Error?) -> Void
  let completionHandler: Completion

  init(completionHandler: @escaping Completion) {
    self.completionHandler = completionHandler
  }

  func resume() {
    let url = URL(fileURLWithPath: "prepared_response.json")
    let data = try! Data(contentsOf: url)
    completionHandler(data, nil, nil)
  }
}
```

위의 OfflineURLSession은 서버 연결 없이 로컬 데이터가 로드되도록 하여 서버 환경에 구애받지 않고 테스트할 수 있도록 합니다.

우리는 의존성 주입으로 객체 간의 의존 관계를 서로가 아니라 추상적 대상에 두어 구현부를 교체할 때 추가적인 수정이 발생하지 않도록 만들었습니다.

아래 코드와 같이 단지 주입하는 객체만 달라질 뿐, 구현부가 교체되었다고 해서 WeatherAPI의 코드를 수정하지 않습니다.

```swift
let productionAPI = WeatherAPI(session: URLSession.shared)
let offlineAPI = WeatherAPI(session: OfflineURLSession())
```

Session 프로토콜이 Task 연관 값을 활용하는 것처럼 URLSession, OfflineURLSession, MockSession 등 하나의 타입으로 인스턴스화 하기 어려울 때 프로토콜 내부의 연관 값은 유용하게 사용됩니다.

이번에는 MockSession과 MockTask를 구현해 WeatherAPI 클래스를 테스트하는 코드를 작성해 봅시다.

```swift
class MockSession: Session {
  let expectedURLs: [URL]
  let expectation: XCTestExpectation

  init(expectation: XCTestExpectation, expectedURLs: [URL]) {
    self.expectation = expectation
    self.expectedURLs = expectedURLs
  }

  func dataTask(
    with url: URL,
    completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void
  ) -> MockTask {
    return MockTask(expectedURLs: espectedURLs, url: url, expectation: expectation)
  }
}

struct MockTask: DataTask {
  let expectedURLs: [URL]
  let url: URL
  let expectation: XCTestExpectation

  func resume() {
    guard expectedURLs.contains(url) else {
      return
    }
    self.expectation.fulfill()
  }
}
```

이번에는 API를 테스트하는 APITestCase 클래스를 살펴봅시다.

```swift
class APITestCase: XCTestCase {
  var api: API<MockSession>!

  func testAPI() {
    let expectation = XCTestExpectation(description: "Expected someweatherstartup.com")
    let session = MockSession(expectation: expectation, expectedURLs: [URL(string: "www.someweatherstartup.com")!])
    api = API(session: session)
    api.run()
    wait(for: [expectation], timeout: 1)
  }
}

let testcase = APITestCase()
testcase.testAPI()
```

Session 프로토콜을 확장하여 dataTask 함수의 구현부를 제공할 수도 있습니다.
기존의 dataTask 함수가 아닌, Result 타입 클로저를 가진 dataTask 함수를 새로 만들고 해당 함수의 구현부를 프로토콜 확장에서 제공하겠습니다.

chapter11에서 살펴본 Result 타입을 활용해 (Data?, URLResponse?, Error?)를 Result로 변형시켜 다루면 에러 핸들링에 유리합니다.

아래 코드로 살펴봅시다.

```swift
protocol Session {
  associatedtype Task: DataTask

  func dataTask(
    with url: URL,
    completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void
  ) -> Task

  func dataTask(
    with url: URL,
    completionHandler: @escaping (Result<Data, AnyError>) -> Void
  ) -> Task
}

extension Session {
  func dataTask(
    with url: URL,
    completionHandler: @escaping (Result<Data, AnyError>) -> Void
  ) -> Task {
    return dataTask(with: url, completionHandler: { data, response, error in
      if let error = error {
        let anyError = AnyError(error)
        completionHandler(Result.failure(anyError))
      } else if let data = data {
        completionHandler(Result.success(data))
      } else {
        fatalError()
      }
    })
  }
}

// dataTask 함수 호출부
URLSession.shared.dataTask(with: url) { (result: Result<Data, AnyError> in
  // ...
}

OfflineURLSession().dataTask(with: url) { (result: Result<Data, AnyError> in
  // ...
}
```

의존성 주입을 활용해 여러 구현(production, debugging, testing)을 전환하며 구체적인 WeatherAPI 타입을 만들 수 있습니다.
또한 추상화(프로토콜)에 구체적 타입을 의존하게 만들어 구체적 타입끼리의 의존관계도 피할 수 있었습니다.

## Conditional conformance

Conditional conformance를 직역하면 조건부 적합성 정도입니다.

조건부 적합성은 Swift 4.1부터 도입되었고 특정 상태에서만 해당 타입이 프로토콜을 따르도록 만들 수 있습니다.

```swift
protocol Purchaseable {
  func buy()
}

struct Book: Purchaseable {
  func buy() {
    print("Buy book")
  }
}

// 조건부 적합성!
extension Array: Purchaseable where Element: Purchaseable {
  func buy() {
    for element in self {
      element.buy()
    }
  }
}
```

extension Array: Purchaseable where Element: Purchaseable와 같은 코드가 생소할 수 있습니다.
이와 같은 코드는 Array 타입의 연관 값인 Element가 Purchaseable 프로토콜을 따를 때 Array 타입이 Purchaseable 프로토콜을 따른다는 조건부 적합성을 구현한 코드입니다.

With conditional conformance, you can make a type adhere to a protocol but only under certain conditions.

**Free functionality**

Equatable 프로토콜이나 Hashable 프로토콜을 채택하여 두 프로토콜에서 제공하는 함수를 별도의 구현 없이 사용하는 것이 조건부 적합성의 예입니다.

아래 코드와 같이 Equatable 프로토콜을 채택한 타입의 경우 == 함수 구현 없이 == 함수를 사용할 수 있습니다.
물론 동일하다는 조건을 수정하기 위해서는 == 함수를 오버라이드 해야합니다.

```swift
struct Movie: Equatable {
  let title: String
  let rating: Float
}

let movie = Movie(title: "The princess bride", rate: 9.7)
movie == movie  // true. You can already compare without implementing Equatable.
```

Swift synthesizes this for free on some protocols, such as Equatable and Hashable, but not every protocol.
For instance, you don't get Comparable for free, or the ones that you introduce yourself.
Unfortunately, Swift doesn't synthesize methods on classes.

**Conditional conformance on associated types**

지금부터는 조건부 적합성을 언제 사용하면 좋을지 살펴봅시다.

결론부터 이야기하면 조건부 적합성은 제네릭을 내부 값으로 가지는 객체에 자주 쓰입니다.
내부 인스턴스가 특정 프로토콜을 따를 때 내부 인스턴스를 감싸는 객체 또한 해당 프로토콜을 따르도록 만들어 외부 객체에서도 해당 프로토콜의 함수를 사용할 수 있도록 만듭니다.

```swift
protocol Sequence<Element>

@frozen
struct Array<Element>
```

Array가 Element 연관 값을 가졌기 때문에 조건부 적합성을 설명하기에 적합합니다.
Array가 구조체인데 어떻게 연관 값을 가졌는지 의문일 수 있지만, Array의 연관 값 Element는 Sequence 프로토콜로부터 온 연관 값입니다.

Track 프로토콜과 AudioTrack 구조체를 만들고 Array의 연관 값을 Element를 Track 프로토콜로 제약해 발생하는 문제를 살펴봅시다.

```swift
protocol Track {
  func play()
}

struct AudioTrack: Track {
  let file: URL

  func play() {
    print("playing audio at \(file)")
  }
}
```

Track 프로토콜을 따르는 객체를 배열에 저장할 때, 배열 자체에 play 함수를 만들어 배열의 play 함수가 호출하여 모든 배열 내부 값들의 play 함수가 호출되도록 만들려합니다.

Array의 연관 값 Element가 Track 프로토콜을 따를 때!
Array 타입에 play 함수를 제공합니다.

아래 코드로 살펴봅시다.

```swift
extension Array where Element: Track {
  // Track 프로토콜의 play 함수와 별개로 Array 타입에 추가한 함수입니다.
  func play() {
    for element in self {
      // Element가 Track 프로토콜을 따르기 때문에 Track 프로토콜에서 제공하는 play 함수를 호출 가능합니다.
      self.play()
    }
  }
}
```

위 코드는 Array의 Element가 Track 프로토콜을 따를 때 Array 객체에 play 함수를 제공합니다.

하지만 문제는 Array 자체로 Track 프로토콜을 따르지 않는다는 점입니다.
단지, Element가 Track 프로토콜을 따를 때 Array를 확장한 기능을 사용할 뿐입니다.

따라서 위 코드로는 Track 타입을 입력으로 받는 함수에 Element가 Track 프로토콜을 따르는 Array를 넘길 수 없습니다. 

또한 중첩된 배열에서는 내부 값이 Track 프로토콜을 따르더라도 play 함수를 호출할 수 없습니다.
Array 자체로 Track 프로토콜을 따르지 않기 때문입니다.

아래 코드로 위 문제들을 살펴봅시다.

```swift
let tracks = [
  AudioTrack(file: URL(fileURLWithPath: "1.mps"))
  AudioTrack(file: URL(fileURLWithPath: "2.mps"))
]

// If An Array is nested, you can't call play() any more.
[tracks, tracks].play()  // error: type of expression is ambigous without more context

// Or you can't pass an array if anything expects the Track protocol.
func playDelayed<T: Track>(_ track: T, delay: Double) {
  // ...snip
}

playDelayed(tracks, delay: 2.0)  // argument type '[AudioTrack]' does not conform to expected type 'Track'
```

**Making Array conditionally conform to a custom protocol**

지금부터 위의 문제를 조건부 적합성으로 해결해봅시다.

지금까지는 Array의 Element가 특정 프로토콜을 따를 때 Array의 확장에서 함수를 제공하는 조건부 적합성을 만들어 Array 자체가 특정 프로토콜을 따르진 못했습니다.

이제는 Array의 Element가 특정 프로토콜을 따를 때 Array의 확장에서 함수를 제공하는 동시에 Array 또한 해당 프로토콜을 따르는 타입이 되도록 구현할 예정입니다.

아래 코드로 살펴봅시다.

```swift
// Before. Not conditionally conforming!!
extension Array where Element: Track {
  // ... snip
}

// After. You have conditional conformance!!!
extension Array: Track where Element: Track {
  func play() {
    for element in self {
      element.play()
    }
  }
}
```

extension Array: Track where Element: Track을 통해 Element 연관 값이 Track 프로토콜을 따를 때, Array 또한 Track 프로토콜을 따르는 타입이 됩니다.

Array에 조건부 적합성을 적용해 Track 타입을 원하는 함수로 Array를 넘길 수 있고 중첩 배열에 play 함수도 호출할 수 있습니다.

```swift
let nestedTracks = [
  [
    AudioTrack(file: URL(fileURLWithPath: "1.mps")),
    AudioTrack(file: URL(fileURLWithPath: "2.mps"))
  ],
  [
    AudioTrack(file: URL(fileURLWithPath: "3.mps")),
    AudioTrack(file: URL(fileURLWithPath: "4.mps"))
  ]
]

// Nesting works.
nestedTracks.play()

// And, you can pass this array to a function expecting a Track!
playDelayed(tracks, delay: 2.0)
```

**Conditional conformance and generics**

지금까지 연관 값을 가진 프로토콜에 조건부 적합성을 적용했다면, 이제는 조건부 적합성을 제네릭 타입에 사용해 봅시다.

조건부 적합성은 연관 값을 가진 프로토콜에도 사용할 수 있지만 제네릭 타입에도 사용 가능합니다.

Optional이 대표적인 제네릭 타입을 가진 열거형입니다.
열거형 Optional은 Wrapped 제네릭 타입을 가지고 있습니다.

아래 코드로 Optional에 조건부 적합성을 적용한 코드를 살펴봅시다.

```swift
extension Optional: Track where Wrapped: Track {
  func play() {
    switch self {
    case .some(let track):
      track.play()
    case nil:
      break  // do nothing
    }
  }
}
```

위 코드는 Optional의 제네릭 타입인 Wrapped가 Track 프로토콜을 따를 때 Optional 또한 Track 프로토콜을 따르고 play 함수를 제공합니다.

물론 extension Optional: Track where Wrapped: Track 방식이 아닌 extension Optional where Wrapped: Track 코드를 사용해도 play 함수를 제공 받지만, Optional 자체가 Track 타입이 되지 못합니다.

```swift
let audio: AudioTrack? = AudioTrack(file: URL(fileURLWithPath: "1.mps"))
audio?.play()
```

audio가 Optional(AudioTrack) 타입이기 때문에 audio?는 Optional(AudioTrack)의 Value 값이므로 Track 타입입니다. 따라서 Track 타입을 입력 받는 함수에 audio?를 넣을 수 있습니다.

```swift
let audio: AudioTrack? = AudioTrack(file: URL(fileURLWithPath: "1.mps"))
playDelayed(audio, delay: 2.0)
```

**Conditional conformance on your types**

위에서 봤듯이 조건부 적합성은 제네릭 타입에 적용하기 좋습니다.

Conditional conformace becomes powerful when you hace a generic type storing an inner type, and you want the generic type to mimic the behavior of the inner type inside.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/c2e1874a-494e-4cf7-93b6-fb3f45281ba8)

좀전에 봤듯이, 조건부 적합성을 통해서 inner 값의 타입이 Track 타입일 때 Array가 Track 타입이 됩니다.

Optional의 제네릭 타입외에도 커스텀 제네릭 타입에도 조건부 적합성을 적용해 봅시다.

CachedValue 타입을 만들어 커스텀 제네릭 타입에 조건부 적합성을 적용해 봅시다.
CachedValue는 시간이 오래 소요되는 계산의 결과를 캐싱하고 리프레쉬 전까지 캐싱한 결과를 유지합니다.

먼저 CachedValue가 동작하는 방식을 살펴봅시다.

```swift
// CachedValue를 생성하고 커스텀 클로저를 전달합니다.
let simplecache = CachedValue(timeToLive: 2, load: { () -> String in
  print("I am being refreshed!")
  return "I am the value inside CachedValue"
}

// Prints: "I am being refreshed!"
simplecache.value  // "I am the value inside CachedValue"
simplecache.value  // "I am the value inside CachedValue"

sleep(3)  // wait 3 seconds

// Prints: "I am being refreshed!"
simplecache.value  // "I am the value inside CachedValue"
```

CachedValue의 구현부를 살펴봅시다.

```swift
final class CachedValue<T> {
  private let load: () -> T
  private var lastLoaded: Date

  private var timeToLive: Double
  private var currentValue: T

  public var value: T {
    let needsRefresh = abs(lastLoaded.timeIntervalSinceNow) > timeToLive
    if needsRefresh {
      currentValue = load()
      lastLoaded = Date()
    }
    return currentValue
  }

  init(timeToLive: Double, load: @escaping (() -> T)) {
    self.timeToLive = timeToLive
    self.load = load
    self.currentValue = load()
    self.lastLoaded = Date()
  }
}
```

아직 CachedValue<T> 타입은 조건부 적합성이 적용되지 않았습니다.

CachedValue의 제네릭 타입이 Equatable을 따를 때, Comparable을 따를 때 그리고 Hashable을 따를 때, 제네릭 타입을 가진 CachedValue 또한 해당 프로토콜을 따르도록 조건부 적합성을 구현해 봅시다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/33637c6f-3da8-49a2-84a1-3c4c76772527)

코드로 살펴봅시다.

```swift
// Conforming to Equatable
extension CachedValue: Equatable where T: Equatable {
  static func == (lhs: CachedValue, rhs: CachedValue) -> Bool {
    return lhs.value == rhs.value
  }
}

// Conforming to Hashable
extension CachedValue: Hashable where T: Hashable {
  func hash(into hasher: inout Hasher) {
    hasher.combine(value)
  }
}

// Conforming to Comparable
extension CachedValue: Comparable where T: Comparable {
  static func <(lhs: CachedValue, rhs: CachedValue) -> Bool {
    return lhs.value < rhs.value
  }

  static func == (lhs: CachedValue, rhs: CachedValue) -> Bool {
    return lhs.value == rhs.value
  }
} 
```

위와 같이 CachedValue에 조건부 적합성을 적용하면 아래 코드와 같이 사용 가능합니다.

```swift
let cachedValueOne = CacheValue(timeToLive: 60) {
  // Perform expensive operation
  // E.g. Calculate the purpose of life
  return 42
}

let cachedValueTwo = CacheValue(timeToLive: 120) {
  // Perform another expensive operation
  return 100
}

cachedValueOne == cachedValueTwo // Equatable: You can check for equality.
cachedValueOne > cachedValueTwo  // Equatable: You can compare two cached values.

let set = Set(arrayLiteral: cachedValueOne, cachedValueTwo)  // Hashable: You can store CachedValue in a set
```

Conditional conformance works best when storing the lowest common denominator inside the generic, meaning that you should aim to not add too many constraints on T in the case.

제네릭에 너무 많은 타입 제약을 걸지 않아야 합니다. 
너무 많은 타입을 제약하지 않아야 제네릭 타입에 조건부 적합성을 적용할 경우 타입을 확장하기 쉽습니다.

과장해서 CacheValue의 제네릭 T가 10개의 프로토콜의 타입 제약을 받는다면 매우 적은 타입이 제네릭 T를 만족합니다.
따라서 조건부 적합성의 이점을 크게 얻을 수 없습니다.

## Dealing with protocol shortcomings

프로토콜은 좋은 방법이지만, 단점 있습니다.
지금부터 프로토콜의 단점을 살펴보고 두 가지 방법(enum, type erasure)으로 해결해 봅시다.

Hashable 타입을 저장하려 할 때 프로토콜을 단점을 찾을 수 있습니다.

Hashable 프로토콜을 따르는 PokerGame 프로토콜과 PokerGame 프로토콜을 따르는 StudPoker 구조체와 TexasHoldem 구조체를 만들었습니다.

```swift
protocol PokerGame: Hashable {
  func start()
}

struct StudPoker: PokerGame {
  func start() {
    print("Starting StudPoker")
  }
}

struct TexasHolem: PokerGame {
  func start() {
    print("Starting Texas Hodlem")
  }
}
```

위의 PokerGame 프로토콜을 따르는 타입은 동시에 Hashable을 따르기 때문에 Set에 저장되거나 Dictionary의 Key에 저장될 수 있습니다.

하지만 아래와 같이 StudPoker 구조체나 TexasHolem 구조체 같은 특정 타입이 아니라 PokerGame 프로토콜로 Dictionary의 Key에 저장될 수 없습니다.

If you want to mix and match different types of PokerGame as keys in a dictionary or inside an array, you stumble upon a shortcoming.

코드로 살펴봅시다.

```swift
// This won't work
var numberOfPlayers = [PlayerGame: Int]()

// The error that the Swift compiler throws is:
// error: using 'PokerGame' as a concrete type conforming to protocol 'Hashable' is not supported
var numberOfPlayers = [PokerGame: Int]()
```

프로토콜이 특정 타입(concrete type)으로 사용될 수 없기 때문에 프로토콜로 타입을 지정하고 해당 프로토콜을 따르는 타입들을 섞을 수 없습니다.

PlayerGame 프로토콜을 따르는 StudPoker 구조체와 같이 특정 타입(concrete type)을 사용할 경우 [StudPoker: Int] 와 같이 Dictionary의 Key나 Array에 저장할 수 있습니다.
하지만 StudPoker 타입을 사용할 경우 PlayerGame 프로토콜을 따르는 타입들을 섞어서 저장할 수 없습니다.

그렇다면 제네릭을 사용하면 추상화된 타입을 사용해 하위 타입들을 섞고 컴파일 에러도 피할 수 있을까요?

아래 코드와 같이 제네릭을 사용해 해결하려... 해봅시다.

```swift
func storeGame<T: PokerGame>(games: [T]) -> [T: Int] {
  /// ... snip
}
```

하지만 위 코드와 같은 제네릭 함수는 PokerGame 프로토콜을 따르는 단일 타입으로만 결정됩니다.
결국 제네릭 타입을 사용해도 특정 프로토콜을 따르는 여러 타입을 섞을 수 없습니다.

아래와 코드와 같이 결국 단일 특정 타입으로 storeGame 함수의 입력 타입이 결정됩니다.

```swift
func storeGame(games: [TexasHoldem]) -> [TexasHoldem: Int] {
  /// ... snip
}

func storeGame(games: [StudPoker]) -> [StudPoker: Int] {
  /// ... snip
}
```

Again, you can't easily mix and match PokerGame types into a single container, such as a dictionary.

제네릭으로 해결할 수 없었지만, 열거형을 사용하거나 type erasure를 사용해 지금까지의 문제를 해결할 수 있습니다.

**Avoiding a protocol using an enum**

열거형은 프로토콜과 달리 특정 타입(concrete type)입니다.

먼저 열거형의 케이스에 사용할 타입들을 넣습니다.
예를 들어 위에서 프로토콜 PokerGame을 따르는 StudPoker와 TexasHoldem 타입을 열거형의 케이스에 넣습니다.

아래 코드로 살펴봅시다.

```swift
enum PokerGame: Hashable {
  case studPoker(StudPoker)
  case texasHoldem(TexasHoldem)
}

struct StudPoker: Hashable {
  // ... 구현부 생략
}

struct TexasHoldem: Hashable {
  // ... 구현부 생략
}  
```

프로토콜과 달리 StudPoker와 TexasHoldem 구조체 모두 Hashable 프로토콜을 채택해야 합니다.

위와 같이 섞어서 사용할 타입들을 열거형의 케이스에 넣어 Set, Array 그리고 Dictionary의 Key 등에 사용할 수 있습니다.
이때 프로토콜과 달리 열거형은 특정 타입(concrete type)이기 때문에 컴파일 에러를 피할 수 있습니다.

**Type erasing a protocol**

프로토콜을 특정 타입으로 사용할 수 없을 때 위와 같이 열거형을 사용하는 방식은 유용합니다.
하지만 너무 많은 타입을 섞으려 할 경우 너무 많은 케이스를 가진 열거형을 만들어 혼란스럽습니다.

또한 프로토콜은 테스트를 위한 의존성 주입을 위해 꼭 필요합니다. (열거형으로 대체되기 어려울 수 있습니다.)

이때 우리는 Type erasing을 사용해 프로토콜을 사용하면서 프로토콜을 따르는 타입들을 섞어서 사용할 수 있습니다.
Type erasing으로 컨테이너에 타입을 감싸서 컴파일 타임 프로토콜을 런타임으로 옮길 수 있습니다.

그렇다면 PockerGame 예시를 Type erasing 한 구조를 그림으로 살펴봅시다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/7cc1901f-2ec6-4fe6-8a7b-b773d0508b77)

Type erasing으로 프로토콜을 따르는 변수를 감싸는 컨테이너는 Any를 붙인 변수명으로 대게 선언합니다.

위 그림과 같이 Type erasing은 프로토콜을 따른는 변수를 내부 값을 구조체로 감싸는 개념입니다.
PokerGame 예시에서도 PokerGame 프로토콜을 따르는 변수(타입)를 만들어 AnyPokerGame 구조체로 감싸고 있습니다.

이때 AnyPokerGame 구조체도 PokerGame 프로토콜을 따르도록 하여 AnyPokerGame 객체에 PokerGame 프로토콜이 제공하는 함수를 호출했을 때 함수 호출을 내부 변수로 전달합니다.

Type erasing을 통해 외부 컨테이너는 구조체로 특정 타입(concrete type)이기 때문에 Array, Set, 그리고 Dictionary의 Key로 넣을 수 있습니다.
또한 외부 컨테이너의 내부 값이 PokerGame 프로토콜을 따르는 변수로 PokerGame 프로토콜의 자식 프로토콜을 섞어서 저장할 수 있게 됩니다.

Each AnyPokerGame can store a different PokerGame type.

이제 AnyPokerGame은 구조체로 특정 타입(concrete type)이 되었기 때문에 AnyPokerGame을 가진 Arrary, Set, Dictionary 등에 넣어 AnyPokerGame의 내부 값을 PokerGame 프로토콜을 따르는 다양한 타입을 저장할 수 있습니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/def7b28a-0489-4650-8934-7a75705287d5)

이번에는 코드로 AnyPokerGame이 동작하는 방식을 살펴봅시다.
이제 AnyPokerGame 타입을 Array, Set, Dictionary key로 사용 가능합니다.

```swift
let studPoker = StudPoker()
let holdEm = TexasHoldem()

// You can mix multiple poker game inside an array.
// Wrapper type으로 추상화합니다.
let games: [AnyPokerGame] = [
  AnyPokerGame(studPoker),
  AnyPokerGame(holdEm)
]

games.forEach { (pokerGame: AnyPokerGame) in
  // Wrapper type의 start 함수 호출 시 inner value의 start 함수가 호출되도록 Wrapper type에 구현해야 합니다.
  pokerGame.start()
}

// You can store them inside a Set, too
let setOfGames: Set<AnyPokerGame> = [
  AnyPokerGame(studPoker),
  AnyPokerGame(holdEm)
]

// You can even use poker games as keys!
var numberOfPlayers = [
  AnyPokerGame(studPoker): 300,
  AnyPokerGame(holdEm): 400
]
```

이전 챕터에서 살펴본 AnyError(Result 속 error에 들어가는 모든 타입에 대응, 모든 error를 대표 가능)와 AnyIterator 모두 type erased가 사용된 것들입니다.

다시 말해, PokerGame을 감싸는 AnyPokerGame 구조체는 내부 값으로 PokerGame 프로토콜을 따르는 변수를 가지고 AnyPokerGame 또한 PokerGame을 채택합니다.

AnyPokerGame이 PokerGame을 채택하여 PokerGame이 제공하는 함수가 호출되었을 때 AnyPokerGame에서 내부 값으로 함수 호출을 전달할 수 있습니다.

이제 AnyPokerGame 구조체의 구현부를 살펴봅시다.

```swift
struct AnyPokerGame: PokerGame {

  private let _start: () -> Void
  private let _hashable: AnyHashable

  init<Game: PokerGame>(_ pokerGame: Game) {
    _start = pokerGame.start
    _hashable: AnyHashable(pokerGame)
  }

  func start() {
    _start()
  }
}

extension AnyPokerGame: Hashable {
  func hash(into hasher: inout Hasher) {
    _hashable.hash(into: &hasher)
  }

  static func ==(lhs: AnyPokerGame, rhs: AnyPokerGame) -> Bool {
    return lhs._hashable == rhs._hashable
  }
}
```

PokerGame 프로토콜이 Hashable 프로토콜을 따르기 때문에 AnyPokerGame 구조체도 Hashable 프로토콜을 따르도록 해야합니다. 

이때 type erased가 사용된 AnyHashable을 사용해 Hashable 프로토콜이 제공하는 함수를 AnyPokerGame에서도 제공할 수 있습니다.

AnyPokerGame 구현부처럼 함수의 호출을(start 함수 호출) Wrapped type의 inner value로 전달해야 합니다.

PokerGame 프로토콜이 start 함수만 가졌기 때문에 AnyPokerGame이 start 함수만 모방했지만, 더 많은 함수를 PokerGame 프로토콜이 가졌다면 모두 모방해서 함수 호출을 inner value로 전달해야 합니다.

AnyPokerGame wraps any PokerGame type, and now you're free to use AnyPokerGame inside Collections.

## An alternative to protocols

프로토콜은 지금까지 살펴봤던 것처럼 유용합니다.
특히 복잡한 API를 구현할 때 API 사용하는 곳에서 알지 못하도록 복잡성을 숨길 수 있습니다.

하지만 오히려 프로토콜의 오용으로 프로젝트에 불필요한 복잡성을 증가시킵니다.

따라서 지금부터는 프로토콜의 대안으로 **제네릭 구조체**를 살펴볼 예정입니다.

먼저 프로토콜을 사용하여 데이터의 유효성 여부를 검사하는 Validator 구조체를 구현해봅시다.

```swift
protocol Validator {
  associatedtype Value
  func validate(_ value: Value) -> Bool
}

struct MinimalCountValidator: Validator {
  let minimalChars: Int

  func validate(_ value: String) -> Bool {
    guard minimalChars > 0 else { return true }
    // isEmpty is faster than count check
    guard !value.isEmpty else { return false }
    return value.count >= minimalChars
  }
}

let validator = MinimalCountValidator(minimalChars: 5)
validator.validate("1234567890") // true
```

프로토콜을 사용한 방식은 프로토콜이 가진 연관 값의 타입에 따라 다른 프로토콜 구현부를 구현해야 합니다.

**Creating a generic struct**

이때 Validator를 프로토콜이 아닌 제네릭 구조체로 만들면, 각 타입에 대응하는 구현부를 만들 필요 없이 여러 타입에 대응하는 하나의 구현부를 만들 수 있습니다.

아래 코드와 같이 Validator 제네릭 구조체를 만들었습니다.

```swift
struct Validator<T> {
  let validate: (T) -> Bool

  init(validate: @escaping (T) -> Bool) {
    self.validate = validate
  }
}  
```

제네릭 구조체에서는 유효성의 조건을 클로저로 입력 받아 여러 타입을 하나의 구현부로 대응할 수 있게 됩니다.

또한 Validator 구조체를 확장하여 함수도 추가할 수 있습니다.

```swift
extension Validator {
  func combine(_ other: Validator<T>) -> Validator<T> {
    let combinedValidator = Validator<T>(validate: { (value: T) -> Bool in
      let ownResult = self.validate(value)
      let otherResult = other.validate(value)
      return ownResult && otherResult
    })
    return combinedValidator
  }
}

let notEmpty = Validator<String>(validate: { string -> Bool in
  return !string.isEmpty
})

let maxTenChars = Validator<String>(validate: { string -> Bool in
  return string.count <= 10
})

let combinedValidator: Validator<String> = notEmpty.combine(maxTenChars)
combinedValidator.validate("")  // false
combinedValidator.validate("Hi")  // true
combinedValidator.validate("This one is way too long")  // false 
```

이처럼 제네릭 구조체는 프로토콜의 대안이 될 수 있습니다.

**Rules of thumb for polymorphism**

Here are some heuristics to keep in mind when reasoning about polymorphism in Swift.

- Light-weight polymorphism -> Use enums.
- A type that needs to work with multiple types -> Make a generic type.
- A type that needs a single configurable implementation -> Store a closure.
- A type that works on multiple types and has a single configurable implementation -> Use a generic struct or class that stores a closure.
- When you need advanced polymorphism, default extensions, and other advanced use cases -> Use protocols.

## Summary
- You can use protocols as an interface to swap out implementations, for testing, or for other use cases.
- An associated type can resolve to a type that you don't own.
- With conditional conformance, a type can adhere to a protocol, as long as its generic type or associated type adheres to this protocol.
- Conditional conformance works well when you have a generic type with very few constraints.
- A protocol with associated types or Self requirements can't be used as a concrete type.
- Sometimes, you can replace a protocol with an enum, and use that as a concrete type.
- You can use a protocol with associated types or Self requirements at runtime via type erasure.
- Often a generic struct is an excellent alternative to a protocol.
- Combining a higher-order function with a generic struct enables you to create hightly flexible types.
