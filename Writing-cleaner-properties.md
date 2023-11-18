# Writing cleaner properties

## This chapter cover
- How to create getter and setter computed properties
- When(not) to use computed properties
- Improving performance with lazy properties
- How lazy properties behave with structs and mutability
- Handling stored properties with behavior

## Computed properties
연산 프로퍼티는 프로퍼티로 가장한 함수로 여겨집니다. 어떤 값을 저장하지 않지만 저장프로퍼티와 형태는 유사합니다.
연산 프로퍼티에 접근할 때마다 동작합니다. 따라서 "always up to date" 성질을 가집니다.

함수보다 연산 프로퍼티가 가독성이 높습니다. 
함수중에서도 인자 없이 return 값은 있는 함수가 연산 프로퍼티로 바꿀만한 함수입니다.
함수를 연산 프로퍼티로 고치는 코드를 살펴봅시다.

```swift
import Foundation

struct Run {
  let id: String
  let startTime: Date
  var endTime: Date?

  func elapsedTime() -> TimeInterval {
    return Date().timeIntervalSince(startTime)
  }

  func isFinished() -> Bool {
    return endTime != nil
  }

  mutating func setFinished() {
    self.endTime = Date()
  }

  init(id: String, startTime: Date) {
    self.id = id
    self.startTime = startTime
    self.endTime = nil
  }
}

// use
run.elapsedTime()
run.isFinished()
```

위 코드에서 elapsedTimed와 isFinished 함수는 인자 없이 return 값만 존재하는 함수입니다.
두 함수를 아래와 같은 연산 프로퍼티로 수정하며 가독성을 높일 수 있습니다.
연산 프로퍼티의 자료형은 해당 프로퍼티가 return하는 값의 자료형을 명시하면 됩니다.

```swift
import Foundation

struct Run {
  let id: String
  let startTime: Date
  var endTime: Date?

  var elapsedTime: TimeInterval {
    return Date().timeIntervalSince(startTime)
  }

  var isFinished: Bool {
    return endTime != nil
  }

  mutating func setFinished() {
    self.endTime = Date()
  }

  init(id: String, startTime: Date) {
    self.id = id
    self.startTime = startTime
    self.endTime = nil
  }
}

// use by computed properties
run.elapsedTime
run.isFinished
```




















