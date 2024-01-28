# Protocol extensions

## This chapter covers
- Flexibly modeling data with protocols instead of subclasses
- Adding default behavior with protocol extensions
- Extending regular types with protocols
- Working with protocol inheritance and default implementations
- Applying protocol composition for highly flexible code
- Showing how Swift prioritizes method calls
- Extending types containing associated types
- Extending vital protocols, such as Sequence and Collection

## Class inheritance vs. protocol inheritance

객체지향 프로그래밍은 서브 클래싱을 사용해 전형적으로 데이터를 모델링하는 방법입니다.
하지만 서브 클래싱의 수직적 데이터 모델링은 다소 엄격합니다.
클래스 기반의 상속을 대신해서 스위프트는 프로토콜 기반의 상속을 제공합니다.

클래스 기반의 상속은 수직적 데이터 구조를 만들고 프로토콜 기반의 상속은 수평적 데이터 구조를 이룹니다.

클래스 기반의 상속에서 부모 클래스를 상속한 자식 클래스에 새로운 함수를 추가하거나 부모 클래스의 함수를 오버라이드 할 수 있습니다.

만약, 네트워크 호출을 위한 URLRequest 타입을 생성하는 RequestBuilder 타입을 서브 클래싱 방식으로 만들어 봅시다.

RequestBuilder 클래스를 최상위 부모 클래스로 두고, RequestBuiler 클래스를 상속하는 RequestHeaderBuilder 자식 클래스를 만들어 RequestHeaderBuilder 클래스에서 header를 붙이는 작업을 추가합니다. 
또한 RequestHeaderBuilder 클래스를 상속한 EncryptedRequestHeaderBuilder 자식 클래스를 만들고 네트워크 요청의 데이터를 암호화하는 기능까지 추가합니다.

**RequestBuilder**

**RequestHeaderBuilder(subclass)**

**EncryptedRequestHeaderBuilder(subclass)**

이처럼 클래스 기반 상속은 수직적(vertical) 데이터 구조를 형성합니다.

하지만 클래스 기반 상속과 달리 프로토콜 기반의 상속을 통해 RequestBuilder 타입을 만든다면 수평적(horizontal) 데이터 구조를 형성할 수 있습니다.

**RequestBuilder** | **HeaderBuilder** | **Encryptable**

RequestBuilder를 비롯해 나머지 프로토콜을 따르면 해당 프로토콜에서 제공하는 함수를 사용 가능합니다.
enum, struct, class, subclass 상관 없이 프로토콜을 따를 수 있습니다.

프로토콜을 통해 기능을 분리하여 특정 기능이 필요할 때 해당 기능을 지원하는 프로토콜을 채택하는 방식으로 새로운 함수를 추가할 수 있습니다.
또한 프로토콜 상속을 통해서 새로운 함수를 추가하고 부모 프로토콜의 함수를 오버라이드 할 수 있습니다.

프로토콜로 기능을 분리하는 방식은 코드 유연성과 재사용성이 높아집니다.
프로토콜을 통해 더 이상 하나의 부모 클래스에 제한되지 않습니다.

**Creating a protocol extension**

프로토콜에서는 함수를 정의할 수만 있고 프로토콜을 채택한 타입에서 해당 함수를 구현해야 합니다. 
하지만 프로토콜 확장을 통해 프로토콜에서 정의한 함수의 구현부(default implementation)를 추가할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
protocol RequestBuilder {
  var baseURL: URL { get }
  func makeRequest(path: String) -> URLRequest
}

extension RequestBuilder {
  func makeRequest(path: String) -> URLRequest {
    let url = baseURL.appendingPathComponent(path)
    var request = URLRequest(url: url)
    request.httpShouldHandleCookies = false
    request.timeoutInterval = 30
  }
}
```

위 코드와 같이 프로토콜 함수에서 정의한 함수의 default 구현부는 extension에 위치해야 합니다.
또한 extension에서 구현하는 함수 구현부에서는 당연히 프로토콜의 프로퍼티에 접근할 수 있습니다.

이제 RequestBuilder 프로토콜을 채택한 타입에서는 추가적인 makeRequest 함수 구현 없이 makeRequest 함수를 사용할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
struct BikeRequestBuilder: RequestBuilder {
  let baseURL: URL = URL(string: "https://www.biketriptracker.com")!
}

let bikeRequestBuilder = BikeRequestBuilder()
let request = bikeRequestBuilder.makeRequest(path: "/trips/all")
print(request)  // https://www.biketriptracker.com/trips/all
```

BikeRequestBuilder 타입은 RequestBilder 프로토콜을 채택하여 makeRequest 함수의 구현부(default implementation)를 얻을 수 있습니다.

**Multiple extension**

클래스 상속의 경우 자식 클래스가 하나의 부모 클래스에만 상속 받을 수 있습니다.
하지만 프로토콜의 경우 하나의 타입이 여러 프로토콜을 채택할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
enum ResponseError: Error {
  case invalidResponse
}

protocol ResponseHandler {
  func validate(response: URLResponse) throws
}

extension ResponseHandler {
  func validate(response: URLResponse) throws {
    guard let httpresponse = response as? HTTPURLResponse else {
      throw ResponseError.invalidResponse
    }
  }
}

// BikeAPI 타입이 여러 프로토콜을 동시에 채택할 수 있습니다.
class BikeAPI: RequestBuilder, ResponseHandler {
  let baseURL: URL = URL(string: "https://www.biketriptracker.com")!
}
```

## Protocol inheritance vs. Protocol composition

지금까지 프로토콜 확장을 통해 프로토콜 함수의 구현부(default implementation)를 제공하는 방법을 살펴봤습니다.

프로토콜 상속과 프로토콜 컴포지션도 데이터 모델링에 유용하게 쓰입니다.

프로토콜 상속과 프로토콜 컴포지션을 살펴보고 차이점을 확인하기 위해 이메일을 보내는 SMTP 프레임워크를 구현하려 합니다.
먼저 프로토콜 상속 방식으로 SMTP를 구현하고 이후 프로토콜 컴포지션 방식으로 다시 구현하여 trade-offs를 확인하겠습니다.

먼저 Email 구조체와 Mailer 프로토콜을 만들겠습니다.
아래 코드를 살펴봅시다.

```swift
// MailAddress가 단순 String 보다 더욱 의도를 드러냅니다.
struct MailAddress {
  let value: String
}

struct Email {
  let subject: String
  let body: String
  let to: [MailAddress]
  let from: MailAddress
}

protocol Mailer {
  func send(email: Email)
}

// Mailer 프로토콜을 확장하여 send 함수의 구현부를 제공합니다.
extension Mailer {
  func send(email: Email) {
    // Omitted: Connect to server
    // Omitted: Submit email
    print("Email is sent!")
  }
}
```

이제 Mailer 프로토콜의 확장에서 구현한 함수의 구현부(default implementation)에 메일을 보내기 이전 메일의 유효성 검사를 추가하려 합니다.
그러나 Mailer 프로토콜의 따르는 모든 타입에서 메일의 유효성 검사를 요구하지 않습니다.
특정 타입에서만 Mailer 프로토콜을 따르며 추가적으로 메일의 유효성 검사를 요구합니다.

먼저 프로토콜 상속을 사용해 위와 같은 조건을 구현해봅시다.

Mailer 프로토콜을 상속해서 ValidatingMailer 자식 프로토콜을 생성하여 ValidatingMailer 프로토콜에 유효성 검사 기능을 추가합니다.

아래 코드로 살펴봅시다.

```swift
protocol Mailer {
  func send(email: Email)
}

// Mailer 프로토콜을 확장하여 send 함수의 구현부를 제공합니다.
extension Mailer {
  func send(email: Email) {
    // Omitted: Connect to server
    // Omitted: Submit email
    print("Email is sent!")
  }
}

protocol ValidatingMailer: Mailer {
  func send(email: Email) throws  // 부모 프로토콜의 send 함수를 오버라이드합니다.
  func validate(email: Email) throws
}

extension ValidatingMailer {
  func send(email: Email) throws {
    try validate(email: Email)
    // Connect to Server
    // Submit email
    print("Email validated and sent.")
  }

  func validate(email: Email) throws {
    // Check email address, and whether subject is missing.
  }
}

struct SMTPClient: ValidatingMailer {
  // Implementation omitted.
}

let client = SMTPClient()
try? client.send(email: Email(subject: "Learn Swift", body: "Lorem ipsum", to: [MailAddress(value: "john@naver.com")], from: MailAddress(value: "Stranger@naver.com")))
```

Mailer 프로토콜의 자식 프로토콜인 ValidatingMailer에서 send 함수를 오버라이드 했기 때문에 ValidationMailer 프로토콜을 확장해 오버라이드한 send 함수의 구현부를 제공해야 합니다.
프로토콜을 상속하여 자식 프로토콜에 요구사항에 필요한 validate 함수를 추가합니다.

하지만 ValidatingMailer 프로토콜에서는 메일의 유효성 검사와 메일을 보내는 두 가지 성격의 일을 동시에 처리하게 됩니다.
이처럼 프로토콜 상속을 통해 데이터를 모델링할 때 의미 단위로 프로토콜이 분리되지 않는다는 단점이 존재합니다.

만약 메일 유효성 검사를 필요로 하지 않은 타입에서는 ValidatingMailer가 아닌 Mailer 프로토콜을 채택해야만 합니다.

그렇다면 프로토콜 상속이 아닌 프로토콜 컴포지션을 사용하면 어떨까요?

프로토콜 컴포지션 방식은 Mailer 프로토콜을 상속하는 ValidatingMailer 프로토콜을 생성하지 않고, 메일 유효성을 검사하는 독립된 MailValidator 프로토콜을 생성합니다.
이후 메일 유효성 검사 기능을 필요로 하는 타입에 Mailer 프로토콜과 함께 MailValidator 프로토콜을 다중 채택하여 메일 유효성 검사 기능을 제공합니다.

아래 코드는 MailValidator 프로토콜을 구현한 코드입니다.

```swift
protocol MailValidator {
  func validate(email: Email) throws
}

extension MailValidator {
  func validate(email: Email) throws {
    // Omitted: Check email address, and whether subject is missing.
  }
}

struct SMTPClient: Mailer, MailValidator {
  // Implementation omitted.
}
```

이제는 Mailer 프로토콜에서 MailValidator 프로토콜에 대해 모르게 되고.
MailValidator 프로토콜도 Mailer 프로토코르이 존재를 모릅니다.

**Protocol intersection(교차점)**

두 개의 프로토콜이 있을 때 두 프로토콜의 교차점에서 적용되는 확장을 구현(추가)할 수 있습니다.

예를 들어 Mailer 프로토콜과 MailValidator 프로토콜을 모두 따르는 SMTPClient 타입이 두 프로토콜의 교차점에 있는 타입이라고 볼 수 있습니다.
Mailer 프로토콜과 MailValidator 프로토콜의 교차점을 확장했다면 확장된 기능을 교차점에 있는 SMTPClient 타입이 사용할 수 있습니다.

두 프로토콜의 교차점을 확장하기 위해서는 **Self 키워드**를 통해 둘 중 하나의 프로토콜을 확장하여 나머지 한 프로토콜을 따르도록 해야 합니다.

아래 코드는 Mailer 프로토콜과 MailValidator 프로토콜의 교차점을 확장한 코드입니다.

```swift
extension MailValidator where Self: Mailer {
  func send(email: Email) throws {
    try validate(email: email)
    // Connect to server
    // Submit email
    print("Email Validated and sent.")
  }
}
```

두 프로토콜의 교차점에서 send 함수의 구현부(default implementation)를 제공하여 Mailer 프로토콜에서 제공하는 send 함수를 오버라이드하고 있습니다.
위에서는 MailValidator를 확장하고 Mailer 프로토콜을 채택했지만, 반대로 Mailer 프로토콜을 확장하고 MailValidator 프로토콜을 채택해도 상관 없습니다.

이제 두 프로토콜을 채택하는 SMTPClient 타입에서 send 함수의 구현부는 프로토콜의 교차점에서 제공하게 됩니다.

프로토콜 교차점에서 기존 함수를 오버라이드하는 경우외에도 새로운 함수를 추가할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
extension MailValidator where Self: Mailer {
  func send(email: Email) throws {
    try validate(email: email)
    // Connect to server
    // Submit email
    print("Email Validated and sent.")
  }

  // 새로운 send(email:, at:) 함수
  func send(email: Email, at: Date) throws {
    try validate(email: email)
    // Connect to server
    // Submit email
    print("Email Validated and sent.")
  }
}
```

실제로 Mailer 프로토콜과 MailValidator 프로토콜을 모두 채택하여 두 프로토콜의 교차점에 있는 SMTPClient 프로토콜에서 교차점 확장의 함수들을 사용하는 모습을 살펴봅시다.

```swift
struct SMTPClient: Mailer, MailValidator {}

let client = SMTPClient()
let email = Email(subject: "Learn Swift", body: "Lorem ipsum", to: [MailAddress(value: "john@naver.com")], from: MailAddress(value: "Stranger@naver.com"))

try? client.send(email: email)
try? client.send(email: email, at: Date(timeIntervalSinceNow: 3600))
```

두 프로토콜을 채택한 타입도 두 프로토콜의 교차점에 있지만, 제네릭 타입에서 한 타입이 두 프로토콜로 타입 제약된 경우에도 해당 타입이 두 프로토콜의 교차점에 있다고 볼 수 있습니다.
따라서 두 프로토콜로 타입이 제약된 제네릭 타입에서도 교차점 확장의 함수들을 사용할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
func submitEmail<T>(sender: T, email: Email) where T: Mailer, T: MailValidator {
  try? sender.send(email: email, at: Date(timeIntervalSinceNow: 3600))
}
```

프로토콜 컴포지션을 사용하면 의미 단위로 코드를 분리할 수 있습니다.

하지만 너무 잘게 분리될 경우 오히려 단점으로 적용됩니다.
또한 프로토콜 상속과 달리 컴포지션 방식의 경우 여러 프로토콜을 다중 채택해야 합니다.

물론 프로토콜 상속은 단일 프로토콜을 채택하도록 하지만 경직된 데이터 구조를 형성하게 됩니다.

프로토콜 상속과 프로토콜 컴포지션 방식의 균형을 맞춰 추상화를 이루도록 노력해야 합니다.

## Overriding priorities

**Overriding a default implementation**

프로토콜에 선언된 함수를 extension에서 구현부(default implementation)를 제공하는 방법을 살펴봤습니다.
그렇다면 프로토콜 확장에서 추가한 함수 구현부를 프로토콜을 채택한 타입에서 오버라이드하면 어떻게 될까요?

grow 함수를 가진 Tree 프로토콜을 구현하여 프로토콜 상속 과정을 살펴봅시다.
Tree 프로토콜을 확장하여 grow 함수를 구현하고 Oak 구조체에서 Tree 프로토콜을 채택하려 합니다.
이때 Oak 구조체에서 grow 함수를 오버라이드하는 상황입니다.

만약 Oak 구조체에서 grow 함수를 오버라이드 하지 않는다면 Oak 구조체가 채택한 Tree 프로토콜의 grow 함수를 호출합니다.
하지만 Oak 구조체에서 grow 함수를 오버라이드 한다면 Oak 구조체에서 호출되는 grow 함수는 Oak 구조체에서 오버라이드한 grow 함수가 호출됩니다.
(If a type implements the same method as the one on a protocol extension, Swift ignores the protocol extension's method.)

프로토콜의 확장에서 구현한 함수를 프로토콜을 채택하는 타입에서 오버라이드 할 수 있지만, 반대로 프로토콜 확장을 통해 실제 타입의 함수를 오버라이드 할 수는 없습니다.

**Overriding with protocol inheritance**

지금부터는 프로토콜이 상속될 때 확장에서 추가한 함수 구현부(default implementation)가 상속되는 규칙을 알아볼 예정입니다.

위에서 봤던 Tree 프로토콜을 Plant 프로토콜의 상속을 받는 자식 프로토콜로 만들고 Oak 구조체가 Tree 프로토콜을 채택하도록 구현하면 Oak 구조체가 호출하는 grow 함수가 무엇인지 살펴봅시다.

**스위프트는 가장 특수화된 구현을 고르는 특성이 있습니다.**

여기서 특수화된 구현이라는 설명이 이해 되지 않을 수 있습니다.
특수화된 구현은 자신과 가장 가까운 구현을 뜻하는 말입니다.

부모 프로토콜인 Plant 프로토콜과 Plant 프로토콜을 상속 받는 Tree 자식 프로토콜이 있고 Oak 구조체가 Tree 프로토콜을 채택하는 상황에서 Plant 프로토콜, Tree 프로토콜, 그리고 Oak 구조체 모두 grow 함수를 오버라이드 했다면
Oak 구조체에서 가장 가까운 grow 함수 구현부는 본인이 가진 grow 함수라고 볼 수 있습니다.
따라서 Oak 구조체에서 호출하는 grow 함수는 본인이 오버라이드한 함수입니다.

만약 Plant 프로토콜과 Tree 프로토콜에만 grow 함수를 오버라이드 했다면, Tree 프로토콜을 채택하는 Oak 구조체에서 호출되는 grow 함수는 Tree 프로토콜의 grow 함수가 될 것 입니다.

마지막으로 Plant 프로토콜에만 grow 함수를 구현했다면 당연히 Oak 구조체에서 호출되는 grow 함수는 Plant 프로토콜의 grow 함수가 됩니다.

아래 코드로 프로토콜 확장이 함수 구현부(default implementation)를 가질 때, 해당 프로토콜을 상속하고 오버라이드하는 과정을 살펴봅시다.

```swift
func growPlant<P: Plant>(_ plant: P) {
  plant.grow()
}

protocol Plant {
  func grow()
}

extension Plant {
  func grow() {
    print("Growing a plant")
  }
}

protocol Tree: Plant {}

extension Tree {
  func grow() {
    print("Growing a tree")
  }
}

struct Oak: Tree {
  func grow() {
    print("The mighty oak is growing")
  }
}

struct CherryTree: Tree {}

struct KiwiPlant: Plant {}

growPlant(Oak())  // The mighty oak is growing
growPlant(CherryTree())  // Growing  a tree
growPant(KiwiPlant())  // Growing a plant
```

클래스에서의 상속과 오버라이드 규칙과 비슷하게 프로토콜 상속과 오버라이드 규칙도 유사하게 동작한다는 사실을 알 수 있습니다.
이와 같은 프로토콜의 동작은 클래스, 구조체, 그리고 열거형에 관계 없이 동일하게 적용됩니다.

## Extending in two directions

**Opting in to extensions**

모든 타입이 특정 프로토콜의 요구사항을 원하지 않을 경우는 많습니다.
따라서 프레임워크에 확장을 추가할 때는 항상 주의해야 합니다.

예를 들어 사용자의 동작을 분석하는 AnalyticsProtocol이 있다고 생각해 봅시다.
또한 AnalyticsProtocol에서는 프로토콜 확장을 통해 함수 구현부(default implementation)까지 제공하고 있습니다.

이때 UIViewController 타입이 AnalyticsProtocol을 채택하면 모든 UIViewController에서 AnalyticsProtocol이 제공하는 기능을 사용하게 됩니다.
심지어 UIViewController 타입의 자식 클래스에 까지 AnalyticsProtocol의 기능을 제공하게 됩니다.

모든 UIViewController에서 해당 기능을 필요로 하지 않고 일부 ViewController에 필요한 기능입니다.

하지만 개발자는 UIViewController 타입이 AnalyticsProtocol의 기능을 당연히 가졌다는 생각을 하지 못합니다.
만약 개발자가 AnalyticsProtocl의 기능을 인지하지 못하고 UIViewController 타입에 AnalyticsProtocol과 동일한 기능을 추가하는 충돌 상황까지 발생할 수 있습니다.

이와 같은 이슈를 해결하기 위해 확장을 접어야 합니다. (flip the extension)

확장을 접는다는 말은 프로토콜 교차점을 확장하는 것과 같습니다.
다시 말해, AnalyticsProtocol과 UIViewController의 교차점을 확장하여 기존에 AnalyticsProtocol에서 제공하려는 기능을 추가하는 방법입니다.

아래 코드로 살펴봅시다.

```swift
protocol AnalyticsProtocol {
  func track(event: String, parameters: [String: Any])
}

// Not like this:
extension UIViewController: AnalyticsProtocol {
  func track(event: String, parameters: [String: Any]) { // ...snip }
}

// But as follows:
extension AnalyticsProtocol where Self: UIViewController {
  func track(event: String, parameters: [String: Any]) { // ...snip}
}
```

타입이 프로토콜을 채택하였던 기존 방식이 아닌 프로토콜을 확장하여 타입을 채택하도록 구현합니다.

이제 확장을 접는 방식을 통해 뷰컨트롤러 중 UIViewController 타입과 AnalyticsProtocol을 모두 따르는 뷰컨트롤러에서만 track 함수와 함수 구현부를 사용할 수 있습니다.
모든 뷰컨트롤러가 AnalyticsProtocol을 따르는 상황을 막을 수 있습니다.

```swift
extension NewsViewController: UIViewController, AnalyticsProtocol {
  // ...snip

  override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    // UIViewController와 AnalyticsProtocl을 모두 채택해 두 타입의 교차점에 있기 때문에 교차점을 확장한 함수(track)를 사용할 수 있습니다.
    track("News.appear", params: [:])  
  }
}
```

Extensions are not namespaced, so be careful with adding public extensions inside a framework, because implementers may not want their classes to adhere to a protocol by default.

## Extending with associated types

연관 값을 가진 프로토콜의 함수가 호출되는 방식을 살펴봅시다.

만약 배열(Array)에 중복 값을 제거하는 unique 함수를 적용하고 싶을 때를 예시로 살펴봅시다.
아래 코드와 같이 앞으로 구현할 unique 함수는 배열 속의 중복된 값을 제거하고 유일한 값만 가진 배열로 만듭니다.

```swift
[3, 2, 1, 1, 2, 3].unique()  // [3, 2, 1]
```

배열은 Element라는 연관 값을 가진 구조체입니다.
따라서 unique 함수도 Element 연관 값을 다뤄야 합니다.

먼저 배열을 확장하여 unique 함수를 추가 해보겠습니다.
이때 unique 함수에서 각 element들을 비교하기 위해서 Element 연관 값은 Equatable 프로토콜로 타입 제약 되어야 합니다.

아래 코드로 살펴봅시다.

```swift
extension Array where Element: Equatable {
  func unique() -> [Element] {
    var uniqueValues = [Element]()
    for element in self {
      if !uniqueValues.contains(element) {
        uniqueValues.append(element)
      }
    }
    return uniqueValues
  }
}
```

Equatable로 Element 연관 값을 타입 제약했기 때문에 Element 연관 값의 타입은 Equatable 프로토콜을 따라야만 unique 함수를 사용할 수 있습니다.
(여기서 궁금증은 Element 연관 값이 Equatable 프로토콜을 따라야만 unique 함수를 사용할 수 있는지, unique 함수의 사용 여부와 관계 없이 Element 연관 값이 Equatable 프로토콜을 따르지 않으면 에러가 발생하는지 입니다. 전자일것 같긴하지만...)

배열을 확장해 unique 함수를 추가하는 방법은 좋은 시작입니다.

그렇다면 저차원으로 내려가 Collection 프로토콜을 따르는 다른 타입에도 unique 함수를 제공하기 위해 Collection 프로토콜을 확장해 unique 함수를 추가해 봅시다.
물론 Collection 프로토콜의 Element 연관 값도 Equatable 타입으로 제약해야 합니다.

아래 코드를 살펴봅시다.

```swift
extension Collection where Element: Equatable {
  func unique() -> [Element] {
    var uniqueValues = [Element]()
    for element in self {
      if !uniqueValues.contains(element) {
        uniqueValues.append(element)
      }
    }
    return uniqueValues
  }
}
```

이제 Array 보다 저차원인 Collection 프로토콜을 확장하여 unique 함수를 추가했기 때문에 더 많은 타입에서 unique 함수를 사용할 수 있습니다.
여기서 Array가 Collection 보다 저차원인 이유는 Collection 프로토콜의 자식 프로토콜들을 Array가 따르기 때문입니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/f87cec24-1de8-47f8-83fb-8a1713ff1872)

물론 Collection 보다 더 저차원인 Sequence 프로토콜도 존재합니다.

unique 함수를 아래 코드와 같이 사용할 수 있습니다.

```swift
// Array still has unique()
[3, 2, 1, 1, 2, 3].unique()  // [3, 2, 1]

// Strings can be unique() now, too
"aaaaaaabcdef".unique()  // ["a", "b", "c", "d", "e", "f"]

// Or a Dictionary's values
let uniqueValues = [
  1: "Waffle",
  2: "Banana",
  3: "Pancake",
  4: "Pancake",
  5: "Pancake"
].values.unique()
print(uniqueValues)  // ["Banana", "Pancake", "Waffle"]
```

이렇게 여러 타입에 unique 함수를 적용할 수 있습니다.
특정 타입(concrete type)을 확장하지 않고 프로토콜을 확장했기 때문에 얻을 수 있는 이점입니다.

**A specialized extension**

하지만 위에서 구현한 unique 함수는 성능 측면에서 개선할 부분이 있습니다.

각 배열의 요소마다 uniqueValues 배열에 이미 있는 요소인지 확인해야 합니다.
다시 말해 배열 요소 하나마다 uniqueValuew 배열을 모두 순회해야 합니다.

만약 배열로 유일한 요소를 저장하지 않고 Set을 사용해 hash valuew를 통해 값을 비교하고 유일한 요소들을 저장한다면 성능을 개선시킬 수 있습니다.
Set을 사용하려면 Element 연관 값의 타입 제약을 Equatable 프로토콜이 아닌 Hashable 프로토콜로 제약해야 합니다.

Element 연관 값을 Hashable 프로토콜로 타입 제약한 상태에서 Collection 프로토콜 확장에 unique 함수를 추가 구현하게 됩니다.
Element 연관 값을 Equatable 프로토콜로 타입 제약한 상태와 Hashable 프로토콜로 타입 제약한 상태 모두 확장에 구현할 것입니다. 

**연관 값의 상속 관계에서도 스위프트는 가장 특수화된 구현을 고릅니다.**

Equatable 프로토콜을 상속 받은 Hashable 프로토콜을 경우, Equatable 타입을 따르는 요소를 가진 배열은 성능적으로 개선되지 못한 배열을 사용한 위의 unique 함수를 호출하게 되고 
Hashable 타입을 따르는 요소를 가진 배열은 Hashable 프로토콜 제약을 가한 새로운 unique 함수가 없다면 기존의 unique 함수를 호출합니다.

하지만 아래 코드와 같이 Element 연관 값을 Hashable 프로토콜로 제약할 경우 기존의 unique 함수가 아닌 Set을 활용하는 개선된 unique 함수를 호출합니다. 

```swift
extension Collection where Element: Hashable {
  func unique() -> [Element] {
    var set = Set<Element>()
    var uniqueValues = [Element]()
    for element in self {
      if !set.contains(element) {
        uniqueValues.append(element)
        set.insert(element)
      }
    }
    return uniqueValues
  }
}
```

이제 Collection 프로토콜은 두 번의 확장을 통해 연관 값의 타입 제약이 다른 두 함수를 구현했습니다.

Collection<Element>에서 Element의 타입 제약이 다를 때도 스위프트는 가장 특수화된 구현을 고릅니다.

























