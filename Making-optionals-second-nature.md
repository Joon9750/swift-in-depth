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
"같은 이름으로 언래핑하자!"

언래핑 이후 결과를 저장하는 변수명을 옵셔널 변수명과 같도록 만듭시다.
새로운 변수명에 언래핑의 결과를 저장하는 방식은 권장하지 않습니다.

```swift
extension Customer: Customer
```











