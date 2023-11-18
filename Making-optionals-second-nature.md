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

관용적으로 Optional<Wrapped>보다 ?를 사용한 방식이 어울립니다.

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
```



















