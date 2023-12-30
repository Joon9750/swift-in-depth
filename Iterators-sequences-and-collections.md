# Iterators, sequences and collections

## This chapter covers
- Taking a closer look at iteration in Swift
- Showing how Squence is related to IteratorProtocol
- Learning useful methods that Sequence supplies
- Understanding the different collection protocols
- Creating data structures with the Sequence and Collection protocols

## Iterating
개발하다 보면 데이터를 순회하는 상황은 굉장히 자주 발생합니다.
우린 for loop 또는 while loop 등으로 데이터를 차례로 순회할 수 있습니다.

먼저 스위프트에서 데이터 순회(iteration)가 내부적으로 어떻게 동작하는지 살펴봅시다.

쉽게 생각하면 for loop을 사용할 때마다 우린 iterator를 사용하는 것입니다.
아래 코드는 for loop을 사용한 일반적인 데이터 순회 방식입니다.

```swift
let cheeses = ["Gouda", "Camembart", "Brie"]

for cheese in cheeses {
  print(cheese)
}
```

하지만 놀랍게도 for in 구문은 syntactic sugar입니다.

for loop을 사용하면 내부적으로 makeIterator() 함수에 의해 iterator가 생성되고, while loop 안에서 iterator의 next() 함수를 반복합니다. 아래 코드는 for loop의 내부적인 동작 방식입니다.

```swift
var cheeseItertor = cheeses.makeIterator()
while let cheese = cheeseIterator.next() {
  print(cheese)
}
```

cheese 객체에서 makeIterator() 함수를 호출할 수 있는 이유가 무엇일까요?
cheese 객체가 Sequence 프로토콜을 따르는 Array 타입이기 때문입니다.

makeIterator() 함수는 데이터 순회에 사용되는 iterator를 생성하는 함수로 Sequence 프로토콜에 선언되어 있습니다.
Sequence 프로토콜을 살펴보기 전에 Sequence 프로토콜과 관련된 IteratorProtocol을 먼저 살펴봅시다.

**IteratorProtocol**

IteratorProtocol은 연관 값 Element와 Optional(Element) 데이터 타입을 리턴하는 next() 함수를 가지고 있습니다.
IteratorProtocol의 이름만 봐도 알 수 있듯이 Iterator의 형태를 정의한 프로토콜입니다.
Sequence 프로토콜이 프로퍼티로 가진 Iterator도 IteratorProtocol을 따르도록 정의하고 있습니다.

Iterator는 데이터를 순회(iteraion)하며 Element를 하나씩 생성합니다.

아래 코드는 IteratorProtocol 정의부입니다.
IteratorProtocol을 채택하는 타입이라면 next() 함수를 필수로 구현해야 합니다.
또한 연관 값 Element에 대한 선언도 필수적입니다.

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

위의 코드에서 next() 함수가 Element? 타입을 리턴하고 있습니다. 
next() 함수가 옵셔널로 감싼 Element를 리턴하는 이유는 iterator가 모든 Element를 순회하여 iterator가 비었을 때 nil을 리턴하기 위함입니다.

이제는 위에서 살펴본 IteratorProtocol과 관련이 깊은 Sequence 프로토콜을 살펴봅시다.

**Sequence**

Sequence 프로토콜은 개발하며 굉장히 자주 쓰입니다.
Sequence 프로토콜이 데이터 순회의 기반이라고 생각될 정도입니다.
또한 나중에 살펴볼 Collection 프로토콜의 부모 프로토콜입니다.

Array, Set, String, Dictionary 등이 Collection 프로토콜을 따르기 때문에 Collection 프로토콜의 부모 프로토콜인 Sequence 프로토콜도 따르게 됩니다.

Sequence 프로토콜은 makeIterator() 함수를 통해 iterator를 생성할 수 있습니다.
반면, iterator에 의해 순회하며 elements들이 소모되며 iterator가 비워지게 됩니다.
하지만 Sequence에서 새로운 loop을 위한 새로운 iterator를 만들 수 있습니다.
Sequence에서 새로운 iterator를 만들 수 있기 때문에 계속해서 순회를 반복할 수 있습니다.

결과적으로 Sequence에서 Iterator들을 생성하고, 생성된 Iterator들이 Element들을 생성하는 방식으로 Sequence가 iteration 됩니다!

아래 코드로 Sequence 프로토콜 정의부를 살펴봅시다.

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
makeIterator() 함수가 IteratorProtocol을 따르는 Iterator 타입을 리턴하기 때문에 Iterator 연관 값을 정의해야 합니다.
또한 Sequence 프로토콜의 Element 연관 값도 정의해야 하고 Iterator.Element의 타입도 Sequence 프로토콜의 Element 연관 값과 같아야 합니다.

Sequence 프로토콜에는 많은 유용한 함수들이 정의되어 있습니다.
filter, map, reduce, flatMap, forEach, dropFrist, contains 등 Sequence 프로토콜에서 여러가지 함수를 제공합니다.

Sequence 프로토콜을 따르는 스위프트 내장 데이터 타입 외에도 커스텀 타입에 Sequence 프로토콜을 채택하여 Sequence 프로토콜이 제공하는 
함수들을 커스텀 타입에도 적용할 수 있습니다.
물론 커스텀 타입이 Sequence 프로토콜의 연관 값 Element, Iterator와 makeIterator() 함수를 모두 구현해야 합니다.
이 과정에서 Sequence 프로토콜의 연관 값 Iterator를 위해 IteratorProtocol을 따르는 커스텀 타입 Iterator가 필요합니다.

## The powers of Sequence

지금부터 Sequence 프로토콜이 제공하는 몇 가지 함수들을 살펴봅시다.

**filter**

filter 함수는 기존 컨테이너의 요소 중 조건에 만족하는 값에 대해서 새로운 컨테이너로 반환합니다.
따라서 filter 함수는 원하는 데이터를 추출할 때 편리하게 사용할 수 있습니다.

아래 코드는 filter 함수의 정의부입니다.

```swift
func filter(_ isIncluded: (Self.Element) throws -> Bool) rethrows -> [Self.Element]
```

데이터를 순회하며 각 Element들이 filter 함수의 클로저로 넘어가고 클로저에서 특정 조건을 통해 해당 Element를 새로운 컨테이너에 포함할지 결정합니다. 
새로운 컨테이너를 filter 함수에서 리턴하기 때문에 기존 컨테이너의 데이터는 변형되지 않습니다.

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

for 구문과 동일한 순서로 컨테이너의 각 요소에 대해 주어진 클로저를 호출합니다.

위에서 보았던 filter 함수와 달리 forEach 함수는 새로운 컨테이너를 리턴하지 않습니다.
forEach 함수는 해당 컨테이너의 데이터를 직접 변경하게 됩니다.

아래 코드는 forEach의 정의부입니다.

```swift
func forEach(_ body: (Self.Element) throws -> Void) rethrows
```

for loop을 사용하다 보면 for loop 안에서 side effect가 발생하는 상황이 있습니다.

예를 들어 아래 코드와 같이 for loop을 돌며 Element를 지울 상황이 있을 때 forEach 구문을 사용할 수 있습니다.
(여기서 side effect는 Element가 지워지는걸 말합니다.)

아래 코드를 확인해 봅시다.

```swift
// 고치기 전 코드입니다.
["file_one.txt", "file_two.txt"].forEach { path in
  deleteFile(path: path)
}

// forEach 함수를 사용해 고친 코드입니다.
["file_one.txt", "file_two.txt"].forEach(deleteFile)

func deleteFile(path: path) {}
```

In fact, if the function only accepts an argument and returns nothing, you can directly pass it to 'forEach' instead.

**enumerated**

데이터 순회를 추적하고 싶을 때 enumerated 함수를 사용할 수 있습니다.
enumerated 함수는 EnumeratedSequence라 불리는 Sequence를 리턴합니다.
enumerated 함수에서 리턴하는 EnumeratedSequence로 iterations를 추적합니다.

enumerated 함수가 리턴하는 EnumeratedSequence는 sequence of pairs (n, x) 형태입니다.
여기서 n은 0에서 시작하는 연속되는 Int 값이고 x는 기본 시퀀스의 요소입니다.

아래 코드로 확인해 봅시다.

```swift
["First line", "Second line", "Third line"]
  .enumerated()  // enumerated() 함수가 (n, x) 쌍의 Sequence를 리턴합니다.
  .forEach { (index: Int, element: String) in  // enumerated return sequence of pairs
    print("\(index+1): \(element)")
}  

// Output:
1: First line
2: Second line
3: Third line 
```

**Lazy iteration**

보통의(lazy 키워드 없이) iterator를 통해 데이터를 순회할 때 sequence 안의 요소들에 즉시 접근하게 됩니다.
일반적인 상황에서는 sequence의 요소들에 즉시 접근하는 것이 바람직합니다.

하지만 방대한 데이터 중 모든 데이터를 순회하지 않고 일부만 순회하고 싶을 때 lazy iteration을 사용하면 됩니다.
또한 lazy iteration은 lazy 변수에 접근할 때 계산이 시작됩니다.

아래 코드로 lazy iteration을 사용하는 상황을 살펴봅시다.

```swift
// lazy 변수 없이 일반적인 iteration
func increment(x: Int) -> Int {
  print("Cumputing next value of \(x)")
  return x+1
}

let array = Array(0..<10)
let incArray = array.map(increment)  // 여기 이미 increment 함수가 계산됩니다.
print("Result: ")
print(incArray[0], incArray[4])

// Output:
Computing next value of 0
Computing next value of 1
Computing next value of 2
Computing next value of 3
Computing next value of 4
Computing next value of 5
Computing next value of 6
Computing next value of 7
Computing next value of 8
Computing next value of 9
Result:
1 5
```

위 코드를 사용하면 incArray 값에 접근하기 전에 모든 출력값이 계산됩니다.
만약 increment가 10이 아니라 더 큰 값을 가진다면 초기 iteration에 큰 비용이 들게 됩니다.

이런 상황에서 lazy 키워드를 사용해 성능을 향상할 수 있습니다.

```swift
// lazy iteration
func increment(x: Int) -> Int {
  print("Cumputing next value of \(x)")
  return x+1
}

let array = Array(0..<10000)
let incArray = array.lazy.map(increment)  // lazy 키워드에 의해 여기서 increment 함수가 계산되지 않습니다.
print("Result: ")
print(incArray[0], incArray[4])  // 여기서 increment 함수가 계산됩니다.

// Output:
Result: 
Computing next value of 0
Computing next value of 4
1 5
```

lazy 키워드 성격 그대로 incArray에 접근할 때 계산을 시작합니다.
또한 앞서 말했듯이 sequence의 모든 element에 접근하지 않고 일부 element에만 접근하여 성능을 향상할 수 있습니다.

아래 링크에서 추가적인 설명을 볼 수 있습니다.

https://zeddios.tistory.com/387

**reduce**

reduce 함수는 sequence의 element를 순회하며 특정 값을 누적하고, reduce 함수가 끝날 때 누적된 값을 리턴합니다.
다시 말해 reduce 함수로 여러 elements를 새로운 값으로 변형할 수 있습니다.

아래는 reduce 함수의 정의부입니다.

```swift
func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) throws -> Result) rethrows -> Result
```

먼저 reduce 함수가 제네릭임을 알 수 있습니다.

reduce 함수의 매개변수로 변수 initialResult와 클로저 nextPartialResult를 입력 받습니다.
initialResult는 초기값으로 사용할 값을 넣으면 클로저가 처음 실행될 때, nextPartialResult의 Result에 전달됩니다.
그리고 nextPartialResult은 컨테이너의 요소를 새로운 누적값으로 결합하는 클로저입니다.

reduce 함수의 iteration이 진행되며 발생되는 Element가 nextPartialResult 클로저의 Element로 넘어오고 이전의 클로저에서 리턴하는 값이 nextPartialResult 클로저의 Result로 넘어옵니다.

두 번째 파라미터로 전달된 클로저를 좀 더 자세히 보겠습니다.
클로저의 첫 번째 파라미터는 initialResult로 전달받은 초기값 또는 이전 클로저가 반환하는 return 값이 전달됩니다.
클로저의 두 번째 파라미터는 reduce(_:_:)를 호출한 컨테이너의 요소가 전달됩니다.
클로저가 반환하는 값은 파라미터로 받은 두 값을 적절하게 처리하여 첫 번째 파라미터로 받은 값과 같은 타입의 값을 리턴해줍니다.

reduce 함수의 리턴으로는 최종 누적값이 반환되며, 컨테이너의 요소가 없다면 initialResult 의 값이 반환됩니다.

아래 코드는 reduce 함수를 사용하여 '\n'의 개수를 계산한 예시입니다.

```swift
let text = "This is some text. \nAnother line. \nYet, another line again."
let startValue = 0

let numberOfLineBreaks = text.reduce(startValue) { (accumulation: Int, char: Character) in
  if char == '\n' {
    return accumulation + 1
  } else {
    return accumulation
  }
}
```

**reduce into**

먼저 reduce 함수와 reduce(into:) 함수의 정의부를 모두 확인하고 비교해 봅시다.

```swift
func reduce<Result>(
  _ initialResult: Result,
  _ nextPartialResult: (Result, Element) throws -> Result
) rethrows -> Result

func reduce<Result>(
  into initialResult: Result,
  _ updateAccumulatingResult: (inout Result, Element) throws -> ()  // reduce(into:)가 reduce()와 다른 부분입니다.
) rethrows -> Result
```

reduce 함수는 두 가지 형태로 사용됩니다.

위에서 보았듯이 기본적인 reduce 함수 그리고 reduce(into:) 함수 형태로 사용할 수 있습니다.
초기값으로 값 타입이 들어온다면 reduce(into:) 함수를 사용합시다.

reduce 함수의 두 번째 매개변수인 클로저의 Result는 immutable 합니다.
따라서 reduce 함수의 nextPartialResult 클로저로 들어오는 Result가 Array, Dictionary 등 값 타입으로 들어오면 직접 수정할 수 없기 때문에 mutable한 변수(var)로 복사 후 계산하고 리턴해야 하는 번거로움이 있습니다.

아래 코드로 reduce() 함수에 값 타입이 들어온 상황을 먼저 살펴봅시다.

```swift
let grades = [3.2, 4.2, 2.6, 4.1]
let results = grades.reduce([:]) { (results: [Character: Int], grade: Double) in
  var copy = results  // results가 딕셔너리 타입이기 때문에 var 변수로 results를 복사해서 사용해야 합니다.
  switch grade {
  case 1..<2: copy["D", default: 0] += 1
  case 2..<3: copy["C", default: 0] += 1
  case 3..<4: copy["B", default: 0] += 1
  case 4...: copy["A", default: 0] += 1
  default: break
  }
  return copy
}
```

위와 같이 반복적인 복사는 성능 저하로 이어집니다.

그에 반해 reduce(into:) 함수는 reduce 함수와 첫 번째 초기값을 받는 매개변수는 동일하지만, 두 번째 매개변수에 inout 키워드가 
추가되었고 클로저의 리턴 값이 없습니다. 리턴 값 없이 inout 키워드가 붙은 Result 값을 변경하며 최종값에 도달합니다.

reduce(into:_:)의 가장 큰 차이는 클로저의 첫 번째 파라미터가 inout 파라미터라는 점입니다.
즉, initialResult로 전달받은 값을 클로저안에서 직접 변경할 수 있고 두 번째 클로저의 Result 또한 변경할 수 있습니다.
reduce(_:_:)는 initialResult로 전달받은 값을 클로저에서 변경할 수 없습니다. (let 변수)

위에서 reduce 함수를 사용한 방식을 reduce(into:) 함수로 고친 코드입니다.

```swift
let grades = [3.2, 4.2, 2.6, 4.1]
let results = grades.reduce([:]) { (results: inout [Character: Int], grade: Double) in
  switch grade {
  case 1..<2: results["D", default: 0] += 1
  case 2..<3: results["C", default: 0] += 1
  case 3..<4: results["B", default: 0] += 1
  case 4...: results["A", default: 0] += 1
  default: break
  }
}
```

iteration 때마다 딕셔너리의 복사본을 만들면 성능 측면에서도 떨어지게 됩니다.
배열이나 딕셔너리와 같은 값 타입과 reduce 함수를 사용할 때는 into 키워드를 함께 사용해 성능을 높입시다.

reduce 함수 사용을 추천하는 이유는 명백합니다.

for loop을 사용할 경우 for 구문 안의 코드를 읽어야 for 구문의 목적을 알 수 있습니다.
하지만 reduce 함수를 사용하면 그 자체로 sequence를 순회하며 새로운 누적값을 생성한다는 사실을 알 수 있습니다.
또한 간결한 코드를 구현할 수 있습니다.

아래 링크로 reduce와 reduce(into:) 함수의 더 많은 예시 코드를 확인해 봅시다.

https://dejavuqa.tistory.com/181, 
https://beepeach.tistory.com/606,
http://yoonbumtae.com/?p=4390

**zip**

마지막으로 살펴볼 함수는 zip입니다. 
zip으로 두 iterator를 합칠 수 있습니다.
둘 중 하나라도 iterator가 고갈되면 zip에서의 iteration은 종료됩니다.

아래 코드를 살펴봅시다.

```swift
for (integer, string) in zip(0..<10, ["a", "b", "c"]) {
  print("\(integer): \(string)")
}
```

## Creating a generic data structure with Sequence

이제는 Sequence 프로토콜을 따르는 커스텀 타입을 만들어 봅시다.
자료구조 중 'Bag' 자료구조를 직접 구현해볼 생각입니다.

Bag을 만들 때, Sequence 프로토콜과 IteratorProtocol을 사용해 Bag 자료구조에 iteration을 제공하고 유용한 IteratorProtocol의 함수를 Bag 타입에 제공하려 합니다.

Bag은 Set과 비슷하게 정렬되지 않는 상태로 데이터를 저장하며 요소를 추가하고 조회할 수 있습니다.
하지만 Set과 달리 동일한 요소를 중복해서 Bag에 넣을 수 있습니다.

아래 코드로 확인해 봅시다.

```swift
var bag = Bag<String>()
bag.insert("Huey")
bag.insert("Huey")
bag.insert("Huey")

bag.insert("Mickey")
bag.remove("Huey")

bag.count  // 3
print(bag) // CustomeStringConvertible 프로토콜을 채택하여 description을 커스텀했습니다.
// Output:
// Heuy occurs 2 times
// Mickey occurs 1 times

let anotherBag: Bag = [1.0, 2.0, 2.0, 3.0, 3.0, 3.0]
print(anotherBag)
// Output:
// 2.0 occurs 2 times
// 1.0 occurs 1 time
// 3.0 occurs 3 times
```

이처럼 Bag 자료구조에는 동일한 요소를 중복해서 넣을 수 있습니다.
그렇다면 Bag 자료구조에서 동일한 요소가 중복해서 들어올 경우 어떤 동작이 일어날까요?

Bag 자료구조에서는 동일한 요소가 저장될 경우 실제 객체로 동일한 요소를 중복해서 저장하지 않고 객체의 counter를 갱신합니다.
Bag의 element 각각에는 counter가 존재하고 counter가 0이 되었을 때 해당 element는 삭제됩니다.

중복되는 객체를 Bag에 실제로 저장하기보다 객체의 counter를 통해 중복되는 객체를 관리했을 때 메모리 사용량 측면에서 이점이 있습니다.

또한 Bag은 Set과 마찬가지로 제네릭 타입입니다.

따라서 Hashable 프로토콜을 따르는 모든 타입이 Bag에 들어올 수 있습니다.
물론 여러 타입을 하나의 Bag에 섞어서 저장 할 수 없습니다.

아래 코드는 Bag 자료구조를 구현한 코드입니다.

```swift
struct Bag<Element: Hashable> {
  private var store = [Element: Int]()

  mutating func insert(_ element: Element) {
    store[element, default: 0] += 1
  }

  mutating func remove(_ element: Element) {
    store[element]? -= 1
    if store[element] == 0 {
      store[element] = nil
    }
  }

  var count: Int {
    return store.values.reduce(0, +)
  }
}
```

Bag의 저장되는 요소들을 관리하는 store 변수를 딕셔너리로 만들어 
Element에 대응하는 Int가 Element의 개수를 관리는 counter의 역할을 합니다.

또한 store 변수가 딕셔너리로 값 타입이기 때문에 store 변수의 값을 다루는 함수인 insert와 remove는 mutating 키워드를 붙여야 합니다.

만약 Bag 타입을 출력할 때 출력 형식을 커스텀하고 싶다면 CustomStringConvertible 프로토콜을 사용할 수 있습니다.
CustomStringConvertible 프로토콜을 Bag 타입에 채택하여 출력(print)형식을 커스텀할 수 있습니다.

아래 코드로 확인해 봅시다.

```swift
extension Bag: CustomeStringConvertible {
  var description: String {
    var summary = String()
    for (key, value) in store {
      let times = value == 1 ? "time" : "times"
      summary.append("\(key) occurs \(value) \(times) \n")
    }
    return summary
  }
}

let anotherBag: Bag = [1.0, 2.0, 2.0, 3.0, 3.0, 3.0]
print(anotherBag)
// Output:
// 2.0 occurs 2 times
// 1.0 occurs 1 time
// 3.0 occurs 3 times
```

CustomeStringConvertible 프로토콜의 description 프로퍼티를 커스텀하면 
Bag 타입을 출력(print)할 때 해당 description에 맞춰 출력됩니다.

지금까지 Bag 타입을 구현했습니다.

하지만 Sequence 프로토콜을 채택하지 않은 상태의 Bag 타입에서 iteration을 진행할 수 없습니다.

Sequence 프로토콜을 Bag 타입에 채택하기 위해서는 Bag 타입을 위한 iterator가 필요합니다.
당연히 Bag 타입을 위한 iterator 또한 IteratorProtocol을 따르는 iterator이어야 합니다.

IteratorProtocl을 채택한 iterator가 element를 생성하기 위해서는 iterator로 Bag 타입 객체(store)의 복사본을 전달해야 합니다.
iterator는 Bag 타입 객체(store)의 복사본을 다루기 때문에 iterator에서 복사본 store에 변형을 가해도
실제 Bag 객체의 store 변수에는 영향을 미치지 않습니다.

위에서 이야기했듯이 Bag 자료구조는 중복되는 요소를 입력받지만 이를 실제 메모리를 사용하며 저장하지 않고 counter로 요소를 관리합니다.

따라서 Bag 타입의 iterator에서도 element의 counter를 보고 counter가 관리하는 수 만큼의 요소를 리턴하게 됩니다.
IteratorProtocol을 채택한 iterator에서 next() 함수를 통해 sequence를 순회하며 요소를 리턴하고 counter를 줄입니다.
특정 요소의 counter가 0이 된다면 nil을 리턴하게 됩니다.

아래 코드는 BagIterator를 구현한 코드입니다.

```swift
// IteratorProtocol 구현부입니다.
public protocol IteratorProtocol {
  /// The type of element traversed by the iterator.
  associatedtype Element

  mutating func next() -> Element?
}

// BagIterator 구현부입니다.
struct BagIterator<Element: Hashable>: IteratorProtocol {
  // Bag의 복사본을 store 변수에 저장해야 합니다.
  var store = [Element: Int]()

  mutating func next() -> Element? {
    guard let (key, value) = store.first else {
      return nil
    }
    if value > 1 {
      store[key]? -= 1
    } else {
      store[key] = nil
    }
    return key
  }
}  
```

위의 BagIterator 구조체의 next() 함수에서 iteration에서의 요소 순회하는 기준을 설정할 수 있습니다.
BagIterator의 store 딕셔너리에 Bag의 store 딕셔너리의 복사본을 넣게 됩니다.

BagIterator를 구현했다면, 이제 Bag 타입에 Sequence 프로토콜을 채택할 준비가 되었습니다.
Sequence 프로토콜을 채택하면 makeIterator() 함수를 필수로 구현해야 합니다.

아래 코드로 확인해 봅시다.

```swift
extension Bag: Sequence {
  func makeIterator() -> BagIterator<Element> {
    return BagIterator(store: store)
  }
}
```

위의 makeIterator() 함수의 구현부에서 BagIterator(store: store)를 리턴하는데 Bag 타입이 가진 store 프로퍼티의 복사본을 BagIterator로 전달하는 것입니다.

값 타입 store의 복사본을 전달하기 때문에 BagIterator에서 store 값을 변경하더라도 Bag 타입의 store 프로퍼티에는 영향을 미치지 않습니다.

이제 Bag 타입이 Sequence 프로토콜을 채택했기 때문에 Sequence 프로토콜이 제공하는 filter, lazy, reduce, contains 함수들을 Bag 타입 객체에 적용할 수 있습니다. 아래 코드는 Bag 타입에 Sequence 프로토콜에서 제공하는 함수를 적용한 코드입니다.

```swift
bag.filter { $0.count > 2 }
bag.lazy.filter { $0.count > 2 }
bag.contains("Huey")  // true
bag.contains("Mickey")  // false
```

지금까지는 Bag 타입이 Sequence 프로토콜을 채택하기 위해 BagIterator를 구현했습니다.

하지만 AnyIterator를 사용하면 BagIterator를 따로 구현하지 않고도 Bag 타입이 Sequence 프로토콜을 채택하고 makeIterator() 함수를 구현하도록 할 수 있습니다.

AnyIterator를 사용하면 Bag 타입에 Sequence 프로토콜을 채택하기 위해서 추가적으로 구현했던 BagIterator를 만들 필요가 없습니다.
Sequence 프로포콜을 채택하면 구현해야할 makeIterator() 함수에서 AnyIterator<Element>를 리턴하면 됩니다.
AnyIterator는 type erased iterator입니다.

아래 코드로 AnyIterator를 사용한 방식을 살펴봅시다.

```swift
extension Bag: Sequence {
  func makeIterator() -> AnyIterator<Element> {  // BagIterator 대신 AnyIterator<Element>을 사용합니다.
    var exhaustiveStore = store

    return AnyIterator<Element> {
      guard let (key, value) = exhaustiveStore.first else {
        return nil
      }
      if value > 1 {
        exhaustiveStore[key]? -= 1
      } else {
        exhaustiveStore[key] = nil
      }
    }
  }
}
```

BagIterator와 같이 cumstom iterator를 대신해 AnyIterator를 사용하여 custom iterator를 위한 코드를 줄일 수 있습니다.

Bag 객체를 정의할 때 array literal 구문과 비슷하게 정의하려 할 때가 있습니다.
ExpressibleByArrayLiteral 프로토콜을 통해 Bag 객체를 array literal 구문과 비슷하게 정의할 수 있습니다.

```swift
// array literal 구문과 비슷하게 Bag 객체 정의
let colors: Bag = ["Green", "Green", "Blue", "Yellow", "Yellow", "Yellow"]
```

위 코드와 같이 Bag 구조체에서 구현한 insert() 함수로 Bag에 요소를 넣는 것이 아닌 배열의 형태로 Bag 객체에 요소를 넣을 수 있습니다.

하지만 이런 배열의 형태로 Bag 구조체에 요소를 넣기 위해서는 ExpressibleByArrayLiteral 프로토콜을 채택해야 합니다.
ExpressibleByArrayLiteral 프로토콜을 통해 요소를 배열로 받는 생성자(init)를 제공받게 됩니다.

아래 코드로 확인해 봅시다.

```swift
extension Bag: ExpressibleByArrayLiteral {
  typealias ArrayLiteralElement = Element
  init(arrayLiteral element: Element...) {
    store = element.reduce(into: [Element: Int]()) { (updating, element) in
      updatingStore[element, default: 0] += 1
    }
  }
}

let colors: Bag = ["Green", "Green", "Blue", "Yellow", "Yellow", "Yellow"]
print(colors)
// Output:
// Green occurs 2 times
// Blue occurs 1 time
// Yellow occurs 3 times       
```

Sequence 프로토콜은 Collection 프로토콜의 기반이 되는 프로토콜입니다.
Collection 프로토콜은 Sequence 프로토콜의 서브 프로토콜입니다.

Collection 프로토콜을 살펴보기 전에 Sequence 프로토콜을 사용해 무한히 반복하는 iteration을 구현해 봅시다.
infinite(무한한) Sequence를 사용한 코드를 먼저 확인해 봅시다.

```swift
let infiniteSequence = InfiniteSequence(["a", "b", "c"])
for (index, letter) in zip(0..<100, infiniteSequence) {
  print("\(index): \(letter)")
}

// Output
// 0: a
// 1: b
// 2: c
// 3: d
// ...
// 97: b
// 98: c
// 99: a
```

zip 함수 성격상 두 iterator 중 하나라도 모든 element를 소비했을 때 zip 함수가 종료됩니다.
위 코드는 infiniteSequence의 iteratore가 종료될 일은 없기 때문에 0부터 100까지 반복되고 종료되는 zip 함수입니다.

그렇다면 무한한 sequence는 어떻게 구현할까요?

next() 함수에서 sequence의 요소가 다음으로 넘어가는 기준을 구현할 수 있기 때문에 IteratorProtocol의 next() 함수에서 무한한 iteration을 위한 조건을 구현해야 합니다.

아래 코드로 살펴봅시다.

```swift
struct infiniteSequence<S: Sequence>: Sequence, IteratorProtocol {
  let sequence: S
  var currentIterator: S.Iterator
  var isFinished: Bool = false

  init(_ sequence: S) {
    self.sequence = sequence
    self.currentIterator = sequence.makeIterator()
  }

  mutating func next() -> S.Element? {
    guard !isFinished else {
      return nil
    }

    if let element = currentIterator.next() {
      return element
    } else {
      self.currentIterator = sequence.makeIterator()
      let element = currentIterator.next()
      if element == nil {
        isFinished = true
      }
      return element
    }
  }
}

let infiniteSequence = InfiniteSequence(["a", "b", "c"])
for (index, letter) in zip(0..<100, infiniteSequence) {
  print("\(index): \(letter)")
}
```

다음으로 Collection 프로토콜에 대해 살펴봅시다.

## The Collection protocol

Collection 프로토콜은 Sequence 프로토콜의 서브 프로토콜입니다.
따라서 Collection 프로토콜을 채택하면 Sequence 프로토콜의 모든 함수를 제공합니다.

Collection 프로토콜과 Sequence 프로토콜은 몇 가지 차이가 있습니다.
첫 번째 차이는 Collection 프로토콜은 Sequence 프로토콜과 달리 인덱스를 제공합니다.
Collection 프로토콜이 인덱스를 제공하기 때문에 myarray[2]와 같이 인덱스를 활용해 요소에 접근할 수 있습니다.

아래 코드로 Collection 프로토콜을 따르는 String 타입에서 인덱스를 활용한 방법을 살펴봅시다.

```swift
let strayanAnimals = "kangaroo Koala"
if let middleIndex = strayanAnimals.index(of " ") {
  strayanAnimals.prefix(upto: middleIndex)  // Kangaroo
  strayanAnimals.suffix(from: strayanAnimals.index(after: middleIndex))  // Koala
}
```

두 번째 차이는 Collection 프로토콜에서 비파괴적인(nondestructive) 순회를 보장합니다. 여기서 비파괴적인 순회란 순회를 돌며 iteration의 요소를 소비하지 않는 것을 말합니다. Collection 프로토콜과 달리 Sequence 프로토콜은 순회를 돌며 iteration의 요소를 소비할 수도 있고 소비하지 않을 수 있습니다. 만약 순회를 요소를 소비하며 파괴적으로 돈다면 동일한 Sequence를 가지고 여러 번 순회하게 될 때 매번 결과가 달라지는 문제가 생길 수 있습니다.

아래 코드는 Sequence 프로토콜이 비파괴적인 순회를 보장하지 않아 발생하는 문제 상황입니다.

```swift
let numbers = // Let's say numbers is a Sequence but not a Collection
for number in numbers {
  if number == 10 {
    break
  }
}

for number in numbers {
  // Will iteration resume, or start from the beginning?
}
```

Sequence는 파괴적(요소를 소비)일 수도 있고 아닐 수 있어서 iteration이 중단되었다가 다시 동일한 Sequence의 iteration이 시작되면 중단된 지점부터 순회를 돌지 아니면 Sequence의 첫 요소부터 순회를 돌지 알 수 없는 문제가 있습니다.

Collection 프로토콜을 채택하여 비파괴적인 순회를 보장할 수 있습니다.
비파괴적인 순회는 동일한 iteration을 반복하더라도 동일한 결과를 리턴할 수 있습니다.

Collection 프로토콜을 따르는 대표적인 타입으로는 String, Array, Dictionary 그리고 Set이 있습니다.

Collection 프로토콜이 Sequence 프로토콜의 자식 프로토콜인 것처럼 Collection 프로토콜도 자식 프로토콜을 가지고 있습니다.
MutableCollection, RangeReplacableCollection, BidirectionalCollection 그리고 RandomAccessCollection 프로토콜들이 Collection 프로토콜의 자식 프로토콜입니다. (사실 RandomAccessCollection 프로토콜은 BidirectionalCollection 프로토콜의 자식 프로토콜입니다)
Collection 프로토콜의 네 가지 자식 프로토콜들을 차례로 살펴봅시다.

**MutableCollection**

MutableCollection offers methods that mutate elements in place without changing the length of a collection.

MutableCollection 프로토콜은 컬렉션의 길이 변화 없는 조건에서 요소들의 변경하도록 만들어줍니다.
컬렉션의 길이 변화가 없음을 보장하기 때문에 성능 측면에서도 이점이 있습니다.
대표적으로 Array 타입이 MutableCollection 프로토콜을 채택한 타입입니다.

MutableCollection 프로토콜이 제공하는 유용한 함수들이 있습니다.
물론 프로토콜이 요소의 변경을 목적으로 하므로 MutableCollection 프로토콜을 채택한 객체는 var로 선언되어야 합니다.

MutableCollection 프로토콜은 sort() 함수를 제공합니다.
아래 코드로 확인해 봅시다.

```swift
var mutableArray = [4, 3, 1, 2]
mutableArray.sort()  // [1, 2, 3, 4]
```

MutableCollection 프로토콜은 partition 함수도 제공합니다.
partition 함수는 배열을 두 덩어리로 나누고 나누는 기준을 커스텀할 수 있습니다.
또한 partition 함수의 리턴은 배열이 나뉘는 인덱스 값입니다.
아래 코드로 확인해 봅시다.

```swift
var arr = [1, 2, 3, 4, 5]
let index = arr.partition { (Int) -> Bool in
  return int & 2 == 0
}

print(arr)  // [1, 5, 3, 4, 2]
print(index)  // 3

arr[..<index]  // [1, 5, 3]
arr[index...]  // [4, 2]
```

한 가지 주의할 점이 String 타입은 MutableCollection 프로토콜을 따르고 있지 않습니다.
MutableCollection 프로토콜은 컬렉션의 길이를 변화시키지 않습니다.
하지만 String 타입은 컬렉션의 길이를 변경시킬 수 있기 때문입니다.

MutableCollection 프로토콜에서는 sort, partiton 함수 외에도 reverse, swapAt 등 여러 함수를 제공합니다.
MutableCollection 프로토콜에서 지원하는 함수들을 적극적으로 사용합시다.

**RangeReplacableCollection**

RangeReplaceableCollection allows you to swap out ranges and change its length.

RangeReplaceableCollection 프로토콜은 컬렉션의 변경을 지원하며 MutableCollection 프로토콜과 달리 컬렉션의 길이 변경을 허용합니다.
대표적으로  Array 타입과 String 타입이 RangeReplaceableCollection 프로토콜을 채택한 타입입니다.

RangeReplaceableCollection 프로토콜이 제공하는 '+=' 함수를 아래 코드로 살펴봅시다.

```swift
var muppets = ["Kermit", "Miss Piggy", "Fozzie bear"]

muppets += ["Statler", "Waldorf"]
print(muppets)  // ["Kermit", "Miss Piggy", "Fozzie bear", "Statler", "Waldorf"]

muppets.removeFrist()  // "kermit"
print(muppets)  // ["Miss Piggy", "Fozzie bear", "Statler", "Waldorf"]

muppets.removeSubrange(0..<2)
print(muppets) // ["Statler", "Waldorf"]
```

String 타입이 RangeReplaceableCollection 프로토콜을 따르기 때문에 아래와 같이 String의 길이를 변경할 수 있습니다.

```swift
var matrix = "The Matrix"
matrix += " Reloaded"
print(matrix)  // The Matrix Reloaded
```

또한 RangeReplaceableCollection 프로토콜에서는 removeAll 함수르 제공합니다.
배열과 같은 컬렉션에서 특정 요소를 삭제할 때 filter를 통해 요소를 삭제하기보다 removeAll 함수를 사용하는 것이 효과적입니다.
아래 코드로 removeAll 함수의 사용을 살펴봅시다.

```swift
var healthyFood = ["Donut", "Lettuce", "Kiwi", "Grapes"]
healthyFood.removeAll(where: { $0 == "Donut" })
print(healthyFood)  // ["Lettuce", "Kiwi", "Grapes"]
```

**BidirectionalCollection**

With a BidirectionalCollection you can traverse a collection backwards from an index.

BidirectionalCollection 프로토콜은 컬렉션의 양방향 (forward, backward) 순회를 지원합니다.
BidirectionalCollection 프로토콜이 제공하는 reversed() 함수를 아래 코드로 확인해 봅시다.

```swift
var letters = "abcd"
for value in letters.reversed() {
  print(value)
}
```

**RandomAccessCollection**

The RandomAccessCollection inherits from BidirectionCollection and offers some performance improvements for its methods.

RandomAccessCollection 프로토콜은 컬렉션에 효율적인 random-access index 순회를 지원합니다.
RandomAccessCollection 프로토콜은 인덱스를 원하는 거리로 이동할 수 있으며 O(1) 시간 내에 인덱스 간의 거리를 측정할 수 있습니다
위에서 살펴보았던 BidirectionalCollection 프로토콜은 전체 collection을 순회해서 카운팅 해야 하므로 O(1)의 시간복잡도를 가질 수 없습니다.
하지만 RandomAccessCollection 프로토콜은 임의의 인덱스 접근에 O(1)의 시간복잡도를 가집니다.

Array 타입은 RandomAccessCollection 프로토콜을 포함하여 모든 collection 프로토콜을 채택합니다.
또한 Repeated type도 RandomAccessCollection 프로토콜을 채택하고 있습니다.

Repeated type은 RandomAccessCollection 프로토콜을 따르는 타입으로 값을 여러 번 낱낱이 계산할 때 편리합니다.
repeatElement 함수를 사용해서 Repeated type을 얻을 수 있습니다.
아래 코드로 확인해 봅시다.

```swift
for element in repeatElement("Broken record", count: 3) {
  print(element)
}

// Output:
// Broken record
// Broken record
// Broken record

zip(repeatElement("Mr. Sniffles", count: 3), repeatElement(100, count:3)).forEach { name, index in
  print("Generated \(name) \(index)")
}

// Output
// Generated Mr. Sniffles 100
// Generated Mr. Sniffles 100
// Generated Mr. Sniffles 100
```

## Creating a collection

개발을 하다 보면 Collection 프로토콜을 따르는 Set, Array 등 스위프트에서 제공하는 데이터 타입을 자주 활용합니다.

하지만 Collection 프로토콜을 따르는 커스텀 타입을 직접 만들 수 있습니다.
지금부터 TravelPlan이라 불리는 Collection 프로토콜을 따르는 데이터 타입을 구현해 봅시다.

TravelPlan은 여러 활동이 담긴 하루(day)들의 Sequence입니다.
예를 들어 Florida에 방문하는 TravelPlan이라면 하루 일정에 밥을 먹고 박물관에 가는 등 여러 활동이 포함될 수 있습니다.

또한 TravelPlan 데이터 타입에서 iteration이 가능하고 idexing도 지원하기 위해서는 Collection 프로토콜을 채택해야 합니다.

먼저 TravelPlane의 일부로 쓰일 Activity 데이터와 Day 데이터를 구현해 봅시다.

```swift
struct Activity: Equatable {
  let date: Date
  let description: String
}

struct Day: Hashable {
  let date: Date

  init(date: Date) {
    let unitFlags: Set<Calendar.Component> = [.day, .month, .year]
    let components = Calendar.current.dateComponents(unitFlags, from: date)
    guard let convertedDate = Calendar.current.date(from: components) else {
      self.date = date
      return
    }
    self.date = convertedDate
  }
}
```

TravelPlan 데이터 타입에서 Day를 딕셔너리의 key로 사용하기 위해 Day 데이터 타입에 Hashable 프로토콜을 채택했습니다.
TravelPlan에서 DataType이라 불리는 타입을 key로 Day 타입을 value로 [Activity] 타입을 가진 딕셔너리로 만들었습니다.

아래 코드는 TravelPlan을 구현한 코드입니다.

```swift
struct TravelPlan {
  typealias DataType = [Day: [Activity]]

  private var trips = DataType()

  init(activities: [Activity]) {
    self.trips = Dictionary(grouping: activites) { activity -> Day in
      Day(date: activity.date)
    }
  }
}
```

아직 TravelPlan 데이터 타입을 가지고 iteration이나 indexing을 사용할 수 없습니다.

Collection 프로토콜을 채택하여 iteration과 indexing을 제공하려 합니다.

그전에 Collection 프로토콜의 정의부를 확인하고 커스텀 타입이 Collection 프로토콜을 채택했을 때 필수로 구현해야 할 부분을 살펴봅시다.

```swift
protocol Collection: Sequence {
    associatedtype Index: Comparable

    var startIndex: Index { get }
    var endIndex: Index { get }
    subscript(position: Index) -> Element { get }
    func index(after i: Index) -> Index
}
```

Collection 프로토콜을 커스텀 타입에 채택하기 위해서는 위 코드에 보이는 startIndex와 endIndex 변수를 구현해야 하고 index(after:) 함수와 subscript(index:) 함수를 필수로 구현해야 합니다.

아래 코드는 TravelPlan 타입에 Collection 프로토콜을 채택하고 필수 구현 변수와 함수들을 구현한 코드입니다.

```swift
extension TravelPlan: Collection {
  typealias KeysIndex = DataType.Index
  typealias DataElement = DataType.Element

  var startIndex: KeysIndex { return trips.keys.startIndex }
  var endIndex: KeysIndex { return trips.keys.endIndex }

  func index(after i: KeysIndex) -> KeysIndex {
    return trips.index(after: i)
  }

  subscript(index: KeysIndex) -> DataElement {
    return trips[index]
  }
}
```

이제 TravelPlan 타입이 Collection 프로토콜까지 채택했기 때문에 iteration과 indexing 모두 가능합니다.

또한 Collection 프로토콜에서 제공하는 makeIterator() 함수를 통해 IndexingIterator를 생성할 수 있습니다.
다시 말해 TravelPlan 타입의 커스텀 Iterator 없이 TravelPlan 타입을 순회할 IndexingIterator를 얻을 수 있습니다.

```swift
// A default Iterator
let defaultIterator: IndexingIterator<TravelPlan> = travelPlan.makeIterator()

// TravelPlan iteration
for (day, activities) in travelPlan {
  print(day)
  print(activities)
}
```

Collection 프로토콜을 따르면 subscript 능력을 갖추게 됩니다.

[] 대괄호 안에 index를 넣어줘서 멤버 요소에 접근하는 것이 subscript입니다.
배열의 서브스크립트는 parameter로 Int형을 index로 받고, 해당 index에 해당하는 Element를 반환하는 형태입니다.
또한 딕셔너리의 서브스크립트는 Prameter로 Key를 받고, 해당 Key에 해당하는 Value를 반환하는 형태입니다.

그렇다면 커스텀 서브스크립트를 TravelPlan 타입에 구현해 봅시다.

아래 코드를 확인해 봅시다.

```swift
extension TravelPlane {
  subscript(date: Date) -> [Activity] {
    return trips[Day(date: date)] ?? []
  }

  subscript(day: Day) -> [Activity] {
    return trips[day] ?? []
  }
}
// Now, you can access contents via convenient subscripts.

travelPlan[Date()]

let day = Day(date: Date())
travelPlan[day]
```

위 코드와 같이 subscript 함수를 구현하여 서브스크립트 또한 커스텀 가능합니다.

아래 링크도 참고해 봅시다.

https://babbab2.tistory.com/123

마지막으로 ExpressibleByDictionaryLiteral 프로토콜을 살펴보겠습니다.

앞에서 보았던 ExpressibleByArrayLiteral 프로토콜과 비슷한 개념의 프로토콜입니다.
ExpressibleByDictionaryLiteral 프로토콜을 채택하면 생성자에 여러 요소를 넣을 수 있도록 합니다.

아래 코드로 살펴봅시다.

```swift
extension TravelPlan: ExpressibleByDictionaryLiteral {
  init(dictionaryLiteral element: (Day, [Activity])...) {
    self.trips = Dictionary(elements, uniquingKeysWith: { (first: Day, ) in
      return true
    })
  }
}

let adrenalineTrip = Day(date: Date())
let adrenalineActivities = [
  Activity(date: Date(), description: "Bungee jumping"),
  Activity(date: Date(), description: "Driving in rush hour LA"),
  Activity(date: Date(), description: "Sky diving")
]

let adrenalinePlan = [adrenalineTrip: activites]  // You can now create a TravelPlan from a dictionary!!
```

그렇다면 아래 Fruits 타입에 Collection 프로토콜을 채택해 봅시다. (연습문제)

```swift
struct Fruits {
  let banana = "Banana"
  let apple = "Apple"
  let tomato = "Tomato"
}

extension Fruits: Collection {
  var startIndex: Int {
    return 0
  }

  var endIndex: Int {
    return 3  // Yep, it's 3, not 2. That's how Collection wants it.
  }

  func index(after i: Int) -> Int {
    return i + 1
  }

  subscript(index: Int) -> String {
    switch index {
    case 0: return banana
    case 1: return apple
    case 2: return tomato
    default: fatalError("The fruits end here.")
    }
  }
}

let fruits = Fruits()
fruits.forEach { (fruit) in
  print(fruit)
}
```

아래 링크는 Sequence, Collection 등 잘 정리된 글이니 참고해 봅시다.

https://itwenty.me/posts/04-swift-collections/

## Summary
- Iterators produce elements.
- To iterate, Swift uses a while loop on makeIterator() under the covers.
- Sequences produce iterators, allowing them to be iterated over repeatedly.
- Sequences won't guarantee the same values when iterated over multiple times.
- Seqence is the backbone for methods such as filter, reduce, map, zip, repeat, and many others.
- Collection inherits from Sequence.
- Collection is a protocol that adds subscript capabilities and guarantedd nondestructive iteration.
- Collection has subprotocols, which are more-specialized versions of Collection.
- MutableCollection is a protocol that restricts collections for easy modification of part of a collection. As a result, the length of a collection may change. It also offers useful methods, such as removeAll(where:).
- BidirectionalCollection is a protocol that defines a collection that can be traversed both forward and backward.
- RandomAccessCollection restricts collections to constant-time traversal between indices.
- You can implement Collection for regular types that you use in day-to-day programming.
