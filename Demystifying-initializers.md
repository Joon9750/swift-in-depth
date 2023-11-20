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

아래 코드는 extension을 통해 memeberwise init을 제공 받으며 custom init을 구현한 코드입니다.

```swift
struct Player {
  let name: String
  let pawn: Pawn
}

extension Player {
  init(name: String) {
    self.name = name
    self.pawn = Pawn.allCases.randomElement()!
  }
}

let player = Player(name: "June", pawn = .shoe)
let anotherPlayer = Player(name: "Hong")
```

위 코드를 읽으면 한 가지 궁금증이 생길것 입니다.
분명 Pawn은 enum인데 어떻게 allCases.randomElement()가 가능할까요!
enum Pawn이 CaseIterable protocol을 채택하여 가능한 일입니다.

CaseIterable protocol은 associated values가 없는 enum에서 사용 가능합니다.
enum에 포함된 모든 case를 allCases 타입 프로퍼티를 통해 컬렉션을 생성합니다.
enum case의 컬렉션으로 만들어진 allCases 프로퍼티는 컬렉션이기 때문에 .randomElement() 함수도 사용 가능하게 됩니다.

단, associated values(연관 값)를 가진 enum의 경우 CaseIterable protocol을 채택하여 allCases를 생성할 수 없습니다.
연관 값을 가진 enum은 무한한 변화가 가능하기 때문입니다.

아래 코드를 확인해 봅시다.
```swift
enum Pawn: CaseIterable {
  case dog, cat, ketchupBottle, iron, shoe, hat
}

let allCases: [Pawn] = Pawn.allCases
print(allCases)  // [Pawn.dog, Pawn.cat, Pawn.ketchupBottle, Pawn.iron, Pawn.shoe, Pawn.hat]
```

## Initializers and subclassing
Subclassing isn't too popular in the Swift community, especially because Swift often is marketed as a protocol-oriented language, which is one subclassing alternative.
Nevertheless, subclassing is still a valid tool that Swift offers.

지금부터 init이 class와 subclassing에 어떻게 동작하는지 살펴봅시다.

먼저 초기화는 Designated init과 convenience init으로 두 가지가 있습니다.
그리고 초기화는 본인의 모든 프로퍼티를 초기화함과 상속받은 부모 클래스의 프로퍼티들을 customizing 할 목적을 가집니다.

Designated init(지정 초기자)은 모든 프로퍼티를 초기화하며 클래스 타입에 반드시 한 개 이상의 지정 초기자가 필요합니다.

convenience init(편의 초기자)은 













