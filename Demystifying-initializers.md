# Demystifying initializers

## This chpater overs
- Demystifying Swift's initializer rules
- Understanding quirks of struct initializers
- Understanding complex initializer rules when subclassing
- How to keep the number of initializers low when subclassing
- When and how to work with required initializers

## Struct initializer rules
구조체는 subclassing을 지원하지 않기 때문에 생성자가 상대적으로 정직합니다.
그럼에도 특별한 규칙이 몇 가지 있습니다.

기본적으로 스위프트에서는 구조체와 클래스의 모든 프로퍼티가 초기화 시키는 것을 원칙으로 여깁니다.
따라서 구조체와 클래스의 프로퍼티가 초기화되지 않았다면 컴파일되지 않습니다.

구조체는 아래 코드와 같이 생성자를 따로 구현하지 않았다면 모든 프로퍼티를 초기화하는 default init을 제공합니다.
default init은 코드에는 보이진 않지만 객체 생성 시 사용할 수 있습니다.

```swift
enum Pawn {
  case dog, cat, ketchupBottle, iron, shoe, hat
}

struct Player {
  let name: String
  let pawn: Pawn
}

let player = Player(name: "June", pawn = .shoe)
```

만약 구조체에 custom init을 만들었다면 memberwise init(모든 프로퍼티를 초기화하는 default init)은 호출되지 않습니다.
custom init을 만들었다고 memeberwise init을 지원하지 않는다는 사실에 굳이 왜 지원하지 않을까? 라는 생각을 할 수 있습니다.
하지만 이는 memeberwise init으로 custom init을 우회하는 행위를 사전에 방지하기 위함입니다. 안정성 측면에서 훌륭한 방식입니다.

물론 memeberwise init을 제공 받는 동시에 custom init을 구현하는 방법도 존재합니다.
extension을 사용해 custom init의 구현을 extension에 한다면 memeberwise init을 제공 받을 수 있습니다.
본인이 놓인 상황에 어울리는 방법을 선택해 사용합시다.

아래 코드는 extension을 통해 memeberwise init을 제공 받으며 custom init을 구현할 코드입니다.

```swift
struct Player {
  let name: String
  let pawn: Pawn
}

extensioin Player {
  init(name: String) {
  }
}
```





















