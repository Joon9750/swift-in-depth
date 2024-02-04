![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/dcf949e1-a45c-4220-972b-ed827ef936ed)# Swift patterns

## This chapter covers
- Mocking types with protocols and associated types
- Understanding conditional conformance and its benefits
- Applying conditional conformance with custom protocols and types
- Discovering shortcomings that come with protocols
- Understanding type erasure
- Using alternatives to protocols

## Dependency injection

이번 챕터에서는 스위프트 개발에 유용한 패턴들을 살펴볼 예정입니다.
전통적인 객체지향 프로그래밍과 다른 스위프트에 집중한 패턴들을 알아보겠습니다.

먼저 살펴볼 것은 의존성 주입입니다.

의존성 주입에서 말하는 의존성이란 서로 다른 객체 사이에 의존 관계가 있다는 것입니다.
다시 말해, 의존하는 객체가 수정되면 다른 객체도 영향을 받게 됩니다.

또한 의존성 주입에서 주입이란 외부에서 생성자 등의 방법으로 미리 생성한 객체를 넣는것을 뜻합니다.

의존성을 가진 코드가 많으면 코드의 재사용성이 떨어집니다.
재사용을 위해 코드를 수정하면 매번 의존성을 가진 객체들을 함께 수정해야하기 때문에 재사용성이 떨어집니다.

의존성을 강하게 가지는 코드를 해결하기 위해 의존성 주입으로 해결할 수 있습니다.

또한 의존성 주입을 통해 아래와 같은 이점을 얻을 수 있습니다.

1. Unit Test가 용이해진다.
2. 코드의 재활용성을 높여준다.
3. 객체 간의 의존성(종속성)을 줄이거나 없엘 수 있다.
4. 객체 간의 결합도이 낮추면서 유연한 코드를 작성할 수 있다.

의존성 주입은 DIP(의존 관계 역전 법칙)와도 관련이 깊습니다.

의존 관계 역전 법칙은 객체 지향 프로그래밍 설계의 다섯가지 기본 원칙(SOLID) 중 하나입니다. 

추상화 된 것은 구체적인 것에 의존하면 안되고 구체적인 것이 추상화된 것에 의존 해야합니다.
즉, 구체적인 객체는 추상화된 객체에 의존 해야 한다는 것이 핵심입니다.

스위프트에서 말하는 추상화된 객체는 프로토콜로 생각할 수 있습니다.

**Swapping an implementation**

지금부터 의존성 주입을 활용하여 구현부를 교환할 수 있는 WeatherAPI를 구현할 것입니다.
여기서 구현부를 교환하는 것은 의존성 주입에 의해 실현됩니다.

WeatherAPI는 실제 네트워크 세션, 오프라인 네트워크 세션 그리고 테스트 세션을 모두 대응하도록 구현합니다.
이때 세 가지 세션은 주입 받는 객체가 됩니다.

WeatherAPI에 Session 프로토콜을 따르는 타입을 주입 받습니다.

주입 받는 세션의 정확한 타입과 상관없이 Session 프로토콜의 함수를 사용할 수 있습니다. 
의존성 주입을 통해 WeatherAPI는 Session 타입과 의존 관계를 만들지 않게 됩니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/c0e48dae-9db6-48a0-a787-0575b6ed8b96)

WeatherAPI가 가진 Session 프로토콜에 의해 Session 프로토콜을 따르는 URLSession, OfflineSession, MockSession 타입을 주입 받을 수 있습니다.
이를 구현부를 교환한다고 볼 수 있습니다.

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

먼저 OfflinheURLSession과 OfflineTask를 구현하겠습니다.























