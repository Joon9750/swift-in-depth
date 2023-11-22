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

Designated init(지정 초기자)은 모든 프로퍼티를 초기화해야 하며 클래스 타입에 반드시 한 개 이상의 Designated init이 필요합니다.
부모 클래스가 있을 경우 자식 클래스는 부모 클래스의 Designated init을 상속받습니다.

convenience init(편의 초기자)은 내부에서 마지막에는 반드시 Designated init를 호출해야 합니다. (self.init())
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

하지만 자식 클래스에서 부모 클래스에 없는 새로운 프로퍼티를 추가했을 때는 다릅니다.
자식 클래스에 새로운 프로퍼티가 추가되면 자식 클래스에 있던 부모 클래스의 모든 init은 사라집니다.
위에서 말했듯이 클래스의 모든 프로퍼티는 초기화되어야 합니다. 하지만 부모 클래스에 없는 프로퍼티가 자식 클래스에 추가되며 더 이상
부모 클래스 init으로는 자식 클래스의 프로퍼티를 초기화할 수 없어지게 됩니다.

이때 우린 새로운 designated init을 구현해야 합니다.
자식 클래스에 초기화 되어야 하는 새로운 프로퍼티가 추가되기 전까지는 자식 클래스에서 부모 클래스의 init을 모두 상속 받았습니다.
하지만 자식 클래스에 초기화 되어야 하는 새로운 프로퍼티가 추가되며 자식 클래스의 새로운 designated init을 구현하고 designated init에서 부모 클래스의 designated init을 호출하는 방식으로
프로퍼티를 초기화해야 합니다.

아래 코드로 확인해 봅시다.

```swift
class MutabilityLand: BoardGame {
  var scoreBoard = [String: Int]()
  var winner: Player?

  // 초기화 되어야 하는 새로운 프로퍼티
  let instructions: String

  // 자식 클래스의 designated init
  init(players: [Player], instructions: String, numberOfTiles: Int) {
    self.instructions = instructions
    super.init(players: players, numberOfTiles: numberOfTiles)
  }
}
```

위 코드에서 scoreBoard와 winner는 초기화될 필요 없는 프로퍼티입니다. 따라서 자식 클래스에 scoreBoard와 winner 프로퍼티만 추가 됐다면 부모 클래스의 모든 init을 상속받을 수 있습니다.
하지만 MutabilityLand 클래스의 instructions 프로퍼티는 초기화 되어야 하는 새로운 프로퍼티입니다.
instructions가 추가되며 부모 클래스의 모든 init을 상속받지 못합니다.
따라서 위와 같이 자식 클래스에 새로운 designated init을 만들어 초기화 되어야 하는 새로운 프로퍼티를 초기화함과 동시에 super.init을 호출하여 부모 클래스로 부터 상속받은 프로퍼티를 초기화 했습니다.

여기서 주의해야할 점은 super.init은 부모 클래스의 designated init을 자식 클래스에서 호출하는것 뿐이지 부모 클래스의 init을 상속받은건 아닙니다.

하지만 자식 클래스에 초기화될 필요가 있는 새로운 프로퍼티가 생겨도 부모 클래스의 init을 상속받는 방법이 있습니다!

자식 클래스에서 부모 클래스의 designated init을 override하면 자식 클래스는 부모 클래스의 모든 init을 상속받을 수 있습니다.
물론 override한 부모 클래스의 designated init에서는 자식 클래스의 새로운 프로퍼티를 반드시 초기화해야 합니다.

아래 코드를 살펴봅시다.

```swift
class Mutability: BoardGame { 
  // 생략
  override init(players: [Player], numberOfTiles: Int) {
    self.instructions = "Read the manual"  // super.init 호출 전에 초기화되어야 합니다.
    super.init(players, numberOfTiles: numberOfTiles)  // 필수입니다.
  }
}
```

super.init으로 부모 클래스로부터 상속받는 프로퍼티를 초기화하기 전에 자식 클래스에 새로 추가된 프로퍼티를 먼저 초기화해야 합니다.

In a class hierarchy, convenience init go horizontal, and designated init go vertical!

## Minimizing class initializers

위에서 보았듯이 subclassing과 함께 자식 클래스에 초기화가 필요한 프로퍼티가 추가되었을 때 부모 클래스의 init을 상속받기 위해 designated init을 상속했습니다.
하지만 이 동작이 반복될수록 자식 클래스의 designated init이 한 개씩 늘어나게 됩니다.

만약 부모 클래스의 designated init 한 개와 convenience init 두 개를 가질 때 자식 클래스에서 부모 클래스의 designated init을 override한다고 가정해봅시다.
결과적으로 자식 클래스가 갖게 되는 init은 부모 클래스에서 상속받는 convenience init 두 개와 override designated init 그리고 자식 클래스 본인만의 (반드시 가져야하는) designated init이 있습니다.

만약 subclassing 될 때마다 위의 동작이 반복되면 자식 클래스는 여러 designated init을 가지게 될것입니다.
(override designated init도 designated init입니다.) 이는 계층을 복잡하게 만들 수 있습니다.

지금부터 자식 클래스에 초기화가 필요한 프로퍼티가 추가되어도 부모 클래스의 init을 상속받으며 designated init을 한 개로 유지하는 방법을 살펴봅시다.

designated init을 override 할 때 convenience init으로 부모 클래스의 designated init을 override한다면 자식 클래스의 designated init을 한 개로 유지할 수 있습니다.
여기서 부모 클래스의 designated init을 자식 클래스의 convenience init으로 override하고 자식 클래스의 convenience override init에서 자식 클래스의 designated init을 호출하고 designated init에서 부모 클래스의 designated init을 
호출하도록 구현하면 됩니다.

아래 코드로 확인해봅시다.

```swift
class MutabilityLand: BoardGame {
  var scoreBoard = [String: Int]()
  var winner: Player?

  let instructions: String

  // 부모 designed init을 override하는 convenience override init
  convenience override init(players: [Player], numberOfTiles: Int) {
    // The initializer now points sideways (self.init) versus upwards (super.init)
    self.init(player: players, instructions: "Read the manual", numberOfTiles: numberOfTiles)
  }

  init(players: [Player], instructions: String, numberOfTiles: Int) {
    self.instructions = instructions
    super.self(players: players, numberOfTiles: numberOfTiles)
  }
}
```

위의 코드처럼 override init(designated init)을 convenience override init으로 바꾸며 designated init을 두 개에서 한 개로 줄입니다.
이제는 MutabilityLand의 자식 클래스가 생기면 자식 클래스에서는 하나의 designated init만 override하면 MutabilityLand의 모든 init을 상속 받을 수 있습니다.

그치만 designated init의 성격상 designated init에서 모든 프로퍼티가 초기화되어야 합니다.
따라서 자식 클래스에 추가된 초기화가 필요한 프로퍼티는 designated init에서 초기화하고 부모 클래스의 designated init을 호출합시다.
다시 말하지만 designated init은 모든 클래스에 한 개 이상 존재해야 하고 모든 프로퍼티를 초기화할 수 있어야 합니다.

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



























