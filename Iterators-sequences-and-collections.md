# Iterators, sequences and collections

## This chapter covers
- Taking a closer look at iteration in Swift
- Showing how Squence is related to IteratorProtocol
- Learning useful methods that Sequence supplies
- Understanding the different collection protocols
- Creating data structures with the Sequence and Collection protocols

## Iterating
개발을 하다보면 데이터를 순회하는 상황은 굉장히 자주 발생합니다.
우린 for loop 또는 while loop 등으로 데이터를 차례로 순회할 수 있습니다.

먼저 스위프트에서 데이터 순회(iteration)가 내부적으로 어떻게 동작하는지 살펴봅시다.

쉽게 생각해서 for loop을 사용할 때마다 우린 iterator를 사용하는 것입니다.
아래 코드처럼 우린 for loop을 사용합니다.

```swift
let cheeses = ["Gouda", "Camembart", "Brie"]

for cheese in cheeses {
  print(cheese)
}
```

하지만 놀랍게도 for in 구문은 syntactic sugar입니다.

실제로는 makeIterator() 함수로 iterator를 생성하고, while loop 안에서 iterator의 next() 함수를 반복합니다.
아래 코드로 살펴봅시다.

```swift
var cheeseItertor = cheeses.makeIterator()
while let cheese = cheeseIterator.next() {
  print(cheese)
}
```

cheese 객체에서 makeIterator() 함수를 호출할 수 있는 이유는 cheese 객체가 배열로 Sequence 프로토콜을 따르기 때문입니다.

makeIterator() 함수는 데이터 순회에 사용되는 iterator를 생성하는 함수로 Sequence 프로토콜에 선언되어 있습니다.
Sequence 프로토콜을 살펴보기 전에 Sequence 프로토콜과 관련된 IteratorProtocol을 먼저 살펴봅시다.

IteratorProtocol은 Element라고 명명한 연관 값을 갖고 Optional(Element) 데이터 타입을 리턴하는 next() 함수를 가지고 있습니다.
IteratorProtocol의 이름만 봐도 알 수 있듯이 Iterator의 형태를 정의해둔 프로토콜입니다.
Sequence 프로토콜이 가진 Iterator도 IteratorProtocol을 따르도록 구현해두었습니다.

Iterator는 데이터를 순회하며 Element를 하나씩 생성합니다.

아래 코드는 IteratorProtocol 코드입니다.
IteratorProtocl을 채틱하는 타입이라면 next() 함수를 필수로 구현해야 합니다.

```swift
public protocol IteratorProtocol {
  /// The type of element traversed by the iterator.
  associatedtype Element

  mutating func next() -> Element?
}

let groceries = ["Flour", "Eggs", "Sugar"]
// IndexingIterator를 리턴하지만 타입마다 달라질 수 있습니다.
var groceriesIterator: IndexingIterator<[String]> = groceries.makeIterator()
print(groceriesIterator.next())  // Optional("Flour")
print(groceriesIterator.next())  // Optional("Eggs")
print(groceriesIterator.next())  // Optional("Sugar")
print(groceriesIterator.next())  // nil
print(groceriesIterator.next())  // nil
```

next() 함수가 Element? 를 리턴하고 있습니다. 
옵셔널로 감싼 Element를 리턴하는 이유는 iterator가 모든 Element를 순회했을 때 iterator가 비었을 때 nil을 리턴하기 위함입니다.

위에서 살펴본 IteratorProtocol과 관련이 깊은 Sequence 프로토콜을 살펴봅시다.

Sequence 프로토콜은 굉장히 자주 쓰입니다.
Sequence 프로토콜이 Iterate의 기반이라고 생각 될 정도입니다.
또한 나중에 살펴볼 Collection 프로토콜의 부모 프로토콜입니다.

Array, Set, String, Dictionary 등이 Collection 프로토콜을 따르기 때문에 Collection 프로토콜의 부모 프로토콜인 Sequence 프로토콜도 따르게 됩니다.

Sequence는 iterator를 생성할 수 있습니다.
반면, iterator에 의해 순회하며 elements들이 소모되며 iterator가 비워지게 됩니다.
하지만 Sequence에서 새로운 iterator를 새로운 loop를 위해 만들 수 있습니다.
Sequence에서 새로운 iterator를 만들 수 있기 때문에 순회를 반복할 수 있습니다.

결과적으로 Sequence에서 Iterator들을 생성하고, 생성된 Iterator들이 각각의 Element들을 생성하는 방식으로 Sequence의 iteratioin이 동작하게 됩니다.

아래 코드로 Sequence 프로토콜을 살펴봅시다.

```swift
public protocol Sequence {
  associatedtype Element
  associatedtype Iterator: IteratorProtocol where Iterator.Element == Element

  func makeIterator() -> Iterator

  func filter(_ isIncluded: (Element) throws -> Bool) rethrows -> [Element]

  func forEach(_ body: (Element) throws -> Void) rethrows

  //...생략
}
```
























