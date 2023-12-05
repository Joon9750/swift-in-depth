# Making optionals second nature

## This chapter covers
- Best practice related to optionals
- Handling multiple optionals with guards
- Properly dealing with optional strings versus empty strings
- Jugglings various optionals at once
- Falling back to default values using the nil-coalescing operator
- Simplifying optional enums
- Dealing with optional Booleans in multiple ways
- Digging deep into values with optional chaining
- Force unwrapping guidelines
- Taming implicity unwrapped optionals

## The purpose of optionals
옵셔널은 값을 가졌거나 가지지 않았을 가능성이 있는 열지 않은 상자와 비슷합니다.

옵셔널을 통해 값이 없을 때 발생하는 크래시를 막을 수 있고 옵셔널 안에 값이 없다면 이를 컴파일 타임에 알 수 있습니다.
옵셔널은 내부 값을 감싸고 있어 언래핑을 통해야만 옵셔널을 벗겨야 내부 값에 접근할 수 있습니다.

## Clean optional unwrapping
아래 코드를 보면 옵셔널 프로퍼티를 ?로 선언하고 있습니다.
다른 프로퍼티와 달리 옵셔널 프로퍼티는 초기화 의무를 지지 않습니다.

```swift
struct Customer {
  let id: String
  let email: String
  let balance: Int
  let firstName: String?
  let lastName: String?
}
```

옵셔널은 실제로 어떻게 구현되어 있을까요?

enum으로 구현되어 있습니다.

옵셔널의 성격이 "값이 있거나 없거나"이기에 "or"에 어울려 enum으로 구현되었다고 이해하면 쉽습니다.
옵셔널에는 내장 데이터 타입은 물론 커스텀 타임도 들어갈 수 있습니다. 따라서 옵셔널이 generic type이라는 사실도 알 수 있습니다.

```swift
public enum Optional<Wrapped> {
  case none
  case some(Wrapped)
}
```

스위프트에서 문법적 편리함을 위해 제공하는 기능들을 syntactic sugar라고 부릅니다.
변수의 옵셔널 선언을 ?로 할 수 있는 이유도 syntactic sugar의 도움에 있습니다.
원래라면 위와 같은 enum은 Optional<Wrapped>로 자료형이 선언되어야 합니다. 
syntactic sugar 덕분에 ?선언만으로 옵셔널을 선언할 수 있습니다.

관용적으로 Optional<Wrapped>보다 ?를 사용한 방식이 더 어울립니다.

```swift
// with syntatic sugar
struct Customer {
  let id: String
  let email: String
  let balance: Int
  let firstName: String?
  let lastName: String?
}

// without syntatic sugar
struct Customer {
  let id: String
  let email: String
  let balance: Int
  let firstName: Optional<String>
  let lastName: Optional<String>
}
```

옵셔널의 내부 값에 접근하려면 옵셔널을 언래핑해야 합니다.
옵셔널 언래핑하면 가장 먼저 떠오르는 건 if let입니다.
아래 코드는 if let을 사용한 옵셔널 언래핑 예시입니다.

```swift
let customer = Customer(id: "30", email: "mayeloe@naver.com", firstName: "Jake", lastName: "Freemason", balance: 300)

print(customer.firstName)  // "Optional("Jake")
if let firstName = customer.firstName {
  print(firstName)  // "Jake"
}
```

enum은 case에 따른 패턴 매칭이 가능합니다.
따라서 Optional도 enum으로 구현되었기에 패턴 매칭이 가능합니다.
아래 코드와 같이 Optional을 언래핑할 때 패턴 매칭을 사용하는 것도 유용합니다.

Optional의 패턴 매칭에도 syntactic sugar가 도움을 줍니다.
아래 코드를 확인해 봅시다. syntactic sugar의 유무와 관계없이 동일한 기능을 하는 코드입니다.

```swift
// Optional pattern match without syntactic sugar 
switch customer.firstName {
  case .some(let name): print("First name is \(name)")
  case .none: print("Customer didn't enter a first name")
}

// Optional pattern match with syntactic sugar
switch customer.firstName {
  case let name?: print("First name is \(name)")
  case nil: print("Customer didn't enter a first name")
}
```

위에서 보았듯이 if let을 사용해 언래핑이 가능합니다.
심지어 언래핑을 중복해 사용하기도 합니다.
언래핑을 중복해 사용해서 들여쓰기 단계를 줄일 수 있습니다. 들여쓰기 단계가 줄어들면 가독성이 높아집니다.

아래 코드를 확인해 봅시다.

```swift
if let firstName = customer.firstName, let lastName = customer.lastName {
  print("Customer's full name is \(firstName) \(lastName)")
}
```

언래핑 if let과 함께 Bool 조건도 사용할 수 있습니다.

```swift
if let firstName = customer.firstName, customer.balance > 0 {
  let welcomeMessage = "Dear \(firstName), you have money on your account, want to spend it on mayonnaise?"
}
```

언래핑 if let과 함께 패턴 매칭도 가능합니다.

```swift
if let firstName = customer.firstName, 4500..<5000 ~= customer.balance {
  let notification = "Dear \(firstName), you are getting close to afford out $50 tub!")
}
```

위와 같이 multipule unwrapped, unwrapped with Bool, unwrapped with pattern matching을 통한 코드 정리는 바람직합니다.

if let을 통한 언래핑은 언래핑 내부 값을 원할 때 적절합니다. 만약 내부 값이 필요 없다면 와일드카드(_)를 사용하여 값을 무시할 수 있습니다.

```swift
if
  let _ = customer.firstName,
  let _ = customer.lastName {
    print("The customer entered his full name")
}
```

## Variable shadowing
"같은 이름으로 언래핑합시다!"

언래핑 이후 결과를 저장하는 변수명을 옵셔널 변수명과 같게 만듭시다.
새로운 변수명에 언래핑의 결과를 저장하는 방식은 권장하지 않습니다.
오히려 새로운 변수명은 혼란을 일으킵니다.

```swift
extension Customer: Customer {
  var description: String {
    var customDescription: String = "\(id), \(email)"

    // 같은 이름으로 언래핑합시다!
    if let firstName = firstName {
      customDescription += ", \(firstName)"
    }
    if let lastName = lastName {
      customDescription += "\(lastName)"
    }

    return customDescription
  }
}
```

## When optionals are prohibited
지금까지는 if let을 사용해 옵셔널을 언래핑했습니다.
if let은 옵셔널이 nil일 경우 함수를 종료시키지 않고 계속해서 함수를 진행합니다.
따라서 if let은 옵셔널이 nil이 필요한 경우 사용합니다.

만약 옵셔널 언래핑에 nil이 필요 없으면 guard let이 더 적합합니다.
guard let에서 옵셔널이 nil이라면 해당 함수를 종료시킵니다.

```swift
struct Customer {
  let id: String
  let email: String
  let firstName: String?
  let lastName: String?

  var displayName: String {
    guard let firstName = firstName, let lastName = lastName else {
      return ""
    }
    return "\(firstName) \(lastName)"
  }
}
```

guard let을 통해 옵셔널을 언래핑하면 guard let 구문 아래로는 옵셔널 타입이 내려갈 일이 없어진다.

## Returing optional strings
optional string의 경우 언래핑되지 않았을 때 빈 문자열("")을 리턴하는건 흔하게 볼 수 있습니다.
빈 문자열을 리턴하는게 틀렸다는 건 아닙니다. 

하지만 일부 경우에는 빈 문자열을 리턴하는것 보다 optional string을 리턴하는게 유용할 때가 있습니다.
예를 들어 위의 Customer 구조체가 가진 displayName에서 언래핑 실패 이후 빈 문자열을 던질 경우, displayName을 호출하는 부분에서 코드 작성자가 직접 displayName.isEmpty를 체크해야 합니다.
displayName의 호출 부에서는 특정 이름을 기대하지 빈 문자열을 기대하고 있지 않습니다.
심지어 isEmpty를 체크하지 않더라도 컴파일 타임에 체크하지 않았다는 사실을 알 수 없습니다.

이럴 때 빈 문자열이 아닌 optional string을 리턴하면 displayName 호출부에서는 옵셔널을 받을 가능성이 생기기며 리턴 받는 optional string을 언래핑해야할 의무가 생깁니다.
따라서 컴파일 타임에 displayName이 값을 가지지 않음을 알 수 있습니다.

아래 코드는 옵셔널 언래핑 실패 시 빈 문자열을 던지던 displayName을 optional string을 던지도록 수정한 코드입니다.
옵셔널을 리턴하며 호출 부에서 옵셔널을 언래핑할 책임을 갖게 됩니다.
이는 컴파일러의 도움을 받기 때문에 더 안전한 코드가 됩니다.

```swift
struct Customer {
  // 생략

  var displayName: String? {
    guard let firstName = firstName, let lastName = lastName else {
      return nil  // nil을 리턴하며 optional string을 던지게 됩니다.
    }
    return "\(firstName) \(lastName)"
  }
}

if let displayName = customer.displayName {
  createConfirmationMessage(name: displayName, product: "Economy size party tub")
} else {
  createConfirmationMessage(name: "customer", product: "Economy size party tub")
}
```

## Granular control over optionals
위의 예에서 displayName은 firstName과 lastName을 모두 요구하고 있습니다.
만약 firstName과 lastName 중 하나라도 있다면 displayName을 형성하도록 만들려면 어떻게 해야 할까요?
이런 경우 옵셔널을 세분화해서 다뤄야 합니다.

?와 switch 그리고 tuple을 사용하여 옵셔널을 세분화하여 언래핑할 수 있습니다. (패턴 매칭과 비슷한 느낌입니다.)

아래 코드로 확인해 봅시다.

```swift
struct Customer {
  // 생략
  let firstName: String?
  let lastName: String?
  
  var displayName: String? {
    // switch + optional in tuple + ?
    switch (firstName, lastName) {
    case let (first?, last?): return first + " " + last  // return에 쓰이는 first와 last는 옵셔널 언래핑된 상태입니다.
    case let (first?, nil): return first  // nil 값을 노골적으로 매칭할 수 있다.
    case let (nil,last?): return last
    default: return nil 
  }
}
```

위와 같이 손님이 풀네임을 입력하지 않을 경우, 일부 이름으로 풀네임을 대신할 수 있습니다.

다만, tuple에 너무 많은 옵셔널을 넣는 행위는 경계해야 합니다.
3개 미만의 옵셔널이 tuple에 적합합니다.

## Falling back when an optional is nil
fallback value(??)는 옵셔널 언래핑 종료 중 하나입니다. 
옵셔널이 nil일 경우 default 값으로 대신하며 언래핑합니다.

아래 코드로 확인해 봅시다.
```swift
let title: String = customer.displayer ?? "customer"
createConfirmationMessage(name: title, product: "Economy size party tub")
```

## Simplifying optional enums
앞서 말했듯이 옵셔널은 enum으로 구현되어 있습니다.
그렇다면 optional enum은 enum안에 enum이 구현된 것입니다.

optional enum을 언래핑할 때 if let을 대신하여 ?을 사용해 코드를 개선할 수 있습니다.
아래 코드를 보고 이해해 봅시다.

```swift
enum Membership {
  case gold
  case silver
}

struct Customer {
  // 생략
  let merbership: Membership?
}
```

위와 같이 Memebership enum을 Customer 구조체에서 옵셔널로 선언했습니다.
이때 if let을 사용하여 언래핑한다면 아래와 같습니다.

```swift
if let membership = customer.membership {
  switch membership {
  case .gold: print("its gold")
  case .silver: print("its silver")
  }
} else {
  print("none")
}
```

if let을 대신해서 ?를 사용한다면 옵셔널 언래핑과 옵셔널 내부 값 접근을 동시에 할 수 있습니다.
이는 코드를 더 짧게 만듭니다.
아래 방법으로 추가적인 if let 없이 enum의 패턴 매칭과 동시에 옵셔널 언래핑을 할 수 있습니다.

```swift
switch customer.membership {
case .gold?: print("its gold")
case .silver?: print("its silver")
case nil: print("none")
}
```

여기서 문제입니다.

만약 아래 코드와 같이 PasteBoardContents enum과 PasteBoardEvent enum이 있을 때 describeAction 함수를 완성해 봅시다. 

```swift
enum PasteBoardContents {
  case url(url:String)
  case emailAddress(emailAddress: String)
  case other(content: String)
}

enum PasteBoardEvent {
  case added
  case erased
  case pasted
}

func describeAction(event: PasteBoardEvent?, contents: PastBoardEvent?) -> String {}
```

describeAction 함수로 옵셔널이 2개 들어오기 때문에 두 옵셔널을 튜플에 넣고 ?와 함께 패턴 매칭하여 옵셔널을 언래핑하는 방식이 적절합니다.
아래 코드를 통해 확인해 봅시다.

```swift
func describeAction(event: PasteBoardEvent?, contents: PastBoardEvent?) -> String {
  switch (event, contents) {
  case let (.added?, url(url)?): return "User added a url to pasteboard: \(url)"  // url 값이 return에 쓰이기 때문에 let이 붙었습니다.
  case (.added?, _): return "User added something to pasteboard"
  case (.erased?, .emailAddress?): return "User erased an email address from the pasteboard"
  default: return "The pasteboard is updated"
}
```

## Chaining optionals
옵셔널 체이닝은 옵셔널이 본인의 프로퍼티로 또 다른 옵셔널을 가질 때, 옵셔널의 옵셔널 프로퍼티에 접근하기 위해 사용됩니다.
?를 사용해 옵셔널 체이닝을 표현합니다.
옵셔널 체이닝으로 옵셔널의 옵셔널 프로퍼티에 접근할 수 있지만 해당한 프로퍼티 또한 옵셔널이기 때문에 추가적인 옵셔널 언래핑이 있어야 순수 값을 추출 가능합니다.

아래 코드를 통해 살펴봅시다.

```swift
struct Product {
  let id: String
  let name: String
  let image: UIImage?
}

struct Customer {
  // 생략
  let favoriteProduct: Product?
}

let imageView = UIImageView()
// optional value입니다. 옵셔널 체이닝을 통해 favoriteProduct의 image 옵셔널 값에 접근하고 있습니다.
imageView.image = customer.favoriteProduct?.image

if let image = customer.favoriteProduct?.image {
  imageView.image = image  // if let으로 더 이상 optional value가 아닙니다.
} else {
  imageView.image = UIImage(named: "missing_image")
}

imageView.image = customer.favoriteProduct?.image ?? UIImage(named: "missing_image")
```

위 코드처럼 옵셔널 체이닝으로 접근한 옵셔널 값을 if let 또는 ??로 옵셔널 언래핑할 수 있습니다.
옵셔널 체이닝으로 옵셔널 값이 갖는 옵셔널에 접근할 때 더욱 간결히 접근할 수 있습니다.

## Constraining optional Booleans
스위프트에서 optional Bool의 상태는 true / false / nil이 될 수 있습니다.
상황의 맥락에 따라 optional Bool 타입 값이 nil일 때 nil을 true / false / nil로 취급할 수 있습니다.

가장 먼저 optional Bool 타입이 두 가지 상태(true/false)를 가지는 경우를 살펴봅시다.
optional Bool 타입이 nil일 때 해당 값을 true / false로 취급할 경우 Bool 타입은 두 가지 상태를 갖게 됩니다.
이때는 ??을 통해 default 값을 설정하는 방식으로 구현할 수 있습니다.
위에서 보았듯이 ??는 언래핑과 동시에 해당 값이 nil일 경우 default 값을 따라가도록 합니다.

아래 코드를 살펴봅시다.

```swift
let preferences = ["autoLogin": true, "faceIdEnabled": true]

let isFaceIdEnabled = preferences["faceIdEnabled"]
print(isFaceIdEnabled)  // Optional(true) 

let isFaceIdEnabled = preferences["faceIdEnabled"] ?? false
print(isFaceIdEnabled)  // true
```

물론 optional Bool 타입이 두 가지 상태(true/false)를 가질 경우 무조건 default를 false로 설정하는 건 아닙니다.
상황에 따라 default를 true 또는 false로 설정해야 합니다.

지금부터 optional Bool 타입이 세 가지 상태(true/false/nil)를 가지는 경우를 살펴봅시다.

optional Bool 타입을 세 가지 상태로 사용할 때 더 여러 상황에 어울립니다.
optional Bool 타입을 세 가지 상태로 사용할 때 enum과 함께 사용합시다.
optional Bool의 내부 값에 따라 enum의 case에 매칭하는 방법으로 세 가지 상태를 나타낼 수 있습니다.

다시 말해, optional Bool의 내부 값을 custom enum으로 변환해 사용합시다. 이때 RawRepresentable Protocol까지 사용하면 더욱 swift 다워집니다.

RawRepresentable Protocol은 Swift에서 enum은 하나 이상의 associated value를 가질 수 있는데 이 값의 rawValue를 정의한 프로토콜입니다. (rawValue를 지원합니다.)
RawRepresentable Protocol을 채택하여 타입을 rawValue로 변환하고 그 반대의 변환도 지원합니다.
특정 rawValue를 enum의 case로 매칭시켜 서로에게 대응할 수 있습니다. (Bool(rawValue) <-> Enum)

아래 코드를 보고 이해해 봅시다.

```swift
enum UserPreference: RawRepresentable {
  case enabled  // true가 매칭되는 case
  case disabled  // false가 매칭되는 case
  case notSet  // nil이 매칭되는 case

  init(rawValue: Bool?) {
    // rawValue를 switch문과 함께 쓸 수 있는 이유는 rawValue 타임을 옵셔널로 받기 때문입니다.
    // 옵셔널은 enum이기 때문에 switch문과 함께 쓸 수 있습니다.
    switch rawValue {
    case true?: self = .enabled
    case false?: self = .disabled
    default: self = .notSet
    }
  }

  // RawRepresentable protocol을 따르면 rawValue를 구현해야 합니다.
  // rawValue 프로퍼티로 enum case에 대응하는 rawValue를 리턴해야 합니다.
  var rawValue: Bool? {
    switch self {
    case .enabled: return true
    case .disabled: return false
    case .notSet: return nil
    }
  }
}
```

위와 같이 optional Bool을 enum과 함께 사용해 세 가지 상태를 표현할 때 컴파일 타임에 이점도 있고 더 여러 상황에 적합하지만, 
결국 새로운 타입(enum)을 만드는 것이기 때문에 코드를 더럽힐 가능성이 있으니 주의해야 합니다.

## Force unwrapping guidelines
옵셔널을 강제 언래핑 했을 때 옵셔널이 nil이라면 충돌이 발생합니다.
아래 코드와 같이 !를 사용해 옵셔널을 강제 언래핑합니다.

```swift
let forceUnwrappedUrl = URL(string: "https://www.the.com")!
```

강제 언래핑은 실패 시 앱이 충돌하기 때문에 사용을 최대한 피합시다. (그냥 쓰지 맙시다.)

강제 언래핑으로 충돌을 발생시키기보다 충돌이 발생할 수 있는 경우라면, 충돌 메뉴얼을 만들어 더 디버깅에 필요한 정보를 제공하는 방식이 바람직합니다.
fatalerror 함수를 사용해서 충돌을 메뉴얼화해 봅시다.

```swift
guard let url = URL(string: path) else {
  fatalerror("Could not create url on \(#line) in \(#function)")
}
```

## Taming implicity unwrapped optionals
Implicity unwrapped optionals(IUO)은 옵셔널이지만 언래핑하지 않고도 내부 값에 접근할 수 있는 옵셔널입니다.
옵셔널 내부 값에 접근할 때마다 옵셔널 언래핑을 하면 코드가 길어지고 가독성이 떨어지기 때문에 프로그램 특정 시점 이후로 옵셔널에 값이 항상 있다면 IUO를 사용할 수 있습니다.

하지만... 프로그램 특정 시점 이후로 옵셔널에 값이 항상 있다고 어떻게 장담할지 모르겠습니다.

일단 사용 방법은 아래와 같습니다.

```swift
let lastName: String! = "Smith"
```

## Summary 
- Optionals are enums with syntactic sugar sprinkled over them
- You can pattern match on optionals
- Patter match on multiple optionals at once by putting them inside a tuple
- You can use nil-coalescing(??) to fall back to default values
- Use optional chaining to dig deep into optional values
- You can use nil-coalescing(??) to transform an optional Boolean into a regular Boolean
- You can transform an optional Boolean into an enum for three explicit states
- Return optional strings instead of empty strings when a value is expected
- Use force unwrapping only if your program can't recover from a nil value
- Use force unwrapping when you want to delay error handling, such as when prototyping
- It's safer to force unwrap optionals if you know better than the compiler
- Use implicity unwrapped optionals for properties that are instantiated right after initialization
