# Understanding map, flatMap, and compactMap

## This chapter overs
- Mapping over arrays, dictionaries, and other collections
- When and how to map over optionals
- How and why to flatMap over collections
- Using flatMap on optionals
- Chaining and chort-circuiting computations
- How to mix map and flatMap for advanced transformations

## Becoming familiar with map
map 함수는 스위프트가 지향하는 함수형 프로그래밍에 어울리는 함수 중 하나입니다.
map은 데이터를 변형하고자 할 때 사용합니다.
기존 데이터를 변형하여 새로운 컨테이너를 만드는데, 기존 데이터는 변형되지 않습니다.

아래 코드는 map 함수의 정의부입니다.

```swift
func map<T>(_ transform: (Self.Element) throws -> T) rethrows -> [T]
```

물론 for loop 방식(명령형)도 좋은 시작입니다.
하지만 for loop 방식에는 몇가지 boilerplate가 존재할 수 있습니다.

아래에서 살펴볼 예시는 프로젝트에 참가한 사람들의 이름과 커밋 개수들을 더 읽기 쉬운 형식으로 변형하는 상황입니다.
먼저 소개되는 코드는 for loop 방식입니다. for loop 방식이 가진 boilerplate를 확인하고 map(함수형) 함수로 개선해 봅시다.

```swift
let commitStats = [
  (name: "Miranda", count: 30),
  (name: "Elly", count: 650),
  (name: "John", count: 0)
]

let readableStats = resolveCounts(statistics: commitStats)
print(readableStats)  // ["Miranda isn't very active on the project", "Elly is quite active", "John isn't involved in the project"]

// for loop 사용
func resolveCounts(statistics: [(String, Int)]) -> [String] {
  var resolvedCommits = [String]()  // for loop을 위한 임시변수
  for (name, count) in statistics {
    let involvement: String

    switch count {
    case 0: involvement = "\(name) isn't involved in the project"
    case 1..<100: involvement = "\(name) isn't active on the project"
    default: involvement = "\(name) is active on the project"
    }

    resolvedCommits.append(involvement)
  }
  return resolvedCommits
}
```

위의 resolveCounts 함수의 for loop 코드를 살펴보면 resolvedCommits 임시변수를 필요로 합니다. 
이 부분이 for loop의 boilerplate라고 할 수 있습니다.
resolvedCommits 변수는 var로 선언되어 실수로라도 변형될 가능성이 있습니다.

이때 map을 사용하면 추가적인 var 타입 임시변수 없이 데이터를 순회하며 클로저로 요소를 보내고 이후에 새로운 결과를 리턴합니다.
map을 사용하면 새로운 결과를 만들어 리턴하기 때문에 추가적인 임시변수가 필요하지 않습니다.

아래 코드는 위의 for loop 코드를 map을 사용한 코드로 리팩토링한 코드입니다.

```swift
func resolveCounts(statistics: [(String, Int)]) -> [String] {
  return statistics.map { (name: String, count: Int) -> String in
    switch count {
    case 0: return "\(name) isn't involved in the project."
    case 1..<100: return "\(name) isn't very active on the project."
    default: return "\(name) is active on the project"
    }
  }
}
```

위 코드는 아래와 같은 과정에 따라 데이터를 변형합니다.

1. You being with an array.

2. With map, you apply a function to each value inside the array.

3. Map creates a new array with the transformed values inside.

map과 for loop가 동일한 결과를 만들지만, map은 for loop 보다 간결하고 불변한 코드를 만들 수 있습니다.

**Creating a pipeline with map**

데이터 변환에 있어 파이프라인은 각 단계를 표현하는 유용한 방법입니다.

지금부터 이름과 커밋 개수 데이터들 중에서 커밋 개수가 0개인 데이터를 필터링하고 나머지 데이터 중 커밋 개수만 내림차순 정렬하는 함수를 만들어 봅시다.
해당 함수를 for loop을 사용해 먼저 구현해 봅시다.

아래 코드를 확인해 봅시다.

```swift
func counts(statistics: [(String, Int)]) -> [Int] {
  var counts = [Int]()
  for (name, count) in statistics where count > 0 {
    counts.append(count)
  }
  return counts.sorted(by: >)
}
```

위의 코드 같은 for loop 방식은 counts 임시변수가 필요합니다.
for loop 방식을 pipeline을 활용한 방식으로 고쳐봅시다.

```swift
func counts(statistics: [(String, Int)]) -> [Int] {
  return statistics
       .map { $0.1 }
       .filter { $0 > 0 }
       .sorted(by: >)
}  
```

for loop을 대신해 각 단계를 개별적으로 설명하는 파이프라인 방식을 사용해 코드를 간결하게 개선했습니다.
위 코드처럼 map, filter, sorted 등의 파이프라인들을 결합하여 사용할 수 있습니다.





















