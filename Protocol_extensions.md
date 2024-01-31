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

하지만 서브 클래싱이 형성하는 수직적 데이터 구조는 다소 엄격한 성격을 지닙니다.
클래스 기반의 상속을 대신해서 스위프트는 프로토콜 기반의 상속을 제공합니다.

클래스 기반의 상속은 수직적 데이터 구조를 만들고 프로토콜 기반의 상속은 수평적 데이터 구조를 이룹니다.

클래스 기반의 상속에서 부모 클래스를 상속한 자식 클래스에 새로운 함수를 추가하거나 부모 클래스의 함수를 오버라이드 할 수 있습니다.

네트워크 호출을 위한 URLRequest 타입을 생성하는 RequestBuilder 타입을 서브 클래싱 방식으로 만들고 프로토콜을 사용한 방식으로도 만들어봅시다.

먼저 서브 클래싱 방식으로 RequestBuilder 타입을 구현해 봅시다.

RequestBuilder 클래스를 최상위 부모 클래스로 두고, RequestBuiler 클래스를 상속하는 RequestHeaderBuilder 자식 클래스를 만들어 RequestHeaderBuilder 클래스에서 header를 붙이는 작업을 추가합니다. 

또한 RequestHeaderBuilder 클래스를 상속한 EncryptedRequestHeaderBuilder 자식 클래스를 만들고 네트워크 요청의 데이터를 암호화하는 기능까지 추가합니다.

이와 같이 서브 클래싱을 하며 자식 클래스에 필요한 기능을 추가하는 방식으로 데이터 구조를 형성합니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/5346bc05-2af9-49be-9166-d9a16b0f775a)

이처럼 클래스 기반 상속은 수직적(vertical) 데이터 구조를 형성합니다.

하지만 클래스 기반 상속과 달리 프로토콜 기반의 상속을 통해 RequestBuilder 타입을 만든다면 수평적(horizontal) 데이터 구조를 형성할 수 있습니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/809509e1-51a7-4ad6-817a-f82d9ad1dd78)

RequestBuilder를 비롯해 나머지 프로토콜을 따르면 해당 프로토콜에서 제공하는 함수를 사용 가능합니다.
enum, struct, class, subclass 상관 없이 모두 프로토콜을 따를 수 있습니다.

프로토콜을 통해 기능을 분리하여 특정 기능이 필요할 때 해당 기능을 지원하는 프로토콜을 채택하는 방식으로 새로운 기능(함수)를 추가할 수 있습니다.
또한 프로토콜 상속을 통해서 새로운 함수를 추가하고 부모 프로토콜의 함수를 오버라이드 할 수 있습니다.

프로토콜로 기능을 분리하는 방식은 코드 유연성과 재사용성을 높입니다.

서브 클래싱 방법은 하나의 부모 클래스에 제한된 자식 클래스를 가지지만, 프로토콜을 활용한다면 하나의 부모 클래스에 제한 되지 않을 수 있습니다.

**Creating a protocol extension**

프로토콜에서는 함수를 정의할 수만 있고, 프로토콜을 채택한 타입에서 프로토콜에 정의된 함수를 구현해야 합니다.

하지만 프로토콜 확장을 통해 프로토콜에서 정의한 함수의 구현부(default implementation)를 제공할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
protocol RequestBuilder {
  var baseURL: URL { get }
  // makeRequest 함수 정의
  func makeRequest(path: String) -> URLRequest
}

extension RequestBuilder {
  // makeRequest 함수 구현
  func makeRequest(path: String) -> URLRequest {
    let url = baseURL.appendingPathComponent(path)
    var request = URLRequest(url: url)
    request.httpShouldHandleCookies = false
    request.timeoutInterval = 30
  }
}
```

위 코드와 같이 프로토콜 함수에서 정의한 함수의 구현부(default implementation)를 제공하기 위해서는 프로토콜을 확장하여 함수 구현부를 제공해야 합니다.

또한 확장에서 구현하는 함수 구현부에서는 프로토콜의 프로퍼티에 접근할 수 있습니다.

이제 RequestBuilder 프로토콜을 채택한 타입에서는 makeRequest 함수 구현 없이 RequestBuiler 프로토콜 확장에서 제공하는 makeRequest 함수 구현부를 사용할 수 있습니다.

물론 RequestBuiler 프로토콜을 채택한 타입에서 makeRequest 함수를 직접 구현하여 사용할 수 있습니다. 
이때는 RequestBuiler 프로토콜의 확장에서 구현된 makeRequest 함수가 아닌 해당 타입에서 직접 구현한 makeRequest 함수를 호출합니다. 

프로토콜 확장에서 제공하는 함수 구현부를 오버라이드하는 내용은 이후에 더 살펴볼 예정입니다.

아래 코드는 프로토콜 확장에서 제공하는 함수의 구현부를 해당 프로토콜을 따르는 타입에서 사용하는 코드입니다.

```swift
struct BikeRequestBuilder: RequestBuilder {
  let baseURL: URL = URL(string: "https://www.biketriptracker.com")!
}

let bikeRequestBuilder = BikeRequestBuilder()
let request = bikeRequestBuilder.makeRequest(path: "/trips/all")
print(request)  // https://www.biketriptracker.com/trips/all
```

BikeRequestBuilder 타입은 RequestBilder 프로토콜을 채택하여 makeRequest 함수의 구현부(default implementation)를 사용할 수 있습니다.

**Multiple extension**

클래스 상속의 경우 자식 클래스가 하나의 부모 클래스만 상속 받을 수 있습니다.

하지만 클래스 상속과 달리, 프로토콜의 경우에는 하나의 타입이 여러 프로토콜을 채택할 수 있습니다.
다시 말해 다중 프로토콜 채택이 가능합니다.

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

지금부터는 데이터 모델링에 유용하게 쓰이는 프로토콜 상속과 프로토콜 컴포지션에 대해 살펴보겠습니다.

이메일을 보내는 SMTP 프레임워크를 구현하며 프로토콜 상속과 프로토콜 컴포지션을 살펴보고 차이점을 확인해 보겠습니다.

먼저 프로토콜 상속으로 SMTP를 구현하고 이후 프로토콜 컴포지션 방식으로 구현하여 trade-offs를 확인하겠습니다.

SMTP의 기본이 되는 Email 구조체와 Mailer 프로토콜을 아래 코드와 같이 구현했습니다.

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

이제 Mailer 프로토콜의 확장에서 구현한 함수의 구현부(default implementation)에 메일을 보내기 전에 메일의 유효성 검사를 추가하려 합니다.

그러나 Mailer 프로토콜의 따르는 모든 타입에서 메일의 유효성 검사를 요구하진 않습니다.
특정 타입에서만 Mailer 프로토콜을 따르며 추가적으로 메일의 유효성 검사를 요구하는 상황입니다.

먼저 프로토콜 상속으로 위와 같은 조건을 구현해봅시다.

Mailer 프로토콜을 상속 받은 ValidatingMailer 자식 프로토콜을 생성하여 ValidatingMailer 프로토콜에 유효성 검사 기능을 추가합니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/50192d93-3627-425d-999a-83c05000926e)

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
  func send(email: Email) throws  // 부모 프로토콜의 send 함수를 오버라이드 합니다.
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
try? client.send(
  email: Email(
    subject: "Learn Swift",
    body: "Lorem ipsum",
    to: [MailAddress(value: "john@naver.com")],
    from: MailAddress(value: "Stranger@naver.com")
  )
)
```

Mailer 프로토콜을 상속하여 ValidationMailer 자식 프로토콜에 요구사항에 필요한 validate 함수를 추가하고 Mailer 프로토콜의 send 함수를 오버라이드하고 있습니다.

Mailer 프로토콜의 자식 프로토콜인 ValidatingMailer에서 send 함수를 오버라이드 했기 때문에 ValidationMailer 프로토콜을 확장하여 send 함수의 구현부를 제공해야 합니다.

프로토콜 상속에서 자식 프로토콜인 ValidatingMailer 프로토콜이 메일의 유효성 검사와 메일을 보내는 두 가지 성격의 일을 모두 처리해야 합니다.
이처럼 프로토콜 상속을 통해 데이터를 모델링할 때 프로토콜이 의미 단위로 분리되지 않는다는 단점이 존재합니다.

만약 메일 유효성 검사를 필요로 하지 않은 타입에서는 ValidatingMailer가 아닌 Mailer 프로토콜을 채택해야만 합니다.

그렇다면 프로토콜 상속이 아닌 프로토콜 컴포지션을 사용하면 어떨까요?

프로토콜 컴포지션 방식은 Mailer 프로토콜을 상속하는 ValidatingMailer 프로토콜을 생성하지 않고, 메일 유효성을 검사하는 독립된 MailValidator 프로토콜을 생성합니다.

이후 메일 유효성 검사 기능을 필요로 하는 타입에 Mailer 프로토콜과 함께 MailValidator 프로토콜을 다중 채택하여 메일 유효성 검사 기능을 제공합니다.

아래 코드는 독립된 MailValidator 프로토콜을 구현한 코드입니다.

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

이제는 Mailer 프로토콜에서 MailValidator 프로토콜의 존재를 모르고, MailValidator 프로토콜도 Mailer 프로토콜의 존재를 모릅니다.

**Protocol intersection(교차점)**

두 프로토콜을 모두 채택한 타입은 두 프로토콜의 교차점에 위치하는 타입이라고 볼 수 있습니다.

두 프로토콜의 교차점에 위치하는 타입에게 특정 기능을 제공하도록 교차점을 확장할 수 있습니다.

예를 들어 Mailer 프로토콜과 MailValidator 프로토콜을 모두 따르는 SMTPClient 타입이 두 프로토콜의 교차점에 있는 타입이라고 볼 수 있습니다.
Mailer 프로토콜과 MailValidator 프로토콜의 교차점을 확장했다면 확장된 기능을 교차점에 있는 SMTPClient 타입이 사용할 수 있습니다.

두 프로토콜의 교차점을 확장하기 위해서는 **Self 키워드**를 통해 둘 중 하나의 프로토콜을 확장하여 나머지 한 프로토콜을 따르도록 해야 합니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/6ef6ed4a-464f-4770-9111-da3d55f19aa6)

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

두 프로토콜의 교차점을 확장하여 send 함수의 구현부(default implementation)를 제공합니다.
교차점의 확장에서 제공하는 send 함수는 Mailer 프로토콜에서 제공하는 send 함수를 오버라이드한 함수입니다.

위 코드에서 MailValidator 프로토콜을 확장하고 Mailer 프로토콜을 채택했지만, 반대로 Mailer 프로토콜을 확장하고 MailValidator 프로토콜을 채택해도 상관 없습니다.

이제 Mailer 프로토콜과 MailValidator 프로토콜을 모두 채택하는 SMTPClient 타입에서 send 함수의 구현부는 두 프로토콜의 교차점에서 제공하게 됩니다.

프로토콜 교차점에서 기존 함수를 오버라이드하는 경우외에도 아래 코드와 같이 새로운 함수를 추가할 수 있습니다.

```swift
extension MailValidator where Self: Mailer {
  func send(email: Email) throws {
    try validate(email: email)
    // Connect to server
    // Submit email
    print("Email Validated and sent.")
  }

  // 두 프로토콜의 교차점을 확장하여 추가한 새로운 send(email:, at:) 함수
  func send(email: Email, at: Date) throws {
    try validate(email: email)
    // Connect to server
    // Submit email
    print("Email Validated and sent.")
  }
}
```

실제로 Mailer 프로토콜과 MailValidator 프로토콜을 모두 채택하여 두 프로토콜의 교차점에 있는 SMTPClient 타입에서 두 프로토콜의 교차점을 확장해 제공하는 함수들을 사용하는 모습을 살펴봅시다.

```swift
struct SMTPClient: Mailer, MailValidator {}

let client = SMTPClient()
let email = Email(
  subject: "Learn Swift",
  body: "Lorem ipsum",
  to: [MailAddress(value: "john@naver.com")],
  from: MailAddress(value: "Stranger@naver.com")
)

try? client.send(email: email)
try? client.send(email: email, at: Date(timeIntervalSinceNow: 3600))
```

두 프로토콜을 채택한 타입도 두 프로토콜의 교차점에 있지만, 제네릭 타입에서 한 타입이 두 프로토콜로 타입 제약된 경우에도 해당 타입이 두 프로토콜의 교차점에 있다고 볼 수 있습니다.

따라서 두 프로토콜로 타입이 제약된 제네릭 타입에서도 교차점 확장의 함수들을 사용할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
// Mailer, MailValidator 프로토콜로 타입 제약된 제니릭 타입도 두 프로토콜 교차점을 확장한 기능을 사용 가능합니다.
func submitEmail<T>(sender: T, email: Email) where T: Mailer, T: MailValidator {
  try? sender.send(email: email, at: Date(timeIntervalSinceNow: 3600))
}
```

프로토콜 상속과 달리 프로토콜 컴포지션을 사용하면 의미 단위로 코드를 분리할 수 있습니다.

하지만 너무 잘게 분리될 경우 오히려 단점으로 적용됩니다.
또한 프로토콜 상속과 달리 컴포지션 방식의 경우 여러 프로토콜을 다중 채택해야 합니다.

물론 프로토콜 상속은 단일 프로토콜을 채택하도록 하여 경직된 데이터 구조를 형성합니다.

프로토콜 상속과 프로토콜 컴포지션 방식의 균형을 맞춰 추상화를 이루도록 노력해야 합니다.

## Overriding priorities

**Overriding a default implementation**

앞에서 프로토콜 확장으로 프로토콜에 선언된 함수의 구현부를 제공하지만, 해당 프로토콜을 채택한 타입에서 구현부를 제공하는 함수를 오버라이드 할 경우를 잠시 살펴봤습니다.

지금부터 더 자세히 알아봅시다.

grow 함수를 가진 Tree 프로토콜을 구현하여 프로토콜 상속 과정을 살펴봅시다.
Tree 프로토콜을 확장하여 grow 함수를 구현하고 Oak 구조체에서 Tree 프로토콜을 채택하려 합니다.
이때 Oak 구조체에서 grow 함수를 오버라이드하는 상황입니다.

만약 Oak 구조체에서 grow 함수를 오버라이드 하지 않는다면 Oak 구조체가 채택한 Tree 프로토콜의 grow 함수를 호출합니다.
하지만 Oak 구조체에서 grow 함수를 오버라이드 한다면 Oak 구조체에서 호출되는 grow 함수는 Oak 구조체에서 오버라이드한 grow 함수가 호출됩니다.
(If a type implements the same method as the one on a protocol extension, Swift ignores the protocol extension's method.)

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/847f79b2-cf72-4f14-b17a-4959b7f37417)

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

Element 연관 값을 Hashable 프로토콜로 타입 제약한 unique 함수를 살펴보면 set 자체로 유일한 값이 저장되지만 추가적으로 [Element] 배열의 uniqueValues 변수를 만들어 set과 동일한 조건으로 조작하고 있습니다.
물론 unique 함수의 리턴 타입이 [Element]이기 때문에 uniqueValues 변수가 필요했지만, 아래와 같이 Set을 확장하여 Set을 배열로 리턴할 수 있습니다.

```swift
extension Set {
  func unique() -> [Element] {
    return Array(self)
  }
}
```

Set을 배열로 변환하는 unique 함수 덕분에 아래와 같이 uniqueValues 배열 없이 Collection 확장의 unique 함수를 구현할 수 있습니다.

```swift
extension Collection where Element: Hashable {
  func unique() -> [Element] {
    var set = Set<Element>()
    for element in self {
      if !set.contains(element) {
        set.insert(element)
      }
    }
    return set.unique()
  }
}
```

The point is, finding the balance between extending the lowest common denominator without weakening the API of concrete type is a bit of an art.

스위프트는 결국 가장 구체적인 구현을 선택합니다. 위에서 이야기했던 가장 특수화된 구현을 선택하다는 것과 동일한 의미입니다.

## Extending with concrete constraints

프로토콜의 연관 값(associated types)에 타입 제약을 가할 때 프로토콜이 아닌 특정 타입(concrete type)으로 연관 값의 타입을 제약할 수 있습니다.

아래 코드로 살펴봅시다.

```swift
// Article 구조체가 특정 타입(concrete type)입니다.
struct Article: Hashable {
  let viewCount: Int
}

// Not like this:
extension Collection where Element: Article {  ... }

// But like this:
extensioin Collection where Element == Article {
  var totalViewCount: Int {
    var count = 0
    for article in self {
      count += article.viewCount
    }
    return count
  }
}
```

특정 타입으로 프로토콜의 연관 값에 타입 제약을 할 때 : 가 아닌 == 연산자를 사용해 제약을 줄 수 있습니다.

위의 Collection 확장을 아래 코드와 같이 사용할 수 있습니다.

```swift
let articleOne = Article(viewCount: 30)
let articleTwo = Article(viewCount: 200)

// Getting the total count on an Array.
let articlesArray = [articleOne, articleTwo]
articlesArray.totalViewCount  // 230

// Getting the total count on a Set
let articlesSet: Set<Article> = [articleOne, articleTwo]
articlesSet.totalViewCount  // 230
```

기능 추가를 위해 확장을 사용할 때 얼마나 저차원 요소를 확장 할지 결정하는 것은 까다롭습니다.

실제로 배열을 확장하면 80%의 케이스에 충족합니다.
따라서 저차원인 Collection을 확장할 필요까지 없을 수 있습니다.

물론 저차원인 Collection을 확장할 경우 더 많은 타입에 확장한 기능을 사용할 수 있게 됩니다.
Collection 보다 더 저차원인 Sequence를 확장하면 더 많은 타입에 확장한 기능을 사용할 수 있습니다.

## Extending Sequence

Sequence 프로토콜을 확장하면 Sequence 프로토콜을 따르는 Set, Array, Dictionary 등의 타입에 확장된 기능을 사용할 수 있습니다.

실제로 프로젝트에서 Sequence 프로토콜을 확장하는 것은 프로젝트에 큰 도움이 됩니다.

**Looking under the hood of filter**

Sequence 프로토콜을 확장하기 전에 filter 함수를 먼저 살펴봅시다.

filter 함수는 함수를 입력 받습니다.
이때 함수는 클로저가 됩니다.

아래 코드와 같이 filter 함수가 사용됩니다.

```swift
let moreThanOne = [1, 2, 3].filter { (int: Int) in
  int > 1
}
print(moreThanOne)  // [2, 3]
```

이제 filter 함수의 구현부 내부를 살펴봅시다.

```swift
public func filter(
  _ isIncluded: (Element) throws -> Bool
) rethrows -> [Element] {
  var result = ContiguousArray<Element>()

  var iterator = self.makeIterator()

  while let element = iterator.next() {
    if try isIncluded(element) {
      result.append(element)
    }
  }
  // ContiguousArray 타입을 다시 Array 타입으로 변환해 리턴합니다.
  return Array(result)
} 
```

filter 구현부를 보면 rethrows 키워드가 붙습니다.
rethrows 키워드를 사용하여 자신의 매개변수로 전달받은 함수가 오류를 던진다는 것을 나타낼 수 있습니다.
다시 말해 입력받은 클로저가 오류를 던질 때 해당 오류를 filter 함수의 호출부로 다시 던집니다.

만약 rethrows를 사용하지 않은 경우, filter 함수를 호출하는 부분에서 오류를 처리하는게 아닌 filter 함수 내부에서 처리해야 합니다.

filter 함수에 rethrows가 붙어서 try 키워드와 함께 catch를 사용하지 않아도 됩니다.
try 키워드에서 오류가 발생할 때 filter 함수의 호출부로 에러가 던져집니다.

filter 구현부에 등장하는 **ContiguousArray** 타입은 배열 요소로 클래스나 Objectivew-C 프로토콜을 가졌을 때 성능적 이점이 있습니다.

filter 함수와 같은 저차원 함수는 성능이 중요시 여겨집니다. 성능을 향상 시키기위해 ContiguousArray를 사용합니다.
하지만 그외의 요소를 가진다면 일반 Array 타입과 동일한 성능을 보입니다.

**Creating the take(while:) method**

지금부터 Sequence 프로토콜을 확장해 봅시다.

Sequence 프로토콜이 제공하는 drop(while:) 함수와 정반대의 기능을 하는 take(while:) 함수를 Sequence 프로토콜 확장에 추가하려 합니다.

drop(while:) 함수는 while로 입력 받는 클로저 속 조건을 만족할 때 요소들을 순회하며 무시하다가(drop) 조건에 만족하지 않을 때 이후의 요소들을 리턴합니다.

아래 공식 문서를 통해 더 자세히 알아봅시다.

https://developer.apple.com/documentation/swift/sequence/drop(while:)

아래 코드와 같이 drop(while:) 함수를 사용할 수 있습니다.

```swift
let numbers = [3, 7, 4, -2, 9, -6, 10, 1]
let startingWithNegative = numbers.drop(while: { $0 > 0 })
// startingWithNegative == [-2, 9, -6, 10, 1]
```

그렇다면 drop(while) 함수와 정반대의 기능을 하는 take(while:) 함수는 while로 입력 받은 클로저의 조건에 만족하는 요소들을 리턴하다가 조건에 만족하지 않을 때 순회를 종료합니다.

아래와 같이 take(while:) 함수가 동작합니다.

```swift
let lines =
  """
  We start with text.
  OKOK let's start.

  This is ignored because it came after empty space
  and more text
  """.components(seperatedBy: "\n")

let firstParts = lines.take(while: { (line) -> Bool in
  !line.isEmpty
})

print(firstParts)  //["We start with text.", "OKOK let's start."]
```

take(while:) 함수를 구현할 때 앞에서 살펴봤던 filter 함수의 구현부를 표방해 봅시다.
아래 코드로 take(while:) 함수를 Sequence 프로토콜 확장에 추가한 코드입니다.

```swift
extension Seequence {
  public func take(
    while predicate: (Element) throws -> Bool
  ) rethrows -> [Element] {
    
  var iterator = makeIterator()

  var result = ContiguousArray<Element>()

  while let element = iterator.next() {
    if try predicate(element) {
      result.append(element)
    } else {
      break
    }
  }
  return Array(result)
}
```

위와 같이 take(while:) 함수를 Sequence 프로토콜 확장에 추가해서 프로젝트에 유용하게 사용할 수 있습니다.

**Creating the Inspect method**

위에서는 Sequence 프로토콜을 확장하여 take(while:) 함수를 추가했다면 이번에는 inspect 함수를 추가해 봅시다.
inspect 함수는 파이프라인 구조로 데이터를 다룰 때 디버깅 용도로 사용되는 함수입니다.

filter나 forEach는 데이터를 조작하여 변형된 데이터를 하위 파이프라인으로 전달하지만, inspect 함수에서는 데이터를 조작하지만 하위 파이프라인으로는 변형되지 않은 상태를 전달합니다.

따라서 보통은 파이프라인 중간에 데이터를 디버깅하는 용도로 주로 사용합니다.

```swift
extension Sequence {
  pubilc func inspect(
    _ body: (Element) throws -> Void
  ) rethrows -> Self {
    for element in self {
      try body(element)
    }
    // inspect 함수의 입력으로 들어오는 클로저로 데이터가 변형되지만 실제로 리턴하는 데이터는 변형되지 않은 상태로 리턴합니다.
    return self
  }
}

["C", "B", "A", "D"]
  .sorted()
  .inspect { (string) in
    print("Inspecting: \(string)")
  }.filter { (string) -> Bool in
    string < "C"
  }.forEach {
    print("Result: \($0)")
  }

// Output:
// Inspecting: A
// Inspecting: B
// Inspecting: C
// Inspecting: D
// Result: A
// Result: B
```

저차원의 Sequence 프로토콜을 확장하여 배열을 비롯해 여러 타입에 확장된 기능을 추가할 수 있습니다.

프로토콜과 확장은 재사용성이 높은 코드를 만들고 의미 단위로 코드를 분리하도록 합니다.
하지만 특정 타입(concrete type)이 추상화를 구현하는 부분에서 더 어울릴 경우도 있습니다.
프로토콜 확장과 특정 타입의 사용을 균형있게 사용해야 합니다.

## Summary
- Protocols can deliver a default implementation via protocol extensions.
- With extensions, you can think of modeling data horizontally, whereas with subclassing, you're modeling data in a more rigid vertical way.
- You can override a default implementation by delivering an implementation on a concrete type.
- Protocol extensions cannot override a concrete type.
- Via protocol inheritance, you can override a protocol's default implementation.
- Swift always picks the most concrete implementation.
- You can create a protocol extension that only unlocks when a type implements two protocols, called a protocol intersection.
- A protocol intersection is more flexible than protocol inheritance, but it's also more abstract to understand.
- When mixing subclasses with protocol extensions, extending a protocol and constraining it to a class is a good heuristic (as opposed to extending a class to adhere to a protocol). This way, an implementer can pick and choose a protocol implementation.
- For associated type, such as Element on the Collection protocol, Swift picks the most specialized abstraction, such as Hashable over Equatable elements.
- Extending a low-level protocol - such as Sequence - means you offer new methods to many types at once.
- Swift uses a special ConiguousArray when extending Sequence for extra performance.
