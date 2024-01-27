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
