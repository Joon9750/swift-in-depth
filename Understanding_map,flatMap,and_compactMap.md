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
// pipeline 방식
func counts(statistics: [(String, Int)]) -> [Int] {
  return statistics
       .map { $0.1 }  // statistics 배열을 순회하며 새로운 [Int] 배열을 리턴합니다.
       .filter { $0 > 0 }  // [Int] 배열 중 0인 요소를 필터링합니다.   
       .sorted(by: >)  // [Int] 배열을 내림차순으로 정렬합니다.
}  
```

for loop을 대신해 각 단계를 개별적으로 설명하는 파이프라인 방식을 사용해 코드를 간결하게 개선했습니다.
위 코드처럼 map, filter, sorted 등을 결합해 파이프라인을 구축할 수 있습니다.
파이프라인을 구축하는 작업에 있어 map 함수는 새로운 객체를 리턴하기 때문에 중요합니다.

파이프라인을 사용한 방법은 깔끔하고 불변한 방식으로 데이터를 변환할 수 있습니다.
그에 반해, for loop 방식은 하나의 for 구문 안에서 많은 연산이 실행될 경우 복잡한 순회를 발생시키고 변경 가능성이 열린 객체를 만들어야 하는 단점이 있습니다.

파이프라인 방식은 각 연산자마다 데이터 순회를 진행합니다.

따라서 위의 counts 함수는 map, filter, sorted 함수로 세 번의 데이터 순회가 발생합니다.
그에 반해 for loop 방식은 한 번의 데이터 순회만 발생합니다.
물론 근소하게 for loop 방식이 성능적 이점을 가지지만 대부분의 상황에서 파이프라인 방식도 성능적으로 충분합니다.

파이프라인 방식은 for loop과 달리 데이터 순회 도중에 멈출 수 없습니다.

파이프라인 방식에서는 완전한 순회를 돌아야 합니다.
for loop에서는 break, continue 키워드를 사용해 순회를 멈출 수 있습니다.
일반적으로 가독성이 높고 불변의 성격을 가진 파이프라인 방식이 성능보다 중요합니다.

**Mapping over a dictionary**

배열과 동일하게 딕셔너리 타입에도 map 함수를 적용할 수 있습니다.

커밋한 데이터를 튜플 배열로 저장하고 있을 때 튜플 배열을 딕셔너리로 변환하고, map을 사용해 딕셔너리의 value에 따라 새로운 문자열 배열로 변환해 보겠습니다.

먼저 uniqueKeysWithValues를 사용해 튜플 배열을 딕셔너리로 쉽게 변환할 수 있습니다.
uniqueKeysWithValues는 딕셔너리에서 제공하는 기능으로 아래와 같이 사용할 수 있습니다.

```swift
let commitStats = [
  (name: "Miranda", count: 30),
  (name: "Elly", count: 650),
  (name: "John", count: 0)
]
let commitsDict = Dictionary(uniqueKeysWithValues: commitStats)
print(commitsDict)  // ["Miranda": 30, "Elly": 650, "John": 0]
```

위의 코드처럼 Dictionary를 생성할 때 uniqueKeysWithValues를 사용해 튜플 배열을 딕셔너리로 변환할 수 있습니다.

[(name: "Miranda", counts: 30), (name: "Miranda", counts: 30) ...] 

하지만 위와 같이 딕셔너리의 Key로 사용될 값이 동일하게 두 개 이상인 튜플 배열의 경우 uniqueKeysWithValues를 통한 딕셔너리로 변환에 런타임 에러가 발생합니다.
이런 경우 uniqueKeysWithValues가 아닌 uniquingKeysWith을 사용해 딕셔너리를 생성할 수 있습니다.

아래 코드로 확인해 봅시다.

```swift
let arr = [("four", 4), ("three", 3), ("own", 1), ("two", 2), ("own", 10)]

var dic = Dictionary(arr, uniquingKeysWith: +)
// [“four": 4, "three": 3, "own": 11, "two": 2]

dic.merge([("two", 20)], uniquingKeysWith: +)
// ["four": 4, "three": 3, "own": 11, "two": 22]
```

튜플 배열을 uniqueKeysWithValues로 딕셔너리 객체를 만들었다면, 딕셔너리에 map을 사용하여 mapping하는 코드를 살펴 봅시다.

```swift
print(commitsDict)  // ["Miranda": 30, "Elly": 650, "John": 0]

let mappedKeysAndValues = commitsDict.map { (name: String, count: Int) -> String in
  switch count {
  case 0: return "\(name) isn't involved in the project."
  case 1..<100: return "\(name) isn't very active on the project."
  default: return "\(name) is active on the project"
  }
}

print(mappedKeysAndValues)  // ["Miranda isn't very active on the project", "Elly is active on the project", ...]
```

map을 딕셔너리와 함께 사용하면 위 코드와 같이 딕셔너리 요소의 key, value를 한 번에 클로저로 넘깁니다.
또한 딕셔너리 요소의 value 값만을 클로저로 넘길 수도 있습니다.
이때 우리는 map을 대신해서 mapValues를 사용하게 됩니다.

아래 코드로 확인해 봅시다.

```swift
let mappedValues = commitsDict.mapValues { (count: Int) -> String in
  switch count {
  case 0: return "Not involved in the project."
  case 1..<100: return "Not very active on the project."
  default: return "Is active on the project"
  }
}

print(mappedValues)  // ["Miranda": "Not very active on the project.", "Elly": "Is active on the project,...]
```

위 코드에서 볼 수 있듯이 map에서는 배열을 리턴하지만 mapValues에서는 딕셔너리를 리턴하고 있습니다.
mapValues에서는 Key 값은 유지되고 새로운 Value만을 부여하게 됩니다.

## Mapping over sequences

You saw before how you could map over Array and Dictionary types. These types implement a Collection and Sequence protocol.

아래 코드는 Range Sequence와 map을 함께 사용하여 mock data를 만드는 코드입니다.

```swift
let names = [
  "John",
  "Mary",
  "Elizabath"
]
let nameCount = names.count

let generatedNames = (0..<5).map { index in
  return names[index % nameCount]
}

print(generatedNames)  // ["John", "Mary", "Elizabath", "John", "Mary"]
```

map을 사용할 때 계속해서 기억해야 하는 부분은 map에서 새로운 Array를 리턴한다는 사실입니다.

## Mapping over optionals















