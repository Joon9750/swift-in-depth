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

연산 프로퍼티를 통해 훨씬 가독성이 높아졌습니다. 앞으로 함수 중 연산프로퍼티로 바꿀것들은 바꿔서 사용합시다.
더 깔끔한 코드가 될 것입니다.

## Lazy properties
앞에서 보았듯이 연산 프로퍼티는 타입에 동적 성질을 부여합니다. 하지만 연산 프로퍼티가 항상 최선은 아닙니다.
연산 프로퍼티를 통한 계산이 복잡해질 경우(시간이 오래 소요될 경우) 부적합합니다. 이때는 lazy 프로퍼티를 고려해야 합니다.
연산 프로퍼티에 접근하면 매번 연산을 진행하게 됩니다. 하지만 lazy 프로퍼티는 다릅니다.

lazy properties make sure properties are calcuted at a later time(if at all) and only once!!

아래 코드는 복잡한 계산을 가진 연산 프로퍼티가 발생시킬 문제점을 보여줍니다.

```swift
struct LearningPlan {
  let level: Int
  var description: String

  // contents is a computed property
  var contents: String {
    print("I'm taking my sweet time to calculate.")
    sleep(2)

    switch level {
    case ..<25: return "Watch an English documentary."
    case ..<50: return "Translate a newspaper article."
    case ..<25: return "Read two academic papers."
    case default: return "Try to read English for 30 minutes."
    }
  }
}

var plan = LearningPlan(level: 18, description: "A special plan for today!")

print(Date())  // 2023-11-18 18:04:43 +0000
print(plan.contents)   // "Watch an English documentary."
print(Date())  // 2023-11-18 18:04:45 +0000
```

위 contents는 동작에 2초가 소요되는 연산 프로퍼티입니다. 만약 contents에 5번 연속으로 접근하면 몇초가 소요될까요?
연산 프로퍼티는 접근할 때마다 동작하기 때문에 2초씩 5번 총 10초가 소요됩니다. 굉장한 딜레이라고 볼 수 있습니다.

물론 2초가 소요되는 동작의 속도를 줄이면 좋겠지만, 동작 자체의 속도와 별개로 lazy 프로퍼티를 사용하면 딜레이를 최소화할 수 있습니다.
lazy 프로퍼티는 본인이 호출되었을 때 단 한번만 동작합니다. 반복되어 호출되어도 가장 첫 호출에만 동작하고 이후 호출에는 첫 동작을 결과를 리턴합니다.
lazy 프로퍼티는 연산 프로퍼티와 달리 첫 호출로 계산된 값을 저장하여 이후 호출에 대응합니다.

아래 코드처럼 연산 프로퍼티를 lazy 프로퍼티로 고칠 수 있습니다. 하지만 아직 완벽한 코드는 아닙니다!

```swift
struct LearningPlan {
  let level: Int
  var description: String

  // contents is a computed property
  lazy var contents: String = {
    print("I'm taking my sweet time to calculate.")
    sleep(2)

    switch level {
    case ..<25: return "Watch an English documentary."
    case ..<50: return "Translate a newspaper article."
    case ..<25: return "Read two academic papers."
    case default: return "Try to read English for 30 minutes."
    }
  }()
}
```

구조체는 custom init을 만들지 않느다면, 컴파일러가 모든 프로퍼티를 초기화하는 init을 제공합니다.
하지만 LearningPlan 구조체가 생성과 동시에 














