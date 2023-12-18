# Putting the pro in protocol-oriented programming

## This chapter covers
- The relationship and trade-offs between generics and using protocols as types
- Understanding associated types
- Passing around protocols with associated types
- Storing and constraining protocols with associated types
- Simplifying your API with protocol inheritance

## Runtime versus compile time
프로토콜은 크게 두 분류로 나뉩니다.
제네릭을 제약할 때의 프로토콜과 단일 타입으로 쓰이는 프로토콜로 나뉩니다.
제네릭을 제약할 때의 프로토콜은 컴파일 타임에 타입을 결정하고 단일 타입으로 쓰이는 프로토콜의 경우 런타임에 타입을 결정합니다.

지금부터 코인 포토폴리오를 구현하며 살펴봅시다.
요구 사항으로 다양한 코인에 대응하는 포토폴리오를 구현해야 합니다.

코인 포토폴리오를 구현하기 위해 열거형을 떠올릴 수 있습니다.
하지만 모든 코인에 대응해야하기 때문에 너무 많은 case들이 나오기 때문에 좋은 선택지는 아닙니다.

이때 우리는 프로토콜을 통해 구현하여 코인 포토폴리오를 추상화 할 수 있습니다.
아래 코드는 프로토콜 코드입니다.

```swift
import Foundation

protocol CryptoCurrency {
  var name: String { get }
  var symbol: String { get }
  var holdings: Double { get set }
  var price: NSDecimalNumber? { get set }
} 
```
