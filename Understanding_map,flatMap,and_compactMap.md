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
map은 기존 컨테이너의 요소에 대해 정의한 클로저로 매핑한 결과를 새로운 컨테이너로 반환합니다.

아래 코드는 map 함수의 정의부입니다.

```swift
func map<T>(_ transform: (Self.Element) throws -> T) rethrows -> [T]
```

정의부에서 transform은 매핑 클로저로, 컨테이너의 요소를 매개변수로 받아들이고 정의한 클로저의 형태에 맞게 변환된 값을 반환합니다.
또한 map 정의부에서 리턴하는 [T]는 컨테이너의 변환된 요소를 포함하는 배열을 반환합니다.

물론 for loop 방식(명령형)도 유용할 때가 많습니다.
하지만 for loop 방식에는 몇 가지 boilerplate가 존재할 수 있습니다.

아래에서 살펴볼 예시는 프로젝트에 참가한 사람들의 이름과 커밋 개수를 더 읽기 쉬운 형식으로 변환하는 상황입니다.

먼저 for loop 방식을 살펴봅시다. 
for loop 방식이 가진 boilerplate를 확인하고 map(함수형) 함수로 개선해 봅시다.

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
  var resolvedCommits = [String]()  // for loop을 위한 임시변수!
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

위의 for loop 방식의 resolveCounts 함수를 살펴보면 resolvedCommits 임시변수가 필요합니다.

이 부분이 for loop의 boilerplate라고 할 수 있습니다.
resolvedCommits 임시변수는 var로 선언되어 실수로라도 변형될 가능성이 있습니다.

for loop 대신 map을 사용하면 별도의 임시변수 없이 데이터 순회 이후 새로운 컨테이너를 리턴할 수 있습니다.
map을 사용하면 새로운 결과를 만들어 리턴하기 때문에 추가적인 임시변수가 필요하지 않습니다.

아래 코드는 위의 for loop 코드를, map을 사용한 코드로 리팩토링한 코드입니다.
map을 통해 데이터를 순회하며 각 요소가 클로저로 넘어갑니다.

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

위 코드는 아래와 같은 과정에 따라 데이터를 변환합니다.

1. You being with an array.
2. With map, you apply a function to each value inside the array.(in closure)
3. Map creates a new array with the transformed values inside.

map과 for loop가 동일한 결과를 만들지만, map은 for loop 보다 간결하고 불변한(immutable) 코드를 만들 수 있습니다.

## map is an abstraction

map은 컨테이너(arrays, dictionaries, optionals)에 상관없이 데이터를 변환할 수 있습니다.

이런 부분에서 map을 추상적 개념으로 볼 수 있습니다.

The map abstraction is called a functor.

map을 통해 컨테이너에 매핑(Mapping) 연산을 수행할 수 있습니다.
여기서 컨테이너란 우리가 흔히 사용하는 자료구조인 Array, Set 그리고 Dictionary와 같은 자료구조들을 일종의 컨테이너라고 할 수 있습니다.

매핑은 내부 원소의 값의 변형만 일어납니다. 
그 원소를 담고 있는 컨테이너의 변형은 일어나지 않습니다.
다시 말해, Array에 매핑 연산을 진행하였다고 Array가 Dictionary가 되는 컨테이너의 변형은 일어나지 않는다는 것입니다.

이렇게 값에 변형을 매핑할 수 있는 모든 것들을 Functor라고 합니다.

주목해야 할 점은 매핑으로 리턴되는 값은 다시 Functor로 감싼 후 반환된다는 것입니다!

매핑에 쓰이는 대표적인 함수가 map입니다. 옵셔널도 Functor의 한 종류입니다.

매핑으로 리턴된 값이 다시 Functor로 감싼 후 반환되기 때문에 옵셔널을 매핑했을 때도 옵셔널로 감싼 후 반환됩니다.

아래 링크를 통해 context, functor, monad에 대해 알아봅시다.

https://baked-corn.tistory.com/131

https://zeddios.tistory.com/449

## Creating a pipeline with map

데이터 변환에 있어 파이프라인은 각 단계를 표현하는 유용한 방법입니다.

지금부터 이름과 커밋 개수 데이터 중에서 커밋 개수가 0개인 데이터를 필터링하고 나머지 데이터 중 커밋 개수만 내림차순 정렬하는 함수를 만들어 봅시다.

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

for loop 방식을, pipeline을 활용한 방식으로 고쳐봅시다.

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
파이프라인을 구축하는 작업에 있어 map 함수는 새로운 객체를 리턴하기 때문에 굉장히 중요합니다.

파이프라인을 사용한 방법은 깔끔하고 불변한(immutable) 방식으로 데이터를 변환할 수 있습니다.

그에 반해, for loop 방식은 하나의 for 구문 안에서 많은 연산이 실행될 경우 복잡한 순회를 발생시키고 변경 가능성이 열린 객체를 만들어야 하는 단점이 있습니다.

파이프라인 방식과 for loop 방식에는 두 가지 차이점이 있습니다.

첫 번째 차이점은 for loop과 달리 파이프라인 방식은 연산자마다 데이터 순회를 진행합니다.

따라서 위의 counts 함수는 map, filter, sorted 함수로 세 번의 데이터 순회가 발생합니다.
그에 반해 for loop 방식은 한 번의 데이터 순회만 발생합니다.
물론 근소하게 for loop 방식이 성능적 이점을 가지지만 대부분의 상황에서 파이프라인 방식도 성능적으로 충분합니다.

두 번째 차이점은 for loop과 달리 파이프라인 방식은 데이터 순회를 도중에 멈출 수 없습니다.

파이프라인 방식에서는 완전한 순회를 돌아야 합니다.
for loop에서는 break, continue 키워드를 사용해 순회를 멈출 수 있습니다.
일반적으로 가독성이 높고 불변의(immutable) 성격을 가진 파이프라인 방식이 성능보다 중요합니다.

**Mapping over a dictionary**

배열과 동일하게 딕셔너리 타입에도 map 함수를 적용할 수 있습니다.

커밋한 데이터를 튜플 배열로 저장하고 있을 때 튜플 배열을 딕셔너리로 변환하고, map을 사용해 딕셔너리의 value에 따라 새로운 문자열 배열로 변환하려 할 때 map 함수를 적용해 봅시다.

먼저 **uniqueKeysWithValues**를 사용해 튜플 배열을 딕셔너리로 쉽게 변환할 수 있습니다.

uniqueKeysWithValues는 딕셔너리를 생성할 때(init) 튜플을 생성자에 넣어 딕셔너리로 변환할 수 있습니다.
아래 코드를 살펴봅시다.

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

[(name: "Miranda", counts: 20), (name: "Miranda", counts: 30) ...] 

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

튜플 배열을 uniqueKeysWithValues로 딕셔너리 객체를 만들었다면, 딕셔너리에 map을 사용하여 mapping하는 코드를 살펴봅시다.

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
이때 우리는 map을 대신해서 **mapValues**를 사용하게 됩니다.

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

mapValues에서는 딕셔너리의 value들을 가지고 새로운 컨테이너를 리턴하게 됩니다.

그렇다면 ["a", "b", "c"]와 같은 단일 배열을 [["a"], ["b"], ["c"]]와 같은 중첩 배열로 변형 시켜봅시다.
아래 코드로 살펴봅시다.

```swift
func makeSubArrays<T>(_ arr: [T]) -> [[T]] {
  return arr.map { [$0] }
}

makeSubArrays(["a", "b", "c"])  // [[a], [b], [c]]
makeSubArrays([100, 50, 1])  // [[100], [50], [1]]
```

이번에는 영화 정보를 저장하고 있는 딕셔너리 구조를 아래와 같이 변환하는 방법을 구현합시다.

```swift
let moviesAndRatings: [String: Float] = ["Home Alone 4": 1.2, "Who FramedRoger Rabbit?": 4.6, "Star Wars": 2.2]
let moviesHumanRadable = transformRating(moviesAndRatings)
print(moviesHumanRadable)  // ["Home Alone 4": "Weak", "Who FramedRoger Rabbit?": "Excellent", "Star Wars": "Average"]
```

[String: Float] 딕셔너리를 [String: String] 딕셔너리로 변환하기 때문에 mapValue를 사용할 수 있습니다.
아래 코드를 살펴봅시다.

```swift
func transformRating<T>(_ dict: [T: Float]) -> [T: String] {
  return dict.mapValues { (rating) -> String in
    switch rating {
    case ..<1: return "Very weak"
    case ..<2: return "Weak"
    case ..<3: return "Average"
    case ..<4: return "Good"
    case ..<5: return "Excellent"
    default: fatalError("Unknown rating")
    }
  }
}

let moviesAndRatings: [String: Float] = ["Home Alone 4": 1.2, "Who FramedRoger Rabbit?": 4.6, "Star Wars": 2.2]
let moviesHumanRadable = transformRating(moviesAndRatings)  // ["Home Alone 4": "Weak", "Who FramedRoger Rabbit?": "Excellent", "Star Wars": "Average"]
```

그렇다면 moviesAndRatings 딕셔너리를 아래와 같은 문자열 배열로 변환하는 경우를 살펴봅시다.

```swift
let movies: [String: Float] = ["Home Alone 4": 1.2, "Who FramedRoger Rabbit?": 4.6, "Star Wars": 2.2]
// ["Home Alone 4 (Weak)", "Who FramedRoger Rabbit? (Excellent)", "Star Wars (Average)"] 이렇게 변환됩니다.
```

위와 같은 문자열 배열로 변환하기 위해 아래와 같이 구현했습니다.

```swift
let movies: [String: Float] = ["Home Alone 4": 1.2, "Who FramedRoger Rabbit?": 4.6, "Star Wars": 2.2]

func convertRating(_ rating: Float) -> String {
  switch rating {
  case ..<1: return "Very weak"
  case ..<2: return "Weak"
  case ..<3: return "Average"
  case ..<4: return "Good"
  case ..<5: return "Excellent"
  default: fatalError("Unknown rating")
  }
}

let movieDescriptions = movies.map { (tuple) in
  return "\(tuple.key) (\(convertRating(tuple.value))"
}

print(movieDescriptions)  // ["Home Alone 4 (Weak)", "Who FramedRoger Rabbit? (Excellent)", "Star Wars (Average)"]
```

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

map을 사용할 때 기억해야 하는 부분은 map에서 새로운 Array를 리턴한다는 사실입니다.

## Mapping over optionals

옵셔널에서도 map을 사용할 수 있습니다.

지금까지 배열을 mapping 하는 것은 배열의 값들을 변환하고 새로운 컨테이너를 리턴하지만 옵셔널을 mapping 하는 것은 단일 값을 변환합니다.

옵셔널과 map을 함께 사용할 때 여러 이점을 얻을 수 있습니다.

먼저 옵셔널을 mapping 하면 특별한 언래핑 없이 옵셔널 속 값을 map 클로저 내에서 언래핑 된 상태로 사용할 수 있습니다.
다시 말해, 옵셔널 언래핑을 매핑을 통해 지연시킬 수 있습니다.

두 번째로 map의 클로저 안에서는 옵셔널을 다루는지 몰라도 됩니다.

옵셔널과 map을 함께 사용할 때 옵셔널이 nil인 경우 map 연산은 동작하지 않고 nil을 리턴합니다.
옵셔널에 값이 있으면 언래핑 된 값을 map 클로저 안으로 전달합니다.
따라서 map 클로저 안에서 호출하는 함수 입장에서 옵셔널의 여부를 알 필요가 없습니다.

세 번째로 map은 결과적으로 새로운 배열을 리턴하기 때문에 for loop에서 필요한 var 타입 임시 변수가 map을 사용하면 필요하지 않습니다.
따라서 var 타입 임시 변수 없이 수동 언래핑 없이 옵셔널을 사용할 수 있습니다.

마지막으로 옵셔널과 map을 사용해 언래핑 없이 반복적인 chaining이 가능합니다.

아래 코드를 살펴봅시다.

```swift
class Cover {
  let image: UIImage
  let title: String?

  init(image: UIImage, title: String?) {
    self.image = image
    self.title = title.map(removeEmojis).map { $0.trimmingCharacters(in: .whitespaces) }
  }
}
```

물론 map 클로저 밖에서(애플리케이션 어딘가에서) Cover 클래스의 title 프로퍼티에 접근하기 위해서는 옵셔널 언래핑이 필요합니다.
하지만 옵셔널을 mapping 할 때는 map 클로저 안에서만 옵셔널을 언래핑한 것처럼 사용할 수 있습니다.

위의 title.map(removeEmojis).map { $0.trimmingCharacters(in: .whitespaces) } 코드를 보면, 어디에도 언래핑 코드가 없지만 옵셔널 속 값을 사용하고 있음을 알 수 있습니다.

**When to use map on optionals**

Imagine that you're creating a printing service where you can print out books with social media photos and their comments. 
Unfortunatley, the printing service doesn't support special characters, such as emojis, so you need to strip emojis from the texts.

위의 상황을 만족하는 함수를 만들어 봅시다.

옵셔널과 map을 함께 사용하기 전에 문자열에서 이모지를 지우는 removeEmojis 함수를 만들어 봅시다.

아래 코드로 확인해 봅시다.

```swift
func removeEmojis(_ string: String) -> String {
  var scalars = string.unicodeScalars
  scalars.removeAll(where: isEmoji)
  return String(scalars)
}

func isEmoji(_ scalar: Unicode.Scalar) -> Bool {
  // 생략...
  return true
}
```

removeEmojis 함수에서 문자열 입력을 unicodeScalars 키워드를 통해 Scalar로 변환해 이모지인지 아닌지 확인합니다.

이제 옵셔널을 mapping 하는 방법을 살펴봅시다.

removeEmojis 함수를 사용해 아래와 같은 포토북 커버의 요구사항을 따르는 과정에서 옵셔널과 map을 사용하려 합니다.

포토북의 커버는 항상 이미지를 가지고, 제목은 있을 수도 없을 수도 있습니다.
여기서 제목 텍스트가 이모지를 가지고 있다면 앞에서 구현했던 removeEmojis 함수로 이모지를 삭제해야 합니다.

먼저 옵셔널과 map 없이 포토북 커버 클래스를 구현해 보겠습니다.

```swift
class Cover {
  let image: UIImage
  let title: String?

  init(image: UIImage, title: String?) {
    self.image = image

    var cleanedTitle: String? = nil
    if let title = title {
      cleanedTitle = removeEmojis(title)
    }
    self.title = cleanedTitle
  }
}
```

map 없이 윱셔널을 사용할 때 옵셔널 변수를 초기화하기 위해 임시 변수인 cleanTitle를 만들었고 if let을 통해 옵셔널 title을 언래핑했습니다.
if let 안에서 removeEmojis 함수를 호출했고 마지막에 self.title 변수에 임시 변수인 cleanTitle 값을 넣었습니다.

if let과 옵셔널을 함께 사용하는 일반적인 코드를 옵셔널 mapping을 통해 확실히 줄일 수 있습니다.

If you were to map over an optional, you'd apply the removeEmojis() function on the unwrapped value inside map(if there is one).

If the optional is nil, the mapping operation is ignored!

아래 코드는 옵셔널 mapping을 통해 위의 Cover 클래스의 코드를 개선한 코드입니다.

```swift
class Cover {
  let image: UIImage
  let title: String?

  init(image: UIImage, title: String?) {
    self.image = image
    self.title = title.map { (string: String) -> String in
      return removeEmojis(string)
    }
  }
}
```

title.map에서 map이 기본적으로 새로운 값을 리턴하기 때문에 더 이상 리턴에 사용할 임시 변수를 선언할 필요가 없습니다.

map 클로저에서 추가적인 옵셔널 언래핑 과정이 필요 없이 옵셔널 내부 값을 사용할 수 있습니다.
또한 title이 nil이었을 때 nil이 map 클로저로 넘어가지 않고 단순히 nil이 리턴됩니다.

하지만 옵셔널을 mapping 했을 때 결과적으로 return 하는 값은 다시 옵셔널을 씌워서 리턴합니다.
이는 위에서 살펴봤던 기존 Functor로 감싸서 리턴되는 성격을 따르는 것입니다.
물론 nil은 Optional(nil)이 아닌 nil로 리턴됩니다.

옵셔널과 map을 함께 사용할 때 조금 더 축약된 코드를 아래와 같이 쓸 수 있습니다.

```swift
self.title = title.map { removeEmojis($0) }
```

심지어 map에서 클로저를 생성하지 않고도 아래와 같이 사용할 수 있습니다.
클로저를 생성하지 않기 때문에 {}가 아닌 ()를 사용합니다. 

```swift
self.title = title.map(removeEmojis)
```

결과적으로 아래와 같은 Cover 클래스를 구현할 수 있게 됩니다.

```swift
class Cover {
  let image: UIImage
  let title: String?

  init(image: UIImage, title: String?) {
    self.image = image
    self.title = title.map(removeEmojis)
  }
}
```

## Grokking flatMap

지금까지 map을 살펴봤다면 이제 flatMap을 살펴봅시다.
flatMap은 스위프트에서 중요하게 쓰입니다.

flatMap은 map 연산 이후 flatten(평탄화) 연산을 추가로 진행합니다.
다시 말해 map의 연산과 동일하지만, map 연산 이후 추가적인 flatten 연산이 진행됩니다.

flatMap에서 진행되는 평탄화 작업은 Optional(Optional(4))을 Optional(4)로 [[1,2,3],[1,2]]를 [1,2,3,1,2]로 중첩 구조를 단일 구조로 푸는 것을 말합니다.

그렇다면 flatMap이 쓰이는 상황은 어떤 상황일까요?

아래 코드는 flatMap이 아닌 map을 사용한 코드입니다.
어떤 문제가 있는지 살펴봅시다.

```swift
let receivedData = ["url": "https://www.clubpenguin.com"]
let path: String? = receivedData["url"]

let url = path.map { (string: String) -> URL? in
  let url = URL(string: string)  // Optional(https://www.clubpenguin.com)
  return url
}
print(url)  // Optional(Optional(https://www.clubpenguin.com))
```

map을 사용했을 때 url은 옵셔널이 중첩된 구조를 가집니다.

URL(string: string) 자체가 옵셔널인 동시에 path가 옵셔널로 리턴되는 결과에도 동일한 Functor인 옵셔널로 감싸서 리턴되며 중첩 옵셔널 구조를 가지게 됩니다.
(매핑 이후 리턴되는 값은 다시 Functor로 감싼 후 반환됨을 기억합시다!)

물론 중첩 옵셔널을 피하고자 강제 언래핑을 사용할 수 있습니다. (하지만 지양해야 합니다.)

아래 코드로 확인해 봅시다.

```swift
let receivedData = ["url": "https://www.clubpenguin.com"]
let path: String? = receivedData["url"]

let url = path.map { (string: String) -> URL in
  let url = URL(string: string)!  // https://www.clubpenguin.com
  return url
}
print(url)  // Optional(https://www.clubpenguin.com)
```

하지만 강제 언래핑으로 인해 URL로 변형되는 string이 잘못된 URL 주소일 경우 크래시가 발생합니다.
언제나 강제 언래핑은 크래시를 발생할 가능성이 있기 때문에 지양해야 합니다.

앞에서 소개했듯이 flatMap은 map 연산 이후 중첩 구조를 푸는 flatten 연산을 진행합니다.

flatMap을 사용해 중첩 옵셔널 문제를 해결해 봅시다.

```swift
let receivedData = ["url": "https://www.clubpenguin.com"]
let path: String? = receivedData["url"]

let url = path.flatMap { (string: String) -> URL? in
  let url = URL(string: string)  // Optional(https://www.clubpenguin.com)
  return url
}
print(url)  // Optional(https://www.clubpenguin.com)
```

flatMap은 map 연산 이후 리턴되는 옵셔널 중첩 중 한 겹을 제거하게 됩니다.

flatMap은 중첩 옵셔널을 해결하기 위한 좋은 방법이 될 수 있습니다.

**Fighting the pyramid of doom**

옵셔널이 중첩되면 옵셔널 언래핑을 위한 들여쓰기가 중첩되어 피라미드 형태의 코드 구조를 만들게 됩니다.

피라미드 형태의 코드 구조는 옵셔널 중첩의 결과물이기 때문에 피해야 할 코드 구조로 flatMap을 사용해 피할 수 있습니다.

half 함수는 짝수가 들어올 경우 이등분한 값에 옵셔널로 감싸서 리턴하고 홀수가 들어올 경우 nil을 리턴합니다.

피라미드 형태의 코드 구조를 살펴보기 전에 Int 값을 이등분하고 옵셔널 Int를 리턴하는 함수 half를 구현하고, half 함수를 반복적으로 사용할 때 발생하는 피라미드 코드 형식을 살펴봅시다.

half 함수가 옵셔널을 리턴하기 때문에 half 함수를 반복적으로 사용하면 옵셔널 언래핑을 위한 중첩 if let 코드로 인해 피라미드 코드 형식이 발생하게 됩니다.

앞으로 피라미드 코드 형식을 살펴보고 이를 flatMap으로 개선하는 방법도 살펴봅시다.

```swift
// half 함수입니다.
func half(_ int: Int) -> Int? {
  guard int % 2 == 0 else { return nil }
  return int % 2
}
print(half(4))  // Optional(2)
print(half(5))  // nil
```

위의 half 함수를 반복적으로 호출하면 당연히 반복적인 언래핑 코드(if let)가 필요합니다.

아래 코드로 확인해 봅시다.

```swift
var value: Int? = nil
let startValue = 80
// 피라미드 코드
if let halvedValue = half(startValue) {
  print(halvedValue)  // 40
  value = halvedValue
  if let halvedValue = half(halvedValue) {
    print(halvedValue)  // 20
    value = halvedValue
    if let halvedValue = half(halvedValue) {
      print(halvedValue)  // 10
      if let halvedValue = half(halvedValue) {
        value = halvedValue
      } else {
        value = nil
      }
    } else {
      value = nil
    }
  } else {
    value = nil
  }
} 

print(value)  // Optional(5)
```

half 함수를 계속해서 사용한다면 리턴되는 옵셔널을 계속해서 언래핑해야 하므로 위 코드와 같은 피라미드 코드가 발생합니다.
물론 flatMap을 사용하기 전에 if let을 아래 코드와 같이 축약해서 사용할 수 있습니다.

```swift
let startValue = 80
var endValue: Int? = nil

if let firstHalf = half(startValue),
  let secondHalf = half(firstValue),
  let thirdHalf = half(secondHalf),
  let fourthHalf = half(thirdHalf) {
  endValue = fourthHalf
}
print(endValue)  // Optional(5)
```

하지만 중첩 if let 구문은 각 if let마다 변수들을 이름 짓기 어렵습니다.
과연 firstHalf, secondHalf 등 같은 변수명들은 아무 의미 없는 변수명을 만들게 됩니다.

또한 모든 함수가 half 함수처럼 하나의 입력에 하나의 결과를 리턴을 갖지 않습니다.
따라서 옵셔널이 중첩으로 발생하는 상황에서 if let은 적합하지 않습니다.

flatMap을 사용해 중첩 옵셔널 언래핑에 의한 피라미드 코드 구조를 해결해 봅시다.

flatMap은 map과 동일한 동작을 하지만 매핑이 끝난 이후 중첩된 구조를 제거합니다.

아래 순서로 flatMap이 동작합니다.

1. Start with an optional.
2. With flatmap, you apply a function to the value inside an optional. This function will itself return an optional.
3. You end up with a nested(중첩) optional.
4. The nested optional is flattened(평탄화) to a regular optional!

코드를 통해 flatMap이 동작하는 방법을 살펴봅시다.

```swift
let startValue = 8
let four = half(startValue)  // Optional(4)
let two = four.flatMap { (int: Int) -> Int? in
  print(int)  // 4
  let nestedTwo = half(two)
  print(nestedTwo)  // Optional(2)
  return nestedTwo
}

print(two)  // Optional(2)
```

flatMap은 옵셔널이 중첩되더라도 매핑 이후 평탄화 작업을 통해 단일 옵셔널로 계속해서 chainig 할 수 있습니다.

아래 코드로 flatMap을 사용해 반복적인 half 함수 호출 과정을 살펴봅시다.
flatMap을 통해 더 이상 피라미드 형태의 코드를 만들지 않을 수 있습니다. 

```swift
let startValue = 40
let twenty = half(startValue)  // Optional(20)
let five =
    twenty
        .flatMap { (int: Int) -> Int? in
          print(int)  // 20
          let ten = half(int)
          print(ten)  // Optional(10)
          return ten
        }.flatMap { (int: Int) -> Int? in
          print(int)  // 10
          let five = half(10)
          print(five)  // Optional(5)
          return five
}

print(five)  // Optional(5)
```

flatMap도 map과 동일하게 동작하기 때문에 옵셔널에 flatMap 연산을 했을 때 옵셔널의 내부 값 즉 언래핑 된 값이 flatMap의 클로저로 들어옵니다. 물론 옵셔널이 nil이라면 클로저가 동작하지 않고 nil을 리턴합니다.

만약 map이었다면 클로저가 끝났을 때 다시 옵셔널로 감싸서 리턴하고 끝나겠지만, flatMap이기 때문에 옵셔널로 감싼 이후 중첩된 옵셔널을 제거하는 동작까지 수행합니다.

flatMap을 통해 중첩 if let 구문을 피하면 if let에 의해 만들어야 했던 무의미한 임시 변수를 더 이상 추적할 필요 없습니다.

map과 같이 nil이 flatMap으로 들어오면 이후의 동작은 이루어지지 않고 nil을 리턴하게 됩니다.

아래 코드로 nil이 flatMap으로 들어왔을 때를 살펴봅시다.

```swift
let startValue = 40
let twenty = half(startValue)  // Optional(20)
let someNil =
    twenty
        .flatMap { (int: Int) -> Int? in
          print(int)  // 20
          let ten = half(int)
          print(ten)  // Optional(10)
          return ten
        }.flatMap { (int: Int) -> Int? in
          print(int)  // 10
          let five = half(10)
          print(five)  // Optional(5)
          return five
        }.flatMap { (int: Int) -> Int? in
          print(int)  // 5
          let someNilValue = half(int)
          print(someNilValue)  // nil
          return someNilValue
        }.flatMap { (int: Int) -> Int? in
          return half(int)  // This code is never called because you're calling flatMap on a nil value!
}

print(someNil)  // nil
```

위의 코드에서 네 번째 flatMap은 동작하지 않습니다.
세 번째 flatMap에서 nil이 리턴되어 네 번째 flatMap이 동작하지 않는 것입니다.

nil을 통해 flatMap의 순회를 중단하는 능력 또한 flatMap의 특징이자 장점입니다.

flatMap을 아래 코드와 같이 더 짧게 사용할 수도 있습니다.

```swift
let endResult =
    half(80)
        .flatMap(half)
        .flatMap(half)
        .flatMap(half)

print(endResult)  // Optional(5) 
```

다른 예시를 두고 flatMap을 적용해 봅시다.

"imagine a scenario where you'd find a user by an id, find the user's favorite product, and see if any related product exists."

map과 flatMap을 사용해 데이터 변환과 함께 chaining 하는 과정에서 불필요한 임시변수들과 if let 중첩을 피할 수 있습니다.
아래 코드로 살펴봅시다.

```swift
let alternativeProduct =
    findUser(3)
    .flatMap(findFavoriteProduct)
    .flatMap(findRelatedProduct)
    .map { product in
      product.description.trimmingCharacters(in: .whitespaces)
    }
```

다시 말해 flatMap을 사용하면 중복 옵셔널에 의한 중복 언래핑(if let)과 임시 변수 없이 코드를 깔끔히 적을 수 있습니다.
하지만 flatMap 또한 map과 동일한 기능을 하고 매핑 이후 추가적인 평탄화 작업을 하게 됩니다.

## flatMapping over collections
위에서는 flatMap을 옵셔널에 사용했습니다.
flatMap은 옵셔널 외에도 Collection 타입들과도 함께 사용할 수 있습니다.

flatMap이 중첩 옵셔널을 평탄화했듯이 Collection 타입들과 사용했을 때도 동일한 개념의 평탄화 작업을 map 연산 이후 진행합니다.

Optional(Optional(3))을 Optional(3)로 평탄화했듯이 [[1, 2],[3, 4]] 배열을 [1, 2, 3, 4]로 평탄화합니다.
flatMap이 Collection 타입과 동작하는 과정을 살펴봅시다.

1. You have a regular array [2, 3].
2. With flatMap, you apply a function to each value inside the array. This function turns a value into [value, value], another array.
3. You end up with subarrays [[2, 2], [3, 3]]. flatMap flattens the nested arrays to [2, 2, 3, 3].

아래 코드는 flatMap을 Collection 타입 중 하나인 배열에 적용한 코드입니다.

```swift
let repeated = [2, 3].flatMap { (value: Int) -> [Int] in
  return [value, value]
}

print(repeated)  // [2, 2, 3, 3]
```

위의 [2, 3] 배열에 map을 사용했다면 [[2, 2], [2, 2]] 배열이 리턴되지만, flatMap에 의해 중첩 배열이 평탄화되어 [2, 3] 배열이 리턴하게 됩니다.

flatMap이 중첩 배열을 단일 배열로 평탄화하는 코드를 한 번 더 살펴봅시다.

```swift
let stringsArr = [["I", "just"], ["want", "to"], ["learn", "about", ["protocols"]]
let flattenedArray = stringsArr.flatMap { $0 }

print(flattenedArray)  // ["I", "just", "want", "to", "learn", "about", "protocols"]
```

**flatMapping over strings**

String도 Collection 타입 중 하나입니다.
당연히 flatMap과 String도 함께 쓰입니다.

아래 코드는 Swift 문자열을 S-w-i-f-t 문자열로 변환하는 함수를 구현한 코드입니다.

```swift
"Swift".interspersed("-")  // S-w-i-f-t

extension String {
  func interspersed(_ element: Character) -> String {
    let characters = self.flatMap { (char: Character) -> [Character] in
      return [char, element]  // [S, -], [w, -] ...
    }.dropLast()
    return String(characters)  // [S, -, w, -, ...]
  }
}
```

만약 interspersed 함수에 map을 사용했다면 [S, -, w, -, ...] 배열이 아닌 [[S, -], [w, -], ...] 형태의 중첩 배열이 리턴됩니다.
하지만 flatMap의 평탄화 작업에 의해 중첩 배열이 [S, -, w, -, ...] 단일 배열로 리턴됩니다.

위의 interspersed 함수를 아래와 같이 축약할 수 있습니다.

```swift
extension String {
  func interspersed(_ element: Character) -> String {
    let characters = self.flatMap { return [$0, element] }.dropLast()
    return String(characters)
  }
}
```

**Combining flatMap with map**

flatMap과 함께 map을 사용할 때 적은 코드로도 강력한 기능을 수행하는 코드를 구현할 수 있습니다.

포커 카드 덱을 만들어야 할 상황을 가정하고 flatMap과 map을 함께 사용한 코드를 구현해 봅시다.

아래 코드를 살펴봅시다.

```swift
let suits = ["Hearts", "Clubs", "Diamonds", "Spades"]
let faces = ["2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"]

var deckOfCards = suits.flatMap { suit in
  faces.map { face in
    (suit, face)
  }
}
deckOfCards.shuffle()
print(deckOfCards)  // [("Diamonds", "5"), ("Hearts", "8"), ...]
```

With flatMap you remove one layer of nesting so that you nestly end up with an array of tuples.

**Using compactMap**

compactMap은 중복된 레이어를 평탄화하는 flatMap과 달리 Collection 속의 nil을 필터링합니다!

String을 URL로 변환할 때 잘못된 String에 의해 nil이 생성되었을 때 Collection 속 nil을 compactMap으로 필터링해 봅시다.

아래 코드로 살펴봅시다.

```swift
let wrongUrl = URL(string: "OMG SHOES")
print(wrongUrl)  // nil
let properUrl = URL(string: "https://www.swift.org")
print(properUrl)  // Optional(hppts://www.swift.org)

let strings = [
  "https://www.duckduckgo.com",
  "https://www.twitter.com",
  "OMG SHOES",
  "https://www.swift.org"
]
// map 사용 시 nil이 필터링되지 않습니다.
let optionalUrls = strings.map { URL(string: $0) }
print(optionalUrls)  // [Optional(https://www.duckduckgo.com), Optional(https://www.twitter.com), nil, Optional(https://www.swift.org)]

// compactMap 사용 시 nil이 필터링됩니다.
let urls = strings.compactMap(URL.init)
print(urls)  // [Optional(https://www.duckduckgo.com), Optional(https://www.twitter.com), Optional(https://www.swift.org)]
```

물론 map이나 compactMap이 효과적일 수 있지만, for loop 또한 좋은 선택지가 될 수 있습니다.
for loop은 break, continue, return 등을 통해 데이터 순회를 원하는 지점에 끊을 수 있습니다.

또한 열거형의 패턴 매칭에 for loop와 case let의 조합은 자주 사용됩니다.
**for case let**을 통해 배열의 nil을 필터링할 수 있습니다.

아래 코드를 살펴봅시다.

```swift
let optionalUrls: [URL?] = [
  URL(string: "https://www.duckduckgo.com"),
  URL(string: "Bananaphone"),
  URL(string: "https//www.swift.org")
]

for case let url? in optionalUrls {
  print("The url is \(url)")
}

// Output:
// The url is https://www.duckduckgo.com
// The url is https//www.swift.org
```

**Nesting or chaining**

flatMap과 compactMap 모두 중첩해서 사용하여 chaining 할 수 있습니다. 굉장히 유용한 방법입니다.

아래 코드로 flatMap과 compactMap을 중첩해서 사용하는 방법을 살펴봅시다.

```swift
let string = "abc"
let results = string.flatMap { a -> [(Character, Character)] in
  string.compactMap { b -> (Character, Character)? in
    if a == b {
      return nil
    } else {
      return (a, b)
    }
  }
}
print(results)  // [("a", "b"), ("a", "c"), ("b", "a"), ("b", "c"), ("c", "a"), ("c", "b")]
```

By adding map and flatMap to your toolbelt, you can harness their powers and add powerful immutable abstractions to your code in a succinct manner!!

## Summary
- The map and flatMap methods are concepts taken from the functional programming world.
- The map method is an abstraction called a functor.
- A functor represents a container - or context - of which you can transform its value with map.
- The map method is defined on many types, including Array, Dictionary, Sequence, and Collections protocol, and Optional.
- The map method is a crucial element when transforming data inside a pipeline.
- Imperative-style programming is a fine alternative to functional, style programming.
- Imperative-style programming can be more performant. In contrast, functional-style programming can involve immutable transformations and can sometimes be more readable and show clearer intent.
- The flatMap method is a flatten operation after a map operation.
- With flatMap you can flatten a nested optional to a single optional.
- With flatMap you can sequence operations on an optional in an immutable way.
- Once an optional is nil, map and flatMap ignores any chained operations.
- If you return nil from a flatMap, you can short-circuit operations.
- With flatMap you can transform arrays and sequences in powerful ways with very little code.
- With compactMap you can filter nils out of arrays and sequences of optionals.
- You can also filter nils with an imperative style by using a for loop.
- You can nest flatMap and compactMap operations for the same results.
- On collections and sequences, you can combine flatMap with map to combine all their values.
