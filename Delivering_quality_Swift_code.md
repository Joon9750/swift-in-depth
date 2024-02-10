# Delivering quality Swift code

## This chapter covers
- Documenting code via Quick Help
- Writing good comments that don't distract
- How style isn't too important
- Getting consistency and fewer bugs with SwiftLint
- Splitting up large classes in a Swifty way
- Reasoning about making types generic

## API documentation
드디어 마지막 챕터입니다. ^~^

마지막 챕터(Delivering quality Swift code)에서는 코드적인 부분보다 프로젝트에 유용하게 쓸 수 있는 도구들을 살펴볼 예정입니다.

프로젝트는 주로 여러 동료들과 함께합니다. 따라서 자연스럽게 동료들이 만든 코드를 해석해야 합니다.
이때 API documentation을 만들어 프로젝트의 코드를 문서화 할 수 있습니다.

API documentation은 외부에서 사용 가능한 public 요소를 설명하는데 중요한 역할을 합니다.
프로젝트 안에서 public 요소에 접근하기 위해서 완성된 문서를 읽는 개념입니다.

프로젝트의 코드를 문서화하는 방법은 Quick Help를 사용하거나 Jazzy 패키지를 사용해 문서화된 웹 페이지를 만드는 방법이 있습니다.

먼저 Quick Help를 사용하는 방법을 살펴봅시다.

Quick Help는 짧은 마크다운으로 코드상에서 아래와 같이 표시됩니다.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/0004055c-4ed8-467d-b25d-48d0a9d9fffa)

위와 같이 코드상에서 마우스를 통해 표시 되며 아래와 같이 sidebar에서도 표시됩니다.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/92721cc4-2f3e-48b8-b493-5f6855be4e81)


위와 같은 Quick Help는 '///'를 통해 작성할 수 있습니다.
아래 코드로 살펴봅시다.

```swift
/// A player's turn in a turn-based online game.
enum Turn {
  /// Player skips turn, will receive gold.
  case skip
  /// Player uses turn to attack location.
  /// - x: Coordinate of x location in 2D space.
  /// - y: Coordinate of y location in 2D space.
  case attack(x: Int, y: Int)
  /// Player uses round to heal, will not receive gold.
  case heal
}
```

**Adding callouts to Quick Help**

Quick Help에 함수가 에러를 던지는 여부나 예시 코드를 추가할 수 있습니다.

이때 'tab text'를 통해 Quick Help에 설명을 추가합니다.
아래 코드로 살펴봅시다.

```swift
/// Takes an array of turns and plays them in a row.
///
/// - Parameter turns: One or multiple turns to play in a round.
/// - Returns: A description of what happened in the turn.
/// - Throws: TurnError
/// - Example: Passing turns to `playTurn`.
///
///         let turns = [Turn.heal, Turn.heal]
///         try print(playTurns(turns)) "Player healed twice."
func playTurns(_ turns: [Turn]) throws -> String {
```

위와 같이 tab text가 추가되어 아래와 같은 Quick Help가 작성됩니다.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/64052fdd-cd2f-4650-b173-9303bc8170d8)

**Documentation as HTML with Jazzy**

Quick Help를 작성하는 방법 이외에도 프로젝트를 문서화하는 방법이 있습니다.

realm에서 제공하는 Jazzy 패키지를 사용하여 프로젝트를 문서화할 수 있습니다.

You can apply Jazzy to any project where you'd like to generate a website with documentation.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/e23d3992-9c35-43e3-9321-1f13186e72a9)

Jazzy는 command-line에서 적용할 수 있고 아래 코드와 같이 실행할 수 있습니다.

```swift
// Jazzy 패키지 설치합니다.
gem install jazzy
jazzy
```

```swift
// Jazzy output
Running xcodebuild
building site
building search index
downloading coverage badge
jam out to your fresh new docs in `docs`
```

## Comments

개발을 하다보면 주석을 자주 보게됩니다. 
어렴풋이 주석은 코드의 가독성을 떨어뜨린다고 들었을 것입니다.

하지만 개발에 주석은 어쩔수 없이 쓰이는 존재입니다. 
주석이 코드 가독성을 떨어뜨릴 수 있지만, 반대로 가독성을 높일 수도 있습니다.

지금부터 코드 가독성을 떨어뜨리는 주석과 가독성을 높이는 주석의 차이를 살펴봅시다.

**Explain the "why"**

"무엇"을 하는 변수인지 설명하는 주석은 코드의 가독성을 떨어뜨리고, 변수가 "왜" 여기 있는지를 설명하는 주석은 코드의 가독성을 높입니다.

변수가 "무엇"을 하는지 설명하는 가독성을 떨어뜨리는 주석을 먼저 살펴봅시다.

```swift
struct Message {
  // The id of the message
  let id: String

  // The date of the message
  let date: Date

  // The contents of the message
  let contents: String
}
```

변수가 무엇을 하는지 설명하는 주석은 필요하지 않습니다.
이미 변수명으로 해당 변수가 어떤 역할을 하는지 충분히 설명하고 있기 때문입니다.

아래 코드와 같이 변수가 "왜" 여기 위치하는지 설명하는 주석으로 코드의 가독성을 높입시다.

```swift
struct Message {
  let id: String
  let date: Date
  let contents: String

  // Messages can get silently cut off by the server at 280 characters.
  let maxLength = 280
}
```

Message 구조체가 maxLength 변수를 가지는 이유를 주석으로 설명하고 있습니다.
이와 같은 왜 변수가 해당 위치에 있는지에 대한 설명은 변수명으로 부족할 수 있기 때문에 주석이 부족한 설명을 보충하게 됩니다.

Not all "Why" need to be explained.

아무리 유용한 주석이더라도 시간이 지나면 쓸모 없어지거나 오히려 코드에 피해만 끼칩니다.
따라서 무분별한 주석 작성은 피해야 합니다.

특히 적절하지 않은 변수명이나 함수명을 보충 설명하기 위한 주석은 최악입니다.

**The code has the truth.**

## Settling on a style

팀원들과 개발을 하다보면 서로 다른 코딩 스타일로 일관되지 못한 결과물이 만들어집니다.
어떤 개발자는 for loop을 선호하고 또 다른 개발자는 forEach를 선호할 수 있습니다.

이때 SwiftLint를 사용하면 일관되는 코딩 스타일을 가진 결과물을 만들 수 있습니다.

SwiftLint는 아래와 같은 .yml 파일로 관리됩니다.

```swift
disabled_rules: # rule identifiers to exclude from running
  - variable_name
  - nesting
  - function_parameter_count
opt_in_rules: # some rules are only opt-in
  - control_statement
  - empty_count
  - trailing_newline
  - colon
  - comma
included: # paths to include during linting. `--path` is ignored if present.
  - Project
  - ProjectTests
  - ProjectUITests
excluded: # paths to ignore during linting. Takes precedence over `included`.
  - Pods
  - Project/R.generated.swift

# configurable rules can be customized from this configuration file
# binary rules can set their severity level
force_cast: warning # implicitly. Give warning only for force casting

force_try:
  severity: warning # explicitly. Give warning only for force try

type_body_length:
  - 300 # warning
  - 400 # error

# or they can set both explicitly
file_length:
  warning: 500
  error: 800

large_tuple: # warn user when using 3 values in tuple, give error if there are 4
   - 3
   - 4

# naming rules can set warnings/errors for min_length and max_length
# additionally they can set excluded names
type_name:
  min_length: 4 # only warning

    error: 35
  excluded: iPhone # excluded via string
reporter: "xcode"
```

.swiftlint.yml 파일을 수정하여 SwiftLint의 규칙을 수정할 수 있습니다.

SwiftLint를 적용한 후 규칙을 따르지 않을 경우 아래와 같이 경고를 띄우거나 아예 에러를 띄울 수도 있습니다.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/47699c60-61e6-444a-8920-33b8b84d13d4)

물론 :previous, :this 또는 :next 키워드로 특정 코드에서 SwiftLint의 규칙을 비활성화 시킬 수 있습니다.
아래 코드로 살펴봅시다.

```swift
//  Turning off violating rules
if list.count == 0 { // swiftlint:disable:this empty_count
      // swiftlint:disable:next force_unwrapping
      print(lastLogin!)
}
```

## Kill the managers

개발을 하다보면 Manager 클래스를 빈번히 볼 수 있습니다.
BluetoothManger나 ApiRequestManger와 같이 Manager가 접미사로 붙는 객체는 너무 많을 책임을 가지게 됩니다.

하나의 클래스가 여러 책임을 가질 경우 코드 수정이 어려워지고 가독성도 떨어집니다.

여러 책임을 가지는 클래스를 작은 타입으로 나누고 앱 다른 부분에서도 쓰일 만한 타입은 제네릭 타입으로 만들어 봅시다.

여러 책임을 가진 ApiRequestManager 클래스를 예시로 살펴봅시다.

ApiRequestManager 클래스는 네트워크 호출, 데이터 캐싱, 큐에 저장, 웹 소켓과 관련된 모든 책임을 갖는 Manager 클래스입니다.
ApiRequestManager 클래스가 가진 책임을 아래와 같이 명시하여 얼마나 많은 책임을 갖는지 살펴봅시다.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/81346efe-dad3-45f1-bc02-6e0967e4c92c)

여러 책임을 나누기 위해서 먼저 여러 책임들을 각 독립된 타입으로 만들어야 합니다.
각 역할을 하는 타입들을 독립시켜, 특정 부분에 문제가 생겼을 때 특정 독립된 타입만 수정할 수 있습니다.

예를 들어 Cache에 문제가 발생했을 때 ApiRequestManager 클래스 전체가 아닌 ResponseCache 속 코드만 살펴보면 됩니다.

또한 ApiRequestManager 클래스를 Network 클래스로 더 정밀한 이름을 사용할 수 있습니다.

**Paving the road for generics**

만약 queueing과 caching 동작이 Network 클래스 이외에 앱 다른 부분에도 사용된다면, ResponseQueue와 ResponseCache를 제네릭 타입으로 만들어 사용할 수 있습니다.

아래와 같이 ResponseQueue를 Queue<T>로 수정하고 제네릭 T 타입에 Response 타입을 넣는 방식으로 제네릭 Queue<T>를 만들게 됩니다.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/98a8a3bb-a82d-44a8-9423-afeaf1a724b7)

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/40266d8f-644b-40c1-88db-9868cf7f0a88)

## Naming abstractions

**Good names don't change**

변수명을 짓는 일이 중요하다는 것은 모두들 알고 있습니다.
하지만 보통 별 고민 없이 변수명을 짓는 경우 아래와 같이 너무 특수화된(overspecified) 변수명을 짓게 됩니다.

너무 특수화된 변수명은 추가적인 기능을 구현할 때 해당 기능과 어울리지 않을 수 있습니다.

아래 코드는 방문했던 커피 가게 중 가장 마음에 들었던 커피 가게 다섯개를 뽑는 요구사항을 구현한 코드입니다.

```swift
let locations = ... // extracted locations.
let favoritePlaces = FavoritePlaces(locations: locations)
let topFiveFavoritePlaces = favoritePlaces.calculateMostCommonPlaces()

let coffeePlaces = topFiveFavoritePlaces.filter { place in place.type == "Coffee" }****
```

가장 마음에 들었던 커피 가게를 뽑는 요구사항에는 favoritePlaces 변수명이 적합할지 몰라도 favoritePlaces는 너무 특정 상황에 국한된 변수명입니다.

만약 사용자가 방문한 커피 가게 중 가장 적게 방문한 커피 가게를 뽑는 요구사항에는 favoritePlaces 변수명이 적합하지 않습니다.
이처럼 너무 특수화된 변수명은 새로운 요구사항에 어울리지 않을 수 있습니다.

The type is named after how it is used, which is to find the favorite places. But the type's name would be better if you can name it after what it does, which is find and group occurrences of places.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/379dd4d1-92d7-4ef7-8ccc-798703481a14)


favoritePlaces과 달리 LocationGroper 변수명은 요구사항이 추가되어도 잘 어울려 변수명을 수정하지 않아도 됩니다.

결과적으로 좋은 변수명은 수정할 필요 없는 변수명입니다.

몇가지 예를 더 살펴봅시다.

1. Don't use something like 'redColor' as a button's property for a warning state; a 'warning' property might be better because the warning's design might change, but a warning's purpose won't.
2. When creating a 'UserheaderView' - which is nothing more than an image and label you can reuse as something else - perhaps 'ImageDescriptionView' would be more fitting as well as reusable.

## Summary
- Quick Help documentation is fruitful way to add small snippets of documentation to your codebase.
- Quick Help documentation is especially valuable to public and internal elements offered inside a project and framework.
- Quick Help supports many useful callouts that enrich documentation.
- You can use Jazzy to generate Quick Help documentation.
- Comments explain the "why", not the "what".
- Be stingy with comments.
- Comments are no bandage for bad naming.
- There's no need to let commented-out code, aka Zombie Code, linger around.
- Code consistency is more important than code style.
- Consistency can be enforced by installing SwiftLint.
- SwiftLint supports configurations that you can let your team decide, which helps eliminate style discussions and disagreements.
- Manager classes can drop the -Manager suffix and still convey the same meaning.
- A large type can be composed of smaller types with focused responsibilities.
- Smaller components are good candidates to make generic.
- Name your types after what they do, not how they are used.
- The more abstract a type is, the more easily you can make it generic and reusable.
