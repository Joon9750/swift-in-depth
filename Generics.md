# Generics

## This chapter covers
- How and when to write generic code
- Understanding how to reason about generics
- Constraining generics with one or more protocols
- Making use of the Equatable, Comparable and Hashable protocols
- Creating highly reusable types
- Understanding how subclasses work with generics

## The benefits of generics

제네릭은 스위프트에 필수적이고 자주 등장합니다.
다형성을 위해서는 제네릭과 프로토콜은 필수적입니다.

제네릭 없이 여러 타입을 하나의 함수가 대응하기에는 어렵습니다.
물론 Any 타입을 사용해 여러 타입에 대응할 수 있지만, Any를 사용하면 런타임에 Any를 특정 타입(String, Int 등)으로 다운 캐스팅해야 합니다.
이런 보일러플레이트 코드를 피하기 위해 제네릭이 사용됩니다.

제네릭으로 컴파일 타입에 다형성을 이룰 수 있습니다.
아래 코드는 제네릭 없이 사용하던 함수를 제네릭 함수로 고친 코드입니다.

```swift
// 제네릭 X
func firstLast(array: [Int]) -> (Int, Int) {
  return (array[0], array[array.count - 1])
}

// 제네릭 O
func firstLast<T>(array:[T]) -> (T, T) {
  return (array[0], array[array.count -1])
}
```

제네릭을 사용할 때는 함수명 뒤에 <T>를 붙이고 사용되는 타입도 T로 바꾸면 됩니다.
물론 T 말고도 Wrapped, U , V 등 자유롭게 사용할 수 있습니다.

하지만 T로 제네릭을 구현했다면 T 타입을 가진 변수는 모두 동일한 데이터 타입을 따르게 됩니다.
제네릭에 Int, String 등과 함께 커스텀 데이터 타입도 넘길 수 있습니다.
제네릭으로 타입에 제약 받지 않을 수 있습니다.

결과적으로 제네릭으로 하나의 함수로 여러 타입을 대응 가능합니다. Any를 사용했다면 런타임에 다운 캐스팅을 해야하지만 제네릭으로 컴파일 타임에 모든 타입을 선언할 수 있습니다.
처음부터 제네릭 함수를 구현하는건 어려울 수 있습니다. 먼저 제네릭 없이 함수를 구현해보고 제네릭 함수로 고치는 방식이 더 쉽습니다.

제네릭은 타임에 제약 없이 여러 타임에 대응하며 유용히 쓰입니다. 하지만 제네릭을 사용하면 주의해야 할 부분이 있습니다.
아래 코드로 확인해 봅시다.

```swift
func illegalWrap<T>(value: T) -> [Int] {
  return [value]
}
```

illegalWrap 함수는 제네릭 타입의 value를 입력 받고 [Int]를 리턴하고 있습니다.
제네릭은 타입을 특정할 수 없습니다. 하지만 illegalWrap 함수에서는 제네릭을 Int로 특정하고 있기 때문에 컴파일 에러가 발생합니다. 

아래 코드로 더 확인해 봅시다.

```swift
// 컴파일 에러
func wrap<T> (value: Int, secondValue: T) -> ([Int], U) {
  return ([value], secondValue)
}

// 컴파일 가능
func wrap<T>(value: Int, secondValue: T) -> ([Int], T) {
  return ([value], secondValue)
}

// 컴파일 에러
func wrap(value: Int, secondValue: T) -> ([Int], T) {
  return ([value], secondValue)
}

// 컴파일 에러
func wrap<T>(value: Int, secondValue: T) -> ([Int], Int) {
  return ([value], secondValue)
}

// 컴파일 가능
func wrap<T>(value: Int, secondValue: T) -> ([Int], Int)? {
  if let secondValue = secondValue as? Int {
    return ([value], secondValue)
  } else {
    return nil
  }
}
```

제네릭을 사용하면 컴파일러가 Value Witness Tables 메타 데이터를 활용하여 제네릭 함수를 구체화하는 과정에서 구체적인 타입(Int 등)으로 치환된 코드를 컴파일러가 반복적으로 생성합니다.
제네릭을 사용하면 어떤 값을 다룰지 컴파일 타임에 알 수 있다는 장점이 있습니다. (좀 더 찾아 구체적으로 설명하겠습니다.)

## Constraining generics

지금까지 소개한 제네릭은 모든 타입을 받을 수 있는 상태였습니다. 하지만 타입이 제한되지 않은 제네릭으로는 많은 일을 할 수 없습니다.
오히려 프로토콜을 통해 제네릭을 제한하면 더 유용히 제네릭을 쓸 수 있습니다.

모든 타입에 대응하는 제네릭에 타입 제약을 주는것이 역설적으로 느낄 수도 있습니다.

제네릭을 제한하지 않았을 때 문제가 되는 상황을 소개하겠습니다.
아래 lowest 함수는 가장 작은 값을 리턴하는 제네릭 함수입니다.

```swift
// 아직 미완성된 lowest 함수입니다.
func lowest<T>(_ array: [T]) -> T? {
  let sortedArray = array.sorted { (lhs, rhs) -> Bool in
    return lhs < rhs
  }
  return sortedArray.first
}

lowest([3, 1, 2]) // Optional(1)
lowest([40.2, 12.3, 99.9]) // Optional(12.3)
lowest(["a", "b", "c"]) // Optional("a")
```

위의 lowest 함수는 에러를 발생시킵니다.
lowest 함수에서 제네릭을 제한하지 않았기 때문에 T에 모든 타입이 들어올 수 있습니다.
lowest 함수 안에서 비교 연산(<)을 수행하기 때문에 비교 연산이 가능하지 않은 타입이 입력으로 들어올 수 있기 때문에 에러가 발생합니다.

비교 연산이 가능한 타입만 함수 입력으로 들어오도록 제네릭 제약을 걸어 에러를 해결할 수 있습니다.
우린 프로토콜을 통해 제네릭에 제약을 걸 수 있습니다.

그렇다면 비교 연산과 관련있는 프로토콜은 어떤게 있을까요?
Equatable 프로토콜과 Comparable 프로토콜을 살펴봅시다.

먼저 Equatable 프로토콜은 두 값이 같은지 확인하는데 쓰입니다.
동일한 타입간의 == 비교연산자를 제공합니다.
따라서 Equatable 프로토콜을 채택했을 때 static == 함수를 필수로 구현해야 합니다.

아래 코드는 Equatable 프로토콜 코드입니다.

```swift
public protocol Equatable {
  static func == (lhs: Self, rhs: Self) -> Bool
}
```

Equatable 프로토콜의 static == 함수를 통해 동일한 타입간의 '같다'는 기준을 만들 수 있습니다.

그렇다면 Comparable 프로토콜은 어떨까요?

Comparable 프로토콜은 Equatable 프로토콜과 마찬가지로 static 함수가 있지만 구현을 필수로 요구하지 않습니다.
아래 Comparable 프로토콜 코드를 확인하면 Equatable을 채택한다는걸 알 수 있습니다.

```swift
public protocol Comparable: Equatable {
  static func < (lhs: Self, rhs: Self) -> Bool
  static func <= (lhs: Self, rhs: Self) -> Bool
  static func >= (lhs: Self, rhs: Self) -> Bool
  static func > (lhs: Self, rhs: Self) -> Bool
}
```

Comparable 프로토콜을 채택할 경우 static 함수 중 하나를 구현하면 나머지 static 함수들은 스위프트가 유추하기 때문에 추가적으로 구현할 필요가 없습니다.
Int, float, string은 기본적으로 Comparable을 따르는 타입입니다.

위에서 lowest 함수의 제네릭을 Comparable 프로토콜로 제약을 걸어 함수 안의 비교 연산에 문제가 없도록 고쳐봅시다.
아래 코드는 lowest 함수의 제네릭을 Comparable 프로토콜로 제약한 코드입니다.
이제 lowest 함수의 입력은 Comparable 프로토콜을 따르는 타입이어야 합니다.

```swift
func lowest<T: Comparable>(_ array: [T]) -> T? {
  let sortedArray = array.sorted { (lhs, rhs) -> Bool in
    return lhs < rhs  // Comparable을 따르는 타입만 입력으로 들어오기 때문에 비교 연산이 가능합니다.
  }
  return sortedArray.first
}

// lowest 간략한 버전
func lowest<T: Comparable>(_ array: [T]) -> T? {
  return array.sorted().first  // 입력이 항상 Comparable 프로토콜을 따르기 때문에 내장 함수인 sorted()도 사용 가능합니다.
}
```

그렇다면 lowest 함수에 입력으로 들어올 커스텀 타입은 어떤게 있을까요?
물론 Int, float, string도 가능하겠지만 Comparable 프로토콜을 따르는 커스텀 타입도 입력으로 들어갈 수 있습니다.

아래 코드는  Comparable 프로토콜을 따르는 커스텀 타입인 RoyalRank 타입 코드입니다.
위에서 말했듯이 Comparable 프로토콜이 지원하는 static 함수 중 하나를 구현하여 나머지 static 함수를 컴파일러가 유추 가능하여 구현하지 않은 static 함수도 사용할 수 있습니다.

```swift
enum RoyalRank: Comparable {
  case emperor
  case king
  case duke

  static func <(lhs: RoyalRank, rhs: RoyalRank) -> Bool {
    switch (lhs, rhs) {
      case (king, emperor): return true
      case (duke, emperor): return true
      case (duke, king): return true
      default: return false
    }
  }
 }

let king = RoyalRank.king
let duke = RoyalRank.duke

duke < king // true
duke > king // false
duke == king // false

let ranks: [RoyalRank] = [.emperor, .king, .duke]
lowest(ranks)  // .duke
```

제네릭의 장점 중 하나는 아직 존재하지 않은 타입에도 대응할 수 있다는 점입니다.

모든 타입이 Comparable 프로토콜을 따르진 않습니다.
예를 들어 Bool 타입은 Comparable 프로토콜을 따르지 않습니다. 따라서 lowest 함수에 Bool 타입이 입력으로 들어갈 수 없습니다.

Comstraining a generic means trading flexibility for functionality. A constrained generic becomes more specialized but is less flexible.

## Multipule constraints

제네릭을 제약해 사용할 때 하나의 프로토콜 만으로는 부족한 경우가 있습니다. 
예를 들어 값을 비교하고 해당 값을 딕셔너리에 저장한다면, 이를 위해 Comparable 프로토콜과 Hashable 프로토콜을 모두 따르는 타입이어야 합니다.

먼저 Hashable 프로토콜을 살펴 봅시다.
아래 코드는 Hashable 프로토콜 코드입니다.

```swift
public protocol Hashable: Equatable {
  func hash(into hasher: inout Hasher) {
    // ...생략
  }
}
```

Hashable 프로토콜은 Equatable 프로토콜을 채택하고 있습니다.
Hashable 프로토콜을 채택한다면 Equatable 프로토콜의 static == 함수와 Hashable 프로토콜의 hash 함수를 모두 필수로 구현해야 합니다.
구조체와 열거형에서는 Equatable과 Hashable 프로토콜의 필수 구현 함수 중 하나를 구현하면 나머지 함수들을 컴파일러가 유추하지만 클래스에서는
유추하지 못하기 때문에 클래스의 경우 모든 함수를 구현해야 합니다.

Hashable 프로토콜의 hash 함수를 통해 hashvalue로 불리는 값을 생성합니다.
예를 들어 "Hedgehog"을 hash 함수에 입력하면 43092483과 같은 hashvalue가 리턴됩니다.
Hashable 프로토콜을 따르는 타입은 주로 딕셔너리의 키 값이나 Set의 일부 등에 사용됩니다.

같은 타입의 인스턴스 a와 b가 있을 경우 a == b이면 a.hashValue == b.hashValue 입니다.
하지만 hashValue가 같다고 해서 동일한 인스턴스는 아닐 수 있습니다.

또한 hashValue는 프로그램의 실행에 따라 달라질 수 있습니다.
따라서 이후 실행에 사용할 hashValue 값을 저장하지 않는게 좋습니다.

대부분의 스위프트 내장 타입들은 Equatable과 Hashable을 채택하고 있습니다.

아래 코드와 같이 제네릭 타입을 하나 이상의 프로토콜로 제약할 수 있습니다.

```swift
func lowestOccurrences<T: Comparable & Hashable>(values: [T]) -> [T: Int] {
  //...생략
}
```

제네릭 제약을 여러 프로토콜로 할 때 where 구문을 사용하면 더 간략히 표현 가능합니다.
아래 코드로 확인해 봅시다.

```swift
func lowestOccurrences<T>(values: [T]) -> [T: Int] {
  where T: Comparable & Hashable {
  // ...생략
}
```

## Creating a generic type

지금까지는 제네릭 함수만 살펴보았지만 제네릭 타입도 만들 수 있습니다.
옵셔널 또한 제네릭 타입으로 구현되어 있습니다.
아래 코드를 살펴봅시다.

```swift
public enum Optional<Wrapped> {
  case none
  case some(Wrapped)
}
```

옵셔널과 배열 등은 모두 제네릭 타입으로 구현되어 있습니다.
우린 커스텀 타입을 제네릭 타입으로 만드는 방법도 이해하고 있어야 합니다.

딕셔너리의 키로 두 개의 Hashable 타입을 함께 사용하는건 불가능합니다.

물론 튜플로 두 개의 Hashable 타입인 String 타입을 묶고 딕셔너리의 키로 사용하는 방법도 스위프트에서 허용하지 않습니다.
아래 코드로 살펴봅시다.

```swift
let stringsTuple = ("I want to be part of a key", "Me too!")
let anotherDictionary = [stringsTuple: "I am a value"]  // error: type of expression is ambiguous without more context
```

두 개의 Hashable 타입을 튜플에 넣으면 더 이상 해당 튜플은 Hashable 타입이 아닙니다.
이런 경우 딕셔너리의 키로 두 개의 Hashable 타입을 사용하려면 어떻게 할까요?

Pair 커스텀 타입을 만들어 해결할 수 있습니다.
Pair 타입에 Hashable 타입 두 개를 묶어 Hashable 타입으로 딕셔너리의 키에 사용될 수 있습니다.
아래 코드로 Pair 타입 구현을 살펴봅시다.








