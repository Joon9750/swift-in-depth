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
print(request)  // 
```


















