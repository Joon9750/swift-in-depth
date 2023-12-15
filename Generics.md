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

모든 타입에 대응하는 제네릭에 타입 제약을 주는것을 역설적으로 느낄 수도 있습니다.
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
우린 프로토콜을 활용해 제네릭에 제약을 걸 수 있습니다.








































