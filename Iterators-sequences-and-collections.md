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

결과적으로 Sequence에서 Iterator들을 생성하고, 생성된 Iterator들이 각각의 Element들을 생성하는 방식으로 Sequence가 iteration 됩니다.

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

Sequence 프로토콜에서 makeIterator() 함수는 필수로 구현되어야 합니다.
makeIterator() 함수가 IteratorProtocol을 따르는 Iterator 타입을 리턴하기 때문에 Iterator와 Element 연관 값 또한 지정해야 합니다.

Sequence 프로토콜에는 많은 유용한 함수들이 정의되어 있습니다.
filter, map, reduce, flatMap, forEach, dropFrist, contains 등 Sequence 프로토콜에서 제공하는 함수는 여러가지가 있습니다.

Sequence 프로토콜을 따르는 스위프트 내장 데이터 타입 외에도 커스텀 타입에 Sequence 프로토콜을 채택하여 Sequence 프로토콜이 제공하는 
함수들을 커스텀 타입에도 적용할 수 있습니다.
물론 커스텀 타입이 Sequence 프로토콜의 연관 값과 makeIterator() 함수를 모두 구현해야 합니다.
이 과정에서 Sequence 프로토콜의 연관 값 Iterator를 위해 IteratorProtocol을 따르는 커스텀 타입 Iterator를 필요로 합니다.

## The powers of Sequence

Sequence 프로토콜은 유용한 함수들을 제공합니다.
이 함수들 중 필수적으로 알아야 할 함수들을 살펴 봅시다.

**filter**

filter 함수는 기존 컨테이너의 요소 중 조건에 만족하는 값에 대해서 새로운 컨테이너로 반환합니다.
filter 함수는 기존의 컨테이너의 요소에서 조건에 만족하는 값에 대해서 새로운 컨테이너로 반환하기 때문에, 
원하는 데이터를 추출할 때 편리하게 사용할 수 있습니다.

아래 코드는 filter 함수의 정의부입니다.

```swift
func filter(_ isIncluded: (Self.Element) throws -> Bool) rethrows -> [Self.Element]
```

데이터를 순회하며 각 Element들이 filter 함수의 클로저로 넘어가고 클로저에서 특정 조건을 통해 해당 Element를 새로운 컨테이너에 포함할지 결정합니다. 

```swift
let result = [1, 2, 3].filter { (value) -> Bool in
  return value > 1
}
print(result)  // [2, 3]

let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9]
let evenNumbers = numbers.filter { $0 % 2 == 0 }

print(evenNumbers)
// [2, 4, 6, 8]
```

**forEach**

for문과 동일한 순서로 컨테이너의 각 요소에 대해 주어진 클로저를 호출합니다.
아래 코드는 forEach의 정의부입니다.

```swift
func forEach(_ body: (Self.Element) throws -> Void) rethrows
```

for loop을 사용하다 보면 for loop과 함께 side effect이 발생하는 상황이 있습니다.
예를 들어 아래 코드와 같이 for loop을 돌며 Element를 지울 상황이 있을 때 forEach 구문을 사용할 수 있습니다.
아래 코드를 확인해 봅시다.

```swift
["file_one.txt", "file_two.txt"].forEach { path in
  deleteFile(path: path)
}

func deleteFile(path: path) {}
```

위의 코드를 아래 코드와 같이 고칠 수 있습니다.

```swift
["file_one.txt", "file_two.txt"].forEach(deleteFile)
```

In fact, if the function only accepts an argument and returns nothing, you can directly pass it to 'forEach' instead.

**enumerated**

loop를 도는 횟수를 추적하고 싶을 때 enumerated 함수를 사용할 수 있습니다.
enumerated 함수는 EnumeratedSequence라 불리는 Sequence를 리턴하는데 EnumeratedSequence로 iterations를 카운트합니다.

enumerated 함수가 리턴하는 EnumeratedSequence는 sequence of pairs (n, x)입니다.
여기서 n은 0에서 시작하는 연속 Int값이고 x는 기본 시퀀스의 요소입니다.
따라서 enumerated 함수의 리턴은 sequence of pairs입니다.

아래 코드로 확인해 봅시다.

```swift
["First line", "Second line", "Third line"]
  .enumerated()
  .forEach { (index: Int, element: String) in  // enumerated return sequence of pairs
    print("\(index+1): \(element)")
}  

// Output:
1: First line
2: Second line
3: Third line 
```

**Lazy iteration**

iterator를 통해 데이터를 순회할 때 sequence 안의 요소들에 즉시 접근하게 됩니다.
보통의 상황에서는 sequence의 요소들에 즉시 접근하는 것이 바람직합니다.
하지만 방대한 데이터 중 모든 데이터를 순회하지 않고 일부만 순회하고 싶을 때 lazy iteration을 사용하게 됩니다.

예를 들어 0부터 massive Int.max까지의 데이터 중에 짝수와 마지막 숫자 3개만 얻고 싶은 상황을 살펴봅시다.
lazy를 사용하여 전체 데이터를 순회하지 않을 수 있습니다.
lazy는 모든 Element가 아닌 일부의 Element만 계산하기 때문에 iteration에 사용되는 비용을 줄일 수 있습니다.

아래 코드로 살펴봅시다.

```swift
let bigRange = 0..<Int.max
```















































