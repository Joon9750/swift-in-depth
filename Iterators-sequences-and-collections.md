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
IteratorProtocol의 이름만 봐도 알 수 있듯이 데이터 순회에 사용되는 Iterator는 
















