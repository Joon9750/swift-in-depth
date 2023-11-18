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
하지만 LearningPlan 구조체가 생성과 동시에 lazy 프로퍼티가 초기화 된다면 프로퍼티가 더 이상 수정되지 못하게 됩니다.
이를 custom init을 통해 해결 가능합니다. 

```swift
struct LearningPlan {
  // ... 생략
  init(level: Int, description: String) {
    self.level = level
    self.description = description
  }
}
```

위의 custom init처럼 lazy 프로퍼티를 제외하고 나머지 프로퍼티를 초기화합니다.

lazy 프로퍼티는 중복해서 해출되어도 한 번만 초기화 되고 초기화된 값을 저장합니다.

lazy 프로퍼티에 접근해 값을 바꿀 수 있습니다. 
하지만 이는 내부 로직을 무시하고 lazy 프로퍼티를 임의로 변경하는 행동이기 때문에 예상과 다른 결과를 초래합니다.

아래 코드에서 plan을 level 18로 초기화하여 plan.contents의 값을 "Watch an English documentary"로 기대합니다.
하지만 plan.contents = "Let's eat pizza and watch Netflix all day"에서 lazy 프로퍼티 값을 바꾸면서 예상과 다른 결과가 보여집니다.

```swift
var plan = LearningPlan(level: 18, description: "A special plan for today!")
plan.contents = "Let's eat pizza and watch Netflix all day"
print(plan.contents) // "Let's eat pizza and watch Netflix all day"
```

lazy 프로퍼티 값을 임의로 변경할 가능성을 막는 방법은 접근 제어자에 있습니다.
lazy 프로퍼티의 접근 제어자를 private(set)으로 설정한다면 변경을 막을 수 있습니다.
private(set)으로 읽기 전용 프로퍼티가 되어 lazy 프로퍼티의 구조체 외부에서의 변경 가능성을 막습니다.

```swift
struct LearningPlan {
  lazy private(set) var contents: String = {
    // 생략
  }()
}
```

Once you initialized a lazy property, you cannot change it.

앞에서 이야기했듯이 lazy 프로퍼티는 한 번 초기화되면 이후로 변경되지 않습니다.
하지만 lazy 프로퍼티 안에서 쓰이는(참조하는) 변수가 변경 가능성이 있다면 어떻게 될까요?
아래 코드를 확인해봅시다.

```swift
var intensePlan = LearningPlan(level: 138, description: "A special plan for today!")
intensePlan.contents
var easyPlan = intensePlan
easyPlan.level = 0
print(easyPlan.contents) // "Read two academic papers."
```

코드에서는 easyPlan의 level을 0으로 만들어 0에 해당하는 contents를 기대하지만 실제로는 기존 intensePlan의 138에 해당하는 contents가 출력됩니다.

lazy 프로퍼티가 초기화 된 이후 lazy 프로퍼티 안에서 쓰이는(참조하는) 변수가 변경되었다면, lazy 프로퍼티에 초기에 저장된 값이 변경되진 않습니다.
오히려 코드를 읽는 사람에게 혼란을 주게 됩니다.

아래 코드처럼 객체 복사 이후 lazy 프로퍼티를 호출한다면 intensePlan, easyPlan의 level에 어울리는 contents가 출력됩니다.

```swift
var intensePlan = LearningPlan(level: 138, description: "A special plan for today!")
var easyPlan = intensePlan
easyPlan.level = 0
print(intensePlan.contents) // "Read two academic papers."
print(easyPlan.contents) // "Watch an English documentary."
```

lazy 프로퍼티 안에서 쓰이는(참조하는) 변수가 변경 가능성이 있다면 굉장히 복잡해집니다.
따라서 lazy 프로퍼티 안에서 쓰이는(참조하는) 변수는 let으로 선언하여 이 문제를 해결합시다.

lazy 프로퍼티 안에서 쓰이는 변수를 let으로 선언하여 immutable한 lazy 프로퍼티로 만들어 복잡성을 줄입시다.
lazy 프로퍼티를 가진 객체가 복사될 때 더 주의 해야합니다.

다시 말해, lazy 프로퍼티가 초기화된 이후 lazy 프로퍼티 내부에서 참조하는 변수가 변경되더라도 
해당 변경 사항이 lazy 프로퍼티에 영향을 미칠 수 없기 때문에 lazy 프로퍼티 내부에서 참조하는 변수를 불변하게 만듭시다.

## Property observers
Property observers are actions triggered when a stored property changes value.

저장 역할과 함께 연산 기능까지 프로퍼티에서 구현하고 싶을 때 Property observers를 사용합니다.
Property observers로는 didSet과 willSet이 있습니다.

didSet은 변수 변경 이후 trigger되고 
willSet은 변수 변경 이전에 trigger됩니다.

스위프트에서는 init으로 초기화 되는 경우 didSet이 따로 호출되지 않습니다.
따라서 아래와 같은 문제가 생깁니다.
아래 코드는 사용자가 이름을 입력했을 때 공백이 포함될 경우 공백을 지워주는 코드입니다.





