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
옵셔널은 값을 가졌거나 가지지 않은 값을 가질 가능성이 있느 상자와 비슷합니다.

옵셔널을 통해 값이 없을 때 발생되는 크래시를 막을 수 있습니다. 
옵셔널 안에 값이 없다면 이를 컴파일 타임에 알 수 있습니다.
옵셔널은 내부 값을 감싸고 있어 언래핑을 통해야만 내부 값에 접근할 수 있습니다.

## Clean optional unwrapping
아래 코드를 보면 옵셔널 프로퍼티를 ?로 선언하고 있습니다.
다른 프로퍼티와 달리 옵셔널 프로퍼티는 초기화 의무를 가지지 않습니다.

```swift
struct Customer {
  let id: String
  let email: String
  let balance: Int
  let firstName: String?
  let lastName: String?
}
```

옵셔널은 어떻게 구현되어 있을까요?
enum으로 구현되어 있습니다.
"값이 있거나 없거나" 이기에 or 속성을 띄고 있으니 enum으로 구현된 사실이 이해됩니다. 

옵셔널에는 내장 데이터 타입은 물론 커스텀 타임도 들어갈 수 있습니다. 따라서 generic type인 사실도 이해됩니다.

```swift
public enum Optional<Wrapped> {
  case none
  case some(Wrapped)
}
```

스위프트에서 문법적 편리함을 위해 제공하는 기능들을 syntactic sugar라고 부릅니다.
변수의 옵셔널 선언을 ?로 할 수 있는 이유도 syntatic sugar의 도움에 있습니다.
원래라면 위와 같은 enum은 Optional<Wrapped>로 자료형이 선언되어야 하는데 말입니다.

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
옵셔널 언래핑하면 가장 먼저 떠오르는건 if let입니다.
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
아래 코드를 확입해봅시다. syntactic sugar의 유무와 관계 없이 동일한 기능을 하는 코드입니다.

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

아래 코드를 확인해봅시다.

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

if let을 통한 언래핑은 언래핑 내의 값을 원할 때 적절합니다. 만약 값이 필요 없다면 와일드카드(_)를 사용하여 값을 무시할 수 있습니다.

```swift
if
  let _ = customer.firstName,
  let _ = customer.lastName {
    print("The customer entered his full name")
}
```

## Variable shadowing
"같은 이름으로 언래핑합시다!"

언래핑 이후 결과를 저장하는 변수명을 옵셔널 변수명과 같도록 만듭시다.
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
if let은 옵셔널이 nil일 경우 함수를 종료시키지 않고 계속해서 함수를 진행시킵니다.
따라서 if let은 옵셔널이 nil이 필요한 경우 사용합니다.

만약 옵셔널 언래핑에 nil이 필요 없을 경우 guard let을 사용합시다.
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
빈 문자열을 리턴하는게 틀렸다는건 아닙니다. 

하지만 일부 경우에는 빈 문자열을 리턴하는것 보다 optional string을 리턴하는게 유용할 때가 있습니다.
예를 들어 위의 Customer 구조체가 가진 displayName에서 언래핑 실패 이후 빈 문자열을 던질 경우, displayName을 호출하는 부분에서 코드 작성자가 직접 displayName.isEmpty를 체크해야합니다.
displayName의 호출부에서는 특정 이름을 기대하지 빈 문자열을 기대하고 있지 않습니다.
심지어 isEmpty를 체크하지 않더라도 컴파일 타임에 체크하지 않았다는 사실을 알 수 없습니다.

이럴 때 빈 문자열이 아닌 optional string을 리턴하면 displayName 호출부에서는 옵셔널을 받을 가능성이 생기기며 리턴 받는 optional string을 언래핑해야할 의무가 생깁니다.
따라서 컴파일 타임에 displayName이 값을 가지지 않음을 알 수 있습니다.

아래 코드는 옵셔널 언래핑 실패시 빈 문자열을 던지던 displayName을 optional string을 던지도록 수정한 코드입니다.
옵셔널을 리턴하며 호출부에서 옵셔널을 언래핑할 책임을 갖게 됩니다.
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
만약 firstName과 lastName 중 하나라도 있다면 displayName을 형성하도록 만들려면 어떻게 해야할까요?
이런 경우 옵셔널을 세분화해서 다뤄야합니다.

?와 switch 그리고 tuple을 사용하여 옵셔널을 세분화하여 언래핑할 수 있습니다. (패턴 매칭과 비슷한 느낌입니다.)

아래 코드로 확인해봅시다.

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

다만, tuple에 너무 많은 옵셔널을 넣는 행위는 경계해야합니다.
3개 미만의 옵셔널이 tuple에 적합합니다.

## Falling back when an optional is nil
fallback value(??)는 옵셔널 언래핑 종료 중 하나입니다. 
옵셔널이 nil일 경우 default 값으로 대신하며 언래핑합니다.

아래 코드로 확인해봅시다.
```swift
let title: String = customer.displayer ?? "customer"
createConfirmationMessage(name: title, product: "Economy size party tub")
```

## Simplifying optional enums
앞서 말했듯이 옵셔널은 enum으로 구현되어 있습니다.
그렇다면 optional enum은 enum안에 enum이 구현된 것입니다.

optional enum을 언래핑할 때 if let을 대신하여 ?을 사용해 코드를 개선할 수 있습니다.
아래 코드를 보고 이해해봅시다.

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
아래 방법으로 추가적인 if let없이 enum의 패턴 매칭과 동시에 옵셔널 언래핑을 할 수 있습니다.

```swift
switch customer.membership {
case .gold?: print("its gold")
case .silver?: print("its silver")
case nil: print("none")
}
```

여기서 문제입니다.

만약 아래 코드와 같이 PasteBoardContents enum과 PasteBoardEvent enum이 있을 때 describeAction 함수를 완성해봅시다. 

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
아래 코드를 통해 확인해봅시다.

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
옵셔널 체이닝으로 옵셔널 값이 갖는 옵셔널 값을 언래핑할 때 간결히 할 수 있습니다.































