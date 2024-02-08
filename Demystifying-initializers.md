# Demystifying initializers

## This chpater overs
- Demystifying Swift's initializer rules
- Understanding quirks of struct initializers
- Understanding complex initializer rules when subclassing
- How to keep the number of initializers low when subclassing
- When and how to work with required initializers

## Struct initializer rules
구조체는 subclassing을 지원하지 않기 때문에 상대적으로 정직한 초기화 규칙을 가집니다.
그럼에도 특별한 규칙이 몇 가지 있습니다.

기본적으로 스위프트에서는 구조체와 클래스의 모든 프로퍼티가 초기화 시키는 것을 원칙으로 여깁니다.
따라서 구조체와 클래스의 프로퍼티가 초기화되지 않았다면 컴파일 단계에서 에러가 발생합니다.

구조체는 아래 코드와 같이 생성자를 따로 구현하지 않았다면 모든 프로퍼티를 초기화하는 memberwise init(default init)을 제공합니다.
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
custom init을 만들었다고 memeberwise init을 왜 굳이 지원하지 않을까? 라는 생각을 할 수 있습니다.
하지만 이는 memeberwise init으로 custom init의 초기화 동작을 우회하는 행위를 방지하기 위함입니다. 이는 안정성 측면에서 훌륭한 방식입니다.

물론 memeberwise init을 제공 받는 동시에 custom init을 구현하는 방법도 존재합니다.
extension을 사용해 custom init을 extension에 구현한다면 custom init과 함께 memeberwise init을 제공 받을 수 있습니다.
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

let player = Player(name: "June", pawn = .shoe)  // memeberwise init
let anotherPlayer = Player(name: "Hong")  // custom init
```

위 코드를 읽으면 한 가지 궁금증이 생깁니다.

분명 위에서 Pawn은 열거형인데 어떻게 열거형에 allCases.randomElement()가 가능할까요!
enum Pawn이 CaseIterable protocol을 채택하여 가능한 일입니다.

CaseIterable protocol은 associated values가 없는 enum에서 사용 가능합니다.
CaseIterable protocol은 열거형의 case들을 배열로 사용하도록 만들어 줍니다.
열거형에 포함된 모든 case를 allCases 타입 프로퍼티를 통해 컬렉션을 생성합니다.
enum case의 컬렉션으로 만들어진 allCases 프로퍼티는 컬렉션이기 때문에 .randomElement() 함수도 사용 가능하게 됩니다.

단, associated values(연관 값)를 가진 enum의 경우 CaseIterable protocol을 채택하여 allCases를 생성할 수 없습니다.
연관 값을 가진 enum은 무한한 변화가 가능하기 때문입니다.

아래 링크으로 더 자세히 확인합시다.

[Apple Developer Documentation](https://developer.apple.com/documentation/swift/caseiterable)

아래 코드로 열거형에 적용된 CaseIterable 프로토콜을 확인해 봅시다.

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

먼저 초기자에는 Designated init과 convenience init 두 가지가 있습니다.
그리고 초기화는 본인의 모든 프로퍼티를 초기화함과 상속받은 부모 클래스의 프로퍼티들을 커스터마이징 할 목적을 가집니다.

Designated init(지정 초기자)은 모든 프로퍼티를 초기화해야 하며 클래스 타입에는 반드시 한 개 이상의 Designated init이 필요합니다.
부모 클래스가 있을 경우 자식 클래스는 부모 클래스의 Designated init을 상속받습니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/3d2b333a-f23e-4d94-b264-e71e9674adb1)

convenience init(편의 초기자)은 초기자 내부에서 마지막에는 반드시 Designated init를 호출해야 합니다. (self.init())
convenience init를 통해 default value를 제공하거나 단순화한 init을 제공할 수 있고 다른 convenience init를 호출할 수 있습니다.

아래 코드는 Designated init과 convenience init을 사용한 예시 코드입니다.

```swift
class BoardGame {
  let players: [Player]
  let numberOfTiles: Int

  // Designated init
  init(players: [Player], numberOfTiles: Int) {
    self.players = players
    self.numberOfTiles = numberOfTiles
  }

  covenience init(players: [Player]) {
    self.init(players: players, numberOfTiles: 32)  // 여기서 default value를 제공합니다.
  }

  convenience init(names: [String]) {
    var players = [Player]()
    for name in names {
      players.append(Player(name: name))
    }
    self.init(players: players, numberOfTiles: 32)  // 마지막에는 반드시 Designated init을 호출하고 있습니다.
  }
}

let boardGame = BoardGame(names: ["hong", "kim", "kang"])

let players = [
  Player(name: "hong"),
  Player(name: "kim"),
  Player(name: "kang")
]
let boardGame = BoardGame(players: players)

let boardGame = BoardGame(players: players, numberOfTiles: 32) 
```

클래스는 구조체와 달리 memberwise init(default init)을 제공하지 않습니다.

지금부터 클래스를 subclassing 할 때를 살펴봅시다.

기본적으로 자식 클래스는 부모 클래스의 모든 init(Designated init & covenience init)을 상속받습니다.
따라서 자식 클래스도 부모 클래스와 동일한 방식으로 초기화 가능합니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/a951fed1-8a32-4b25-a941-d5106ea30779)

하지만 자식 클래스에서 부모 클래스에 없는 새로운 프로퍼티를 추가했을 때는 다릅니다.

자식 클래스에 새로운 프로퍼티가 추가되면 자식 클래스에 있던 부모 클래스의 모든 init은 사라집니다.
위에서 말했듯이 클래스의 모든 프로퍼티는 초기화되어야 합니다. 하지만 부모 클래스에 없는 프로퍼티가 자식 클래스에 추가되며 더 이상 부모 클래스 init으로 자식 클래스의 프로퍼티를 초기화할 수 없어지게 됩니다.

이때 우린 자식 클래스의 모든 프로퍼티를 초기화하는 새로운 designated init을 구현해야 합니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/cfa77de4-1a32-46b9-ba0e-26a801ba32c3)

자식 클래스에 초기화 되어야 하는 새로운 프로퍼티가 추가되면 자식 클래스에 새로운 designated init을 구현하고 자식 클래스의 designated init에서 부모 클래스의 designated init을 호출하는 방식으로
프로퍼티를 초기화해야 합니다.

아래 코드로 확인해 봅시다.

```swift
class MutabilityLand: BoardGame {
  var scoreBoard = [String: Int]()
  var winner: Player?

  // 자식 클래스에 추가된 초기화 되어야 하는 새로운 프로퍼티
  let instructions: String

  // 자식 클래스의 designated init
  init(players: [Player], instructions: String, numberOfTiles: Int) {
    self.instructions = instructions
    super.init(players: players, numberOfTiles: numberOfTiles)
  }
}
```

위 코드에서 scoreBoard와 winner는 초기화될 필요 없는 프로퍼티입니다. (winner는 옵셔널 프로퍼티로 초기화 의무가 없습니다.)
따라서 자식 클래스에 scoreBoard와 winner 프로퍼티만 추가 되었다면 부모 클래스의 모든 init을 상속받을 수 있습니다.

하지만 MutabilityLand 클래스의 instructions 프로퍼티는 초기화 되어야 하는 새로운 프로퍼티입니다.
instructions가 추가되며 MutabilityLane 클래스는 부모 클래스인 BoardGame 클래스의 모든 init을 상속받지 못합니다.

따라서 위와 같이 자식 클래스에 새로운 designated init을 만들어 초기화 되어야 하는 새로운 프로퍼티를 초기화 이후 super.init을 통해 부모 클래스로 부터 상속받은 프로퍼티를 초기화해야 합니다.
여기서 주의해야할 점은 super.init은 부모 클래스의 designated init을 자식 클래스에서 호출할 뿐이지 부모 클래스의 init을 상속받은건 아닙니다.

하지만 자식 클래스에 초기화될 필요가 있는 새로운 프로퍼티가 생겨도 부모 클래스의 init을 상속받는 방법이 있습니다!

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/191ffa6d-109d-44a6-a47c-f3e58bcb763b)

자식 클래스에서 새로운 designated init을 만들지 않고, 부모 클래스의 designated init을 override하면 자식 클래스는 부모 클래스의 모든 init을 상속받을 수 있습니다.
물론 override한 부모 클래스의 designated init에서는 자식 클래스의 새로운 프로퍼티를 반드시 초기화해야 합니다.

아래 코드를 살펴봅시다.

```swift
class Mutability: BoardGame { 
  // 생략
  // 부모 클래스의 designated init을 자식 클래스에서 override
  override init(players: [Player], numberOfTiles: Int) {
    self.instructions = "Read the manual"  // super.init 호출 전 새로운 프로퍼티 초기화합니다.
    super.init(players, numberOfTiles: numberOfTiles)  // 필수입니다.
  }
}
```

부모 클래스의 designated init을 자식 클래스에서 override 할 때 주의할 점은 super.init으로 부모 클래스로부터 상속받는 프로퍼티를 초기화하기 전에 자식 클래스에 새로 추가된 프로퍼티를 먼저 초기화해야 합니다.

하지만 designated init을 override을 하면 클래스 계층이 깊어질 수 있습니다.

In a class hierarchy, convenience init go horizontal, and designated init go vertical!

## Minimizing class initializers

위에서 보았듯이 subclassing과 함께 자식 클래스에 초기화가 필요한 프로퍼티가 추가되었을 때 부모 클래스의 init을 상속받기 위해 designated init을 상속했습니다.
하지만 이 동작이 반복될수록 자식 클래스의 designated init이 한 개씩 늘어나게 됩니다.

만약 부모 클래스의 designated init 한 개와 convenience init 두 개를 가질 때 자식 클래스에서 부모 클래스의 designated init을 override한다고 가정해봅시다.
결과적으로 자식 클래스가 갖게 되는 init은 부모 클래스에서 상속받는 convenience init 두 개와 override designated init 그리고 자식 클래스 본인만의 (반드시 가져야하는) designated init이 있습니다.

만약 subclassing 될 때마다 위의 동작이 반복되면 자식 클래스는 여러 designated init을 가지게 될것입니다.
(override designated init도 designated init입니다.) 이는 계층을 복잡하게 만들 수 있습니다.

자식 클래스에 초기화가 필요한 프로퍼티가 추가되어도 부모 클래스의 init을 모두 상속받으며 designated init을 한 개로 유지하는 방법을 살펴봅시다.

designated init을 override 할 때 convenience init으로 부모 클래스의 designated init을 override한다면 자식 클래스의 designated init을 한 개로 유지할 수 있습니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/faef01e3-ddcc-4c7f-a69c-01b7f1ffcd60)

여기서 부모 클래스의 designated init을 자식 클래스의 convenience init으로 override 할 때 자식 클래스의 convenience override init에서 자식 클래스의 designated init을 호출하고 designated init에서 부모 클래스의 designated init을 
호출하도록 구현하면 부모 클래스의 모든 init을 상속 받으며 하위 자식 클래스의 designated init을 한 개로 유지할 수 있습니다.

아래 코드로 확인해봅시다.

```swift
class MutabilityLand: BoardGame {
  var scoreBoard = [String: Int]()
  var winner: Player?

  let instructions: String

  // 부모 designated init을 override하는 convenience override init
  convenience override init(players: [Player], numberOfTiles: Int) {
    // The initializer now points sideways (self.init) versus upwards (super.init)
    self.init(player: players, instructions: "Read the manual", numberOfTiles: numberOfTiles)
  }

  // 자식 클래스의 designated int
  init(players: [Player], instructions: String, numberOfTiles: Int) {
    self.instructions = instructions
    super.self(players: players, numberOfTiles: numberOfTiles)
  }
}
```

위의 코드처럼 override init(designated init)을 convenience override init으로 바꾸면 자식 클래스의 designated init을 두 개에서 한 개로 줄입니다.

이제는 MutabilityLand의 자식 클래스가 생기면 자식 클래스에서는 하나의 designated init만 override하면 MutabilityLand의 모든 init을 상속 받을 수 있습니다.

물론 designated init의 특성상 designated init에서 모든 프로퍼티가 초기화되어야 합니다.
따라서 자식 클래스에 추가된 초기화가 필요한 프로퍼티는 designated init에서 초기화하고 부모 클래스의 designated init을 호출합시다.

다시 말하지만 designated init은 모든 클래스에 한 개 이상 존재해야 하고 모든 프로퍼티를 초기화할 수 있어야 합니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/8b31c897-857c-4e05-ab5d-bfc380e5c4d5)

```swift
class MutabilityLandJunior: MutabilityLand { 
  let soundsEnabled: Bool

  init(soundsEnabled: Bool, players: [Player], instructions: String, numberOfTiles: Int) {
    self.soundsEnabled = soundsEnabled
    super.init(players: players, instructions: instructions, numberOfTiles: numberOfTiles)
  }

  // 부모 클래스의 designated init을 convenience init으로 override 했습니다.
  convenience override init(players: [Player], instructions: String, numberOfTiles: Int) {
    self.init(soundsEnabled: false, player: players, instructions: "Read the manual", numberOfTiles: numberOfTiles)
  }
}
```

Thanks to convenience override, this subclass gets many initializers for free.

## Required initializers

Required initializers play a crucial role when subclassing classes.

init 앞에 required를 붙여 자식 클래스가 해당 init을 필수로 구현하도록 만들 수 있습니다.
required init을 사용하는 경우는 크게 두 가지입니다.

1. factory methods
2. protocol with have init

먼저 factory methods는 인스턴스를 사전에 구성하도록 하는 전략입니다. 
따라서 아래 makeGame과 같이 Self를 리턴하는 함수를 구현하게 될 것입니다.

```swift
class BoardGame {
  // ...생략

  class func makeGame(players: [Player]) -> Self {
    let boardGame = self.init(players: players, numberOfTiles: 32)
    return boardGame
  }
}
```

Self를 리턴하는 함수는 말 그대로 본인의 타입을 리턴하는 함수입니다.
위의 makeGame 함수는 BoardGame 타입을 리턴하게 됩니다.
BoardGame 인스턴스를 self.init을 통해 만들어 집니다.

하지만 위 코드에는 required error가 발생합니다. 
makeGame 함수가 self.init을 사용하기 때문에 BoardGame 클래스의 자식 클래스가 생겼을 경우 리턴 타입인 Self와 makeGame 함수에서
만드는 self.init의 타입이 일치하지 않을 수 있습니다.

따라서 이 경우 자식 클래스에서 init을 구현하도록 강제해야합니다.
자식 클래스에서 required init을 재정의할 때는 override 키워드 대신 required 키워드를 사용합니다.

아래 코드를 확인해 봅시다.

```swift
class BoardGame {
  // ...생략

  class func makeGame(players: [Player]) -> Self {
    let boardGame = self.init(players: players, numberOfTiles: 32)
    return boardGame
  }

  required init(players: [Player], numberOfTiles: Int) {
    self.players = players
    self.numberOfTiles = numberOfTiles
  }
}

class Mutability: BoardGame {
  //... 생략

  // convenience init으로 부모 클래스의 required init을 재정의했습니다.
  covenience required init(players: [Player], numberOfTiles: Int) {
    // self.init에서 초기화 이후, 부모 클래스의 designated init 호출합니다.
    self.init(players: players, instructions: "Read the manual", numberOfTiles: numberOfTiles)
  }
}
```

두 번째로 required init이 필요한 경우는 프로토콜이 init을 가졌을 때입니다.
클래스가 해당 프로토콜을 채택하면 반드시 required init을 구현해야 합니다.

아래와 같이 프로토콜이 init을 가질 수 있습니다.

```swift
protocol BoardGameType {
  init(players: [Player], numberOfTiles: Int)
}

class BoardGame: BoardGameType {
  // ...생략
  required init(players: [Player], numberOfTiles: Int) {
    self.players = players
    self.numberOfTiles = numberOfTiles
  }
}
```

BoardGame이 BoardGameType 프로토콜을 따르기 때문에 BoardGame의 자식 클래스들도 BoardGameType을 따라야 합니다.
따라서 BoardGame의 init에 required 키워드를 붙여 자식 클래스에서 init의 구현을 강제합니다.

만약, 클래스에 final 키워드를 붙였다면 어떨까요?

init을 가진 프로토콜을 채택한 클래스를 final로 만든다면, 자식 클래스로 subclassing 될 가능성이 없어졌기 때문에 init을 required 할 필요가 없어집니다.

required 자체가 subclassing의 상황에 대응하기 위해 사용하는데 final로 subclassing 기능을 막는다면 init을 가진 프로토콜, factory method 경우 모두 init에 required 키워드를 붙일 필요가 없습니다.

## Summary
- Structs and classes want all their non-optional properties initialized
- Structs generate "free" memberwise initializers
- Structs lose memberwise initializers if you add a custom initializer
- If you extend structs with your custom initializers, you can have both memberwise and custom initializers
- Classes must have one or more designated initializers
- Convenience initializers point to designated initializers
- If a subclass has its own stored prperties, it won't directly inherit its superclass initializers
- If a subclass overrides designated initializers, it gets the convenience initializers from the superclass
- When overriding a superclass initializer with a convenience initializer, a subclass keeps the number of designated initializers down
- The required keyword makes sure that subclasses implement an initializer and that factory methods work on subclasses
- Once a protocol has an initializer, the required keyword makes sure that subclasses conform to the protocol
- By making a class final, initializers can drop the required keyword
