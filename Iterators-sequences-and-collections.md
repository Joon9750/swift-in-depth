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
또한 lazy iteration은 해당 lazy 변수에 접근할 때 계산이 이루어집니다.

아래 코드로 lazy iteration을 사용하는 상황을 살펴봅시다.

```swift
func increment(x: Int) -> Int {
  print("Cumputing next value of \(x)")
  return x+1
}

let array = Array(0..<10)
let incArray = array.map(increment)
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

위 코드를 사용하면 incArray 값에 접근하기 전에 모든 출력 값이 계산됩니다.
만약 increment가 10이 아니라 더 큰 값을 가진다면 interation에 큰 비용이 들게 됩니다.

이런 상황에서 lazy 키워드를 사용해 성능을 향상시킬 수 있습니다.

```swift
func increment(x: Int) -> Int {
  print("Cumputing next value of \(x)")
  return x+1
}

let array = Array(0..<10000)
let incArray = array.lazy.map(increment)
print("Result: ")
print(incArray[0], incArray[4])

// Output:
Result: 
Computing next value of 0
Computing next value of 4
1 5
```

lazy 키워드를 사용하면 아래 코드를 만나도 계산하지 않습니다.

```swift
let incArray = array.lazy.map(increment)
```

lazy 키워드 성격 그대로 incArray에 접근할 때 계산을 시작합니다.
또한 앞서 말했듯이 sequence의 모든 element에 접근하지 않고 일부 element에만 접근하여 성능을 향상시킬 수 있습니다.

아래 링크에서 추가적인 설명을 볼 수 있습니다.

https://zeddios.tistory.com/387

**reduce**

reduce 함수는 sequence의 element를 순회하며 특정 값을 누적하고, reduce 함수가 끝날 때 누적된 값을 리턴합니다.
다시 말해 reduce 함수로 여러 elements를 새로운 값으로 변형할 수 있습니다.

아래는 reduce 함수의 정의부입니다.

```swift
func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) throws -> Result) rethrows -> Result
```

먼저 제네릭 함수임을 알 수 있습니다.

reduce 함수의 매개변수로 initialResult과 nextPartialResult를 받고 있습니다.
initialResult은 초기값으로 사용할 값을 넣으면 클로저가 처음 실행될 때, nextPartialResult 에 전달됩니다.
그리고 nextPartialResult은 컨테이너의 요소를 새로운 누적값으로 결합하는 클로저입니다.

두 번째 파라미터로 전달된 클로저를 좀 더 자세히 보겠습니다.
클로저의 첫번째 파라미터는 initialResult로 전달받은 초기값 또는 이전 클로저가 반환하는 return값이 전달됩니다.
클로저의 두 번째 파라미터는 reduce(_:_:)를 호출한 컨테이너의 요소가 전달됩니다.
클로저가 반환하는 값은 파라미터로 받은 두 값을 적절하게 처리하여 첫 번째 파라미터로 받은 값과 같은 타입의 값을 리턴해줍니다.

reduce 함수의 리턴으로는 최종 누적 값이 반환되며, 컨테이너의 요소가 없다면 initialResult 의 값이 반환됩니다.

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

먼저 reduce 함수와 reduce(into:) 함수의 정의부를 모두 확인해봅시다.

```swift
func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) throws -> Result) rethrows -> Result
func reduce<Result>(into initialResult: Result, _ updateAccumulatingResult: (inout Result, Element) throws -> ()) rethrows -> Result
```

reduce 함수는 두 가지 형태로 사용됩니다.
위에서 보았듯이 기본적인 reduce 함수 그리고 reduce(into:) 함수가 있습니다.
reduce 함수의 초기 값으로 값 타입이 들어온다면 reduce(into:) 함수를 사용합시다.

reduce 함수의 두번째 매개변수인 클로저의 Result는 immutable합니다.
따라서 reduce 함수에서는 Array, Dictionary 등 값 타입을 직접 수정할 수 없기 때문에 새로운 변수로 copy 한 다음에 
그 값을 변경하고 사용해야 하는 번거로움이 있습니다.

reduce 함수로 값 타입 변수를 초기 값으로 받았을 때 immutable한 reduce의 두 번째 Result을 mutable한 변수로 복사해야합니다.
아래 코드로 살펴봅시다.

```swift
let grades = [3.2, 4.2, 2.6, 4.1]
let results = grades.reduce([:]) { (results: [Character: Int], grade: Double) in
  var copy = results
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

그에 반해 reduce(into:) 함수는 reduce 함수와 첫 번째 초기 값을 받는 매개변수는 동일하지만, 두 번째 매개변수에 inout 키워드가 
추가되었고 리턴 값이 사라졌습니다.
리턴 값 없이 inout 키워드가 붙은 Result 값을 변경하며 최종 값에 도달합니다.

reduce(into:_:)의 가장 큰 차이는 클로저의 첫 번째 파라미터가 inout 파라미터라는 점입니다.
즉, initialResult로 전달받은 값을 클로저안에서 직접 변경 가능하고 두 번째 클로저의 Result 또한 변경 가능합니다.
reduce(_:_:)은 initialResult로 전달 받은 값을 클로저에서 변경할 수 없습니다.(let 변수)

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

배열이나 딕셔너리와 같은 값 타입과 reduce 함수를 사용할 때는 into 키워드를 함께 사용해 성능을 높입시다.

reduce 함수 사용을 추천하는 이유는 명백합니다.
for loop을 사용할 경우 for 구문 안의 코드를 읽어야 for 구문의 목적을 알 수 있습니다.
하지만 reduce 함수를 사용하면 그 자체로 sequence를 순회하며 새로운 누적 값을 생성한다는 사실을 알 수 있습니다.
또한 간결한 코드를 구현할 수 있습니다.

아래 링크로 reduce와 reduce(into:) 함수의 더 많은 예시 코드를 확인해봅시다.

https://dejavuqa.tistory.com/181, 
https://beepeach.tistory.com/606,
http://yoonbumtae.com/?p=4390

**zip**

마지막으로 살펴볼 함수는 zip입니다. 
zip으로 두 iterator를 합칠 수 있습니다.
둘 중 하나라도 iterator가 고갈되면 zip에서의 interation은 종료됩니다.

아래 코드를 살펴봅시다.

```swift
for (integer, string) in zip(0..<10, ["a", "b", "c"]) {
  print("\(integer): \(string)")
}
```

## Creating a generic data structure with Sequence

이제는 Sequence 프로토콜을 따르는 커스텀 타입을 만들어 봅시다.

자료구조 중 'Bag' 자료구조를 직접 구현해볼 생각입니다.
Bag을 만들 때, Sequence 프로토콜과 IteratorProtocol을 사용해 Bag 자료구조에 iteration을 제공하고 다양한 함수를 제공하려 합니다.

Bag은 Set과 비슷하게 정렬되지 않는 상태로 존재하며 요소를 추가하고 조회할 수 있습니다.
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
Bag 자료구조에서는 동일한 요소일 경우 실제 객체로 동일한 요소를 중복해서 저장하지 않고 객체의 counter를 갱신하여 최적화합니다.
Bag의 element 각각에 counter가 존재하고 counter가 0이 되었을 때 해당 element는 삭제됩니다.

중복되는 객체를 Bag에 저장하기보다 객체의 counter를 통해 중복되는 객체를 관리했을 때 메모리 사용량 측면에서 이점이 있습니다.

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

Bag의 저장되는 요소들을 관리하는 store 변수를 딕셔너리로 만들어 Element에 대응하는 Int가 Element의 개수를 관리는 counter의 역할을 합니다.
또한 store 변수가 딕셔너리로 값 타입이기 때문에 store 변수의 값을 다루는 함수인 insert와 remove는 mutating 키워드를 붙여야 합니다.

우리 Bag 타입을 출력할 때 출력 형식을 커스텀하고 싶을 때가 있습니다.
이때 CustomStringConvertible 프로토콜을 Bag 타입에 채택하여 출력(print) 형식을 커스텀할 수 있습니다.
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

CustomeStringConvertible 프로토콜의 description 프로퍼티를 커스텀하면 Bag 타입을 출력(print)할 때 해당 description에 맞춰 출력됩니다.

지금까지 Bag 타입을 구현했습니다.
하지만 Sequence 프로토콜을 채택하지 않은 상태의 Bag 타입에서 iteration을 진행할 수 없습니다.
Sequence 프로토콜을 Bag 타입에 채택하기 위해서는 Bag 타입을 위한 iterator가 필요합니다.

















