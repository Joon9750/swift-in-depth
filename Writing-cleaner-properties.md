# Writing cleaner properties

## This chapter cover
- How to create getter and setter computed properties
- When(not) to use computed properties
- Improving performance with lazy properties
- How lazy properties behave with structs and mutability
- Handling stored properties with behavior

## Computed properties
연산 프로퍼티는 프로퍼티로 가장한 함수로 여겨집니다. 어떤 값을 저장하지 않지만, 저장 프로퍼티와 형태는 유사합니다.
연산 프로퍼티에 접근할 때마다 동작합니다. "always up to date" 성질을 가집니다.

함수보다 연산 프로퍼티가 가독성이 높습니다. 
함수 중에서도 "들어오는 인자 없이 리턴 값만 있는 함수"가 연산 프로퍼티로 바꿀 후보가 되는 함수입니다.
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

위 코드에서 elapsedTimed와 isFinished 함수는 인자 없이 리턴 값만 존재하는 함수입니다.
두 함수를 아래와 같은 연산 프로퍼티로 수정하며 가독성을 높일 수 있습니다.
연산 프로퍼티의 자료형은 해당 프로퍼티가 리턴턴 하는 값의 자료형을 명시하면 됩니다.

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

연산 프로퍼티를 통해 훨씬 가독성이 높아졌습니다. 앞으로 함수 중 연산프로퍼티로 바꿀 것들은 바꿔서 사용합시다.
더 깔끔한 코드가 될 것입니다.

## Lazy properties
앞에서 보았듯이 연산 프로퍼티는 타입에 동적 성질을 부여합니다. 하지만 연산 프로퍼티가 항상 최선은 아닙니다.
연산 프로퍼티에서 발생하는 계산이 복잡해질 경우(시간이 오래 소요될 경우) 부적합합니다. 이때는 lazy 프로퍼티를 고려해야 합니다.
연산 프로퍼티는 접근할 때마다 연산을 진행하게 됩니다. 따라서 복잡한 계산을 포함할 경우 성능이 굉장히 떨어집니다.

하지만 lazy 프로퍼티는 다릅니다. lazy 프로퍼티는 연산 프로퍼티와 다르게 처음 호출되었을 때 한 번만 실행됩니다.

lazy properties make sure properties are calcuted at a later time(if at all) and only once!!

아래 코드는 복잡한 계산을 가진 연산 프로퍼티가 발생시키는 문제점을 보여줍니다.

```swift
struct LearningPlan {
  let level: Int
  var description: String

  // contents is a computed property - 2초 딜레이 발생생
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
연산 프로퍼티는 접근할 때마다 동작하기 때문에 2초씩 5번 총 10초가 소요됩니다. 굉장한 딜레이가 발생합니다.

물론 2초가 소요되는 동작의 속도를 줄이면 좋겠지만, 동작 자체의 속도와 별개로 lazy 프로퍼티를 사용하면 딜레이를 최소화할 수 있습니다.
lazy 프로퍼티는 본인이 호출되었을 때 단 한 번만 동작합니다. 반복되어 호출되어도 가장 첫 호출에만 동작하고 이후 호출에는 첫 동작을 결과를 리턴합니다.
lazy 프로퍼티는 연산 프로퍼티와 달리 첫 호출로 계산된 값을 저장하여 이후 호출에 리턴합니다.

아래 코드처럼 연산 프로퍼티를 lazy 프로퍼티로 고칠 수 있습니다. 하지만 아직 완벽한 코드는 아닙니다.

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

스위프트는 기본적으로 구조체의 모든 프로퍼티가 초기화되기를 바랍니다.
따라서 구조체의 custom init을 만들지 않는다면, 컴파일러가 모든 프로퍼티를 초기화하는 init을 제공합니다.

하지만 LearningPlan 구조체가 생성과 동시에 lazy 프로퍼티가 초기화된다면 프로퍼티가 더 이상 수정되지 못하여 프로퍼티로서 역할을 못합니다.
이를 custom init을 통해 해결할 수 있습니다. 

```swift
struct LearningPlan {
  // ... 생략
  init(level: Int, description: String) {
    self.level = level
    self.description = description
  }
}
```

위의 custom init처럼 lazy 프로퍼티를 제외하고 나머지 프로퍼티를 초기화해야 합니다.
custom init으로 객체 생성과 동시에 lazy 프로퍼티가 초기화되지 않기 때문에 lazy 프로퍼티의 연산이 필요할 때 접근해 사용할 수 있습니다.

앞에서 말했듯이 lazy 프로퍼티는 중복해서 해출되어도 한 번만 초기화되고 처음 초기화된 값을 저장합니다. 
하지만 누군가 lazy 프로퍼티에 접근해 임의로 값을 변경한다면 어떻게 될까요?

내부 로직을 무시하고 lazy 프로퍼티를 임의로 변경하는 행동은 예상과 다른 결과를 초래하기며 혼란을 가져옵니다.

아래 코드에서 plan 객체를 level 18로 초기화하여 plan.contents의 값을 "Watch an English documentary"로 기대합니다.
하지만 plan.contents = "Let's eat pizza and watch Netflix all day"에서 lazy 프로퍼티 값을 바꾸면서 예상과 다른 결과가 보입니다.

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

"Once you initialized a lazy property, you cannot change it!!"

앞에서 이야기했듯이 lazy 프로퍼티는 한 번 초기화되면 이후로 변경되지 않습니다.
하지만 lazy 프로퍼티 안에서 쓰이는(참조하는) 변수가 변경 가능성이 있다면 어떻게 될까요?
아래 코드를 확인해 봅시다.

```swift
var intensePlan = LearningPlan(level: 138, description: "A special plan for today!")
intensePlan.contents
var easyPlan = intensePlan
easyPlan.level = 0
print(easyPlan.contents) // "Read two academic papers."
```

코드에서는 easyPlan의 level을 0으로 만들어 0에 해당하는 contents를 기대하지만 실제로는 기존 intensePlan의 138에 해당하는 contents가 출력됩니다.

lazy 프로퍼티가 초기화된 이후 lazy 프로퍼티 안에서 쓰이는(참조하는) 변수가 변경되어도, lazy 프로퍼티에 처음 저장된 값은 변경되지지 않습니다.
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
따라서 lazy 프로퍼티 안에서 쓰이는(참조하는) 변수는 let으로 선언하도록 합시다.

lazy 프로퍼티 안에서 쓰이는 변수를 let으로 선언하여 lazy 프로퍼티를 immutable하게 만들어 복잡성을 줄입시다.
lazy 프로퍼티를 가진 객체가 복사될 때 더 주의해야 합니다.

다시 말해, lazy 프로퍼티가 초기화된 이후 lazy 프로퍼티 내부에서 참조하는 변수가 변경되더라도 
해당 변경 사항이 lazy 프로퍼티에 영향을 미칠 수 없기 때문에 lazy 프로퍼티 내부에서 참조하는 변수를 불변하게 만듭시다.

lazy var 프로퍼티를 사용하다보면 retain cycle을 주의해야 합니다.

아래와 같이 블록{} 마지막에 () 가 붙은 형태는 nonescaping closure라고 해서 실행 즉시 결과를 반환함을 가정하기 때문에 클로저 내부에서 self를 명시적으로 참조할 필요가 없고, self instance의 retain count를 올리지도 않기 때문에 retain cycle을 걱정할 필요가 없습니다.

마지막에 () 을 통해서 즉시 실행하고 결과를 돌려주기에 메모리 누수는 없습니다.

```swift
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
```

반면 아래와 같은 형태로 closure를 타입으로 선언 후 사용하면 self를 명시적으로 참조해야 하고, closure가 실행되고 나서도 self의 retain count가 증가한 상태로 유지되기 때문에 weak capture없이 self를 사용하면 retain cycle이 발생할 수 있습니다.

```swift
lazy var contents: () -> String = {
  print("I'm taking my sweet time to calculate.")
  sleep(2)

  switch self.level {
  case ..<25: return "Watch an English documentary."
  case ..<50: return "Translate a newspaper article."
  case ..<25: return "Read two academic papers."
  case default: return "Try to read English for 30 minutes."
  }
}()
```

contents 프로퍼티는 () -> String 클로저를 리턴하고 있습니다. 따라서 [weak self]를 통해서 메모리 누수를 방지해주어야 합니다.
contents 프로퍼티에 의해 메모리에는 값이 아닌 클로저가 올라가 있고 해당 클로저는 self의 참조를 유지하고 있습니다.

```swift
lazy var contents: () -> String = { [weak self] in
  print("I'm taking my sweet time to calculate.")
  sleep(2)

  switch self?.level {
  case ..<25: return "Watch an English documentary."
  case ..<50: return "Translate a newspaper article."
  case ..<25: return "Read two academic papers."
  case default: return "Try to read English for 30 minutes."
  }
}()
```

따라서 앞에서 lazy var 프로퍼티가 값을 리턴한다면 프로퍼티 초기화 이후 계속해서 동일한 값을 리턴하지만 클로저를 리턴하는 경우 클로저 안에서 참조하는 변수의
변경에 따라 리턴 값이 달라질 수 있습니다.

그렇다면 lazy 프로퍼티는 항상 옳을까요?

아쉽게도 아닙니다.

lazy 프로퍼티는 비동기 작업, 멀티 쓰레드 환경에서 안전하지 못합니다.
lazy 프로퍼티가 단일 쓰레드 환경에서는 초기 접근에만 동작하고 이후에는 첫 동작의 결과를 리턴하지만 멀티 쓰레드 환경에서는 이 특징이 보장되지 않습니다.

아래 링크는 멀티 쓰레드 환경에서 lazy 프로퍼티가 가지는 취약점을 잘 설명한 글입니다.
책을 읽으며 lazy 프로퍼티를 초기화에 높은 비용이 드는 경우 사용하면 유용하겠다고 생각했지만 thread-safe하지 않다는 글을 보니 자주 사용할지 의구심이 들긴합니다. 

[Swift-Lazy-진짜-필요할-때만-씁시다](https://velog.io/@niro/Swift-Lazy-%EC%A7%84%EC%A7%9C-%ED%95%84%EC%9A%94%ED%95%A0-%EB%95%8C%EB%A7%8C-%EC%94%81%EC%8B%9C%EB%8B%A4)

[Swift의 lazy var](https://brunch.co.kr/@tilltue/71)

## Property observers
Property observers are actions triggered when a stored property changes value.

저장 역할과 함께 연산 기능까지 프로퍼티에서 구현하고 싶을 때 Property observers를 사용합니다.
Property observers로는 didSet과 willSet이 있습니다.

didSet은 변수 변경 이후 trigger되고 
willSet은 변수 변경 이전에 trigger됩니다.

스위프트에서는 init으로 초기화되는 경우 didSet이 따로 호출되지 않습니다.
따라서 아래와 같은 문제가 생깁니다.
아래 코드는 사용자가 이름을 입력했을 때 공백이 포함될 경우 공백을 지워주는 코드입니다.

```swift
import Foundation

class Player {
  let id: String

  var name: String {
    didSet {
      print("My previous name was \(oldValue)")
      name = name.trimmingCharacters(in: .whitespaces)
    }
  }

  init(id: String, name: String){
    self.id = id
    self.name = name
  }
}

let jeff = Player(id: "1", name: "SuperJeff  ")
print(jeff.name) // "SuperJeff   "
print(jeff.name.count) // 13

jeff.name = "SuperJeff  "
print(jeff.name) // "SuperJeff"
print(jeff.name.count) // 9
```

위 코드에서 name 프로퍼티의 didSet에는 공백 문자를 지워주는 기능을 제공하고 있습니다.
하지만 init을 통해 name 프로퍼티를 초기화했을 때 didSet에 도달하지 못해 공백 문자를 지우지 못했습니다.
위에서 말했듯이 스위프트에서는 init으로 프로퍼티를 초기화할 때는 Property observers가 호출되지 않습니다.

만약 init으로 프로퍼티를 초기화할 때부터 Property observers를 호출하고 싶다면, defer 클로저를 사용하면 가능합니다.
defer 클로저는 함수 안에 위치하고 해당 함수가 종료되었을 때 호출되는 클로저입니다.
아래 코드와 같이 init 안에 defer 클로저를 추가하여, defer 클로저에서 변수를 초기화하여 didSet을 호출할 수 있습니다.

```swift
class Player {
  // 생략

  init(id: String, name: String) {
    defer { self.name = name }
    self.id = id
    self.name = name // 생략 가능하다.
  }
} 
```

defer 클로저는 init과 별개이기 때문에 defer에서 name 초기화하면 didSet이 호출됩니다.
물론 defer 클로저가 생겨난 이유가 init에서 Property observers를 호출하기 위함은 아닙니다.
그렇치만 근사한 방법입니다.

## Summary
- You can use computed properties for properties with sepcific behavior but with-out storage.
- Computed properties are functions masquerading as properties.
- You can use computed properties when a value can be different each time you call it.
- Only lightweight functions should be made into computed properties.
- Lazy properties are excellent for expensive or time-consuming computations.
- Use lazy properties to delay a computation or if it may not even run at all.
- Lazy properties allow you to refer to other properties inside classes and structs.
- You can use the private(set) annotation to make properties read-only to outsiders of a class or struct.
- When a lazy property refers to another preperty, make sure to keep this other property immutable to keep complexity low.
- You can use property observers such as willSet and didSet to add behavior on stored properties.
- You can use defer to trigger property observers from an initializer.
