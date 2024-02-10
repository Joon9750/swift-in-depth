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

The code has the truth.

## Settling on a style

















