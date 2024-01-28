# Modeling data with enums

## This chapter covers
- How enums are an alternative to subclassing
- Using enums for polymorhism
- Learning how enums are "or" types
- Modeling data with enums instead of structs
- How enums and structs are algebraic types
- Converting structs to enums
- Safely handling enums with raw values
- Converting strings to enums to create robust code

## Or vs. and

열거형은 "or" 타입으로 여겨집니다. 
데이터를 모델링 할 때 "or" 성격이 보인다면 열거형을 고려합시다.

아래 코드는 채팅 서비스의 메세지 데이터를 모델링한 코드입니다.
메세지 데이터는 보내는 메세지, 채팅방 참여 메세지, 채팅방 퇴장 메세지, 풍선 이모티콘 중 하나의 상태를 가집니다.

먼저 구조체를 통해 메세지 데이터를 만든 코드를 살펴봅시다.

```swift
import Foundation

struct Message {
  let userId: String
  let contents: String?
  let date: Date

  let hasJoined: Bool
  let hasLeft: Bool
  let isBeingDrafted: Bool
  let isSendingBalloons: Bool
}

let joinMessage = Message(
  userId: "1", 
  contents: nil, 
  date: Date(), 
  hasJoined: true,
  hasLeft: false, 
  isBeingDrafted: false,
  isSendingBalloons: false
)

let brokenMessage = Message(
  userId: "1", 
  contents: "Hi there", 
  date: Date(), 
  hasJoined: true,
  hasLeft: true, 
  isBeingDrafted: false, 
  isSendingBalloons: false
)
```

위와 같이 구조체로 모델을 만드는 방식이 일반적으로 떠올리는 메세지 데이터 모델입니다.
열거형이 "or" 개념이라면 구조체는 "And" 개념입니다.

우리가 만드는 메세지 데이터는 본인이 보낸 텍스트, 채팅방 참여 텍스트, 채팅방 퇴장 텍스트, 풍선 이모티콘이어야 합니다.
하지만 구조체는 여러 형태의 값을 만들 가능성이 있습니다.
예를 들어 hasJoined와 hasLeft가 동시에 true인 요상한 객체를 만들 수 있습니다.
만약 메세지 데이터의 유효성을 검사하려 하더라도, 구조체의 내부값이 잘못됐다는 사실을 컴파일 타임이 아닌 런타임에 알 수 밖에 없습니다.

이는 결국 버그로 이어집니다. 

우린 열거형을 사용해 객체의 상태를 한정시키고 유효하지 않은 데이터를 컴파일 타임에 체크할 수 있습니다.
데이터 모델링에서 상호 배타적인(or) 성격을 가졌다면 구조체보다는 열거형을 고려합시다.
열거형과 함께 튜플을 사용하면 더 복잡한 데이터를 표현하기 쉽습니다. 

아래 코드로 확인해 봅시다.

```swift
import Foundation

enum Message {
  case text
  case draft
  case join
  case leave
  case ballon
}
```

```swift
import Foundation

// 2. with value (enum + tuple)
enum Message {
  case text(userId: String, contents: String, date: Date)
  case draft(userId: String, date: Date)
  case join(userId: String, date: Date)
  case leave(userId: String, date: Date)
  case ballon(userId: String, date: Date)
}

let textMessage = Message.text(userId: "2", contents: "Bonjour", date: Date())
let joinMessage = Message.join(userId: "2", date: Date())
```

이전 구조체와 달리 열거형은 다섯가지 케이스 중 하나를 선택했다면 해당 케이스에서 지원하는 튜플 속 프로퍼티만 요구합니다.
해당 요구사항을 따르지 않으면 당연히 컴파일 타임에 오류가 발생합니다.
열거형으로 메세지 데이터를 만들며 어떤 상황(case)에 어떤 데이터가 필요한지 명시하고 컴파일 타임에 검사할 수 있게 됩니다.

여기서 "or" 성격의 열거형에 강점이 드러납니다.

물론 열거형은 필연적으로 switch 구문을 요구합니다. 반복적인 switch 구문은 중복되는 코드를 발생시키기도 합니다.
우린 if case let 구문으로 반복적인 switch 구문을 줄일 수 있습니다.

if case let 구문을 사용해 반복적인 switch 문을 피할 수 있습니다.
아래 코드를 확인해 봅시다.

```swift
import Foundation

if case let Message.text(userId: id, contents: contents, date: date) = {
  print("Received: \(contents)")
}
```

위 코드에서 userId와 date 변수는 사용하지 않기 때문에 와일드카드(_)를 사용해 무시할 수 있습니다

```swift
import Foundation

if case let Message.text(_, contents: contents, _) = {
  print("Received: \(contents)")
}
```

살펴봤듯이 열거형은 컴파일 타임에 이점이 있습니다. 하지만 열거형의 case가 하나뿐이라면 구조체가 더 좋은 선택일 수 있습니다.

본인이 작성한 코드에 구조체가 보인다면 구조체 속 프로퍼티들을 사용되는 케이스 별로 그룹화해 봅시다.
아래 메세지 구조체의 프로퍼티를 text, draft, join, leave, ballon 케이스 중 사용되는 부분에 적절히 그룹화한걸 볼 수 있습니다.

```swift
struct Message {
  let userId: String
  let contents: String?
  let date: Date

  let hasJoined: Bool
  let hasLeft: Bool
  let isBeingDrafted: Bool
  let isSendingBalloons: Bool
}

enum Message {
  case text(userId: String, contents: String, date: Date)
  case draft(userId: String, date: Date)
  case join(userId: String, date: Date)
  case leave(userId: String, date: Date)
  case ballon(userId: String, date: Date)
}
```

구조체의 프로퍼티가 그룹화 된다면 열거형으로 구조체를 수정해 봅시다. 열거형이 더 좋은 선택지가 됩니다.

## Enums for polymorphism

Polymorphism(다형성) means that a single function, method, array, dictionary - you name it - can work with different types.

```swift
let arr: [Any] = [Date(), "aa", 789]
```

위 코드와 같이 Any 타입 배열로 다형성을 만들 수도 있습니다. 하지만 Any 타입의 무분별한 사용은 피하는게 좋겠습니다.
Any가 어떤 타입으로 표현될지 컴파일 타임에 알 수 없고 런타임에서야 알게 됩니다. 

Any 타입이 쓰이는 경우는 서버로부터 unknown data를 받을 때 정도입니다. 

열거형을 사용하면 다형성을 구현함과 동시에 컴파일 타임 안전성을 보장받을 수 있습니다.
열거형이 제공한는 컴파일 타임 안전성은 열거형에 있는 케이스의 대응 여부를 컴파일러가 체크하는 것입니다.
열거형의 케이스에 대응하지 않아도 컴파일 타임 에러가 발생하고 열거형 케이스가 요구하는 데이터를 전달하지 않아도 컴파일 에러가 발생합니다.

열거형이 컴파일 타임 안전성을 보장한다는건 알겠는데 열거형이 다형성을 이룬다는 건 무슨 이야기일까요?
Date 타입과 Range<Date> 타입을 같은 배열에 저장해야 하는 경우를 살펴봅시다.

```swift
import Foundation

// enum과 연관값
enum DateTpye {
  case singleDate(Date)
  case dateRange(Range<Date>)
}

let now = Date()
let hourFromNow = Date(timeIntervalSinceNow: 3600)

// enum을 통한 다형성
let dates: [DateType] = [
  DateType.singleDate(now),
  DateType.dateRange(now..<hourFromNow)
]

for dateType in dates {
  switch dateType {
  case .singleDate(let date): print("Date is \(date)")
  case .dateRange(let range): print("Range is \(range)")
  }
}
```

열거형과 함께 튜플을 사용해 데이터 타입이 다른 Date과 Range<Date>을 각 케이스 별 튜플에 넣어 다형성을 이루게 됩니다.

## Enums instead of subclassing

개발을 하다보면 클래스를 사용 서브클래싱으로 데이터 계층을 만드는 경우가 많습니다. 

하지만 이때 자식 클래스에 어울리지 않는 코드를 부모 클래스가 가졌다면 해당 코드를 어쩔 수 없이 자식 클래스에 강제해야 하는 경우가 생깁니다.
기능이 변경되거나 추가되었을 때 자식 클래스를 만들며 상속 받는 부모 클래스와 어울리지 않을 수 있습니다.

열거형을 사용하면 새로운 요구사항에 대응하기 더 쉽습니다.

여러 운동 객체(Run, Cylcle)를 생성한다고 가정해 봅시다.
서브 클래싱 방식으로 구현한다면 Workout 부모 클래스를 만들고 여러 운동 객체를 Workout 클래스의 자식 클래스로 만들것 입니다.
Workout 부모 클래스의 프로퍼티로 id, startTime, endTime, distance가 있습니다. 그리고 Run, Cycle 자식 클래스는 Workout 클래스의 프로퍼티가 모두 필요합니다. 
Workout 클래스는 부모 클래스로서 부족함이 없습니다.

하지만 id, repetitions, date 프로퍼티만 요구하는 Pushups 객체가 추가된다면 문제가 생깁니다.
Workout 부모 클래스에서 id를 제외한 모든 프로퍼티는 새로 추가된 Pushups 객체에 어울리지 않게 됩니다.

만약 여기서 Workout 부모 클래스를 수정한다면 id 프로퍼티를 제외한 나머지 프로퍼티는 삭제됩니다.
너무 많은 수정이 따르게 됩니다.

심지어 id를 요구하지 않는 abs 객체가 추가된다면 더 이상 Workout 부모 클래스는 부모 클래스로서 역할하지 못합니다.
서브클래싱 방식은 자식 클래스의 중복을 줄일 수 있지만 변경에 용이하지 못합니다.

열거형을 사용하면 서브 클래싱과 달리 변경에 용이합니다. 
아래 코드는 열거형을 사용해 운동 객체를 구현한 코드입니다.

```swift
enum Workout {
  case run(Run)
  case cycle(Cycle)
  case pushups(Pushups)
}
```

여기서 abs가 추가된다면 아래 코드와 같이 간단히 추가할 수 있습니다.

```swift
enum Workout {
  case run(Run)
  case cycle(Cycle)
  case pushups(Pushups)
  case abs(Abs)
}
```

물론 서브 클래싱을 사용한다면 공통 프로퍼티나 함수를 부모 클래스에 넣어 자식 클래스들에서의 중복을 줄일 수 있습니다. 
하지만 기능이 추가되거나 변경되면 부모 클래스를 리팩터링해야 합니다. 
열거형을 사용한다면 기능 확장에 추가적인 리팩터링이 필요 없습니다. 

These are trade-offs you'll have to make. If you can lock down your data model to a fixed, manageable number of cases, enums can be a good choice.

## Algebraic data types

Algebraic data에는 sum types와 product types이 있습니다. 
sum types은 "or"로 설명되고 열거형이 해당합니다. product types은 "and"로 설명되고 구조체와 튜플이 해당합니다. 
sum types의 열거형은 고정된 개수의 데이터를 표현합니다. product types은 여러 개수의 데이터를 표현합니다. 

데이터 모델링할 때 데이터가 표현할 수 있는 상태의 수를 체크하는 게 중요합니다. 
데이터가 표현할 수 있는 상태가 많을수록 표현할 수 있는 타입을 추론하기 어렵습니다.

```swift
enum PaymentType {
  case invoice
  case creditcard
  case cash
}

struct PaymentStatus {
  let paymentDate: Date?
  let isRecurring: Bool
  let paymentType: PaymentType
}
```

위와 같이 PaymentStatus를 만든다면 2가지(Bool)와 3가지(PaymentType)를 곱하여 6가지 변화를 가지는 데이터로 볼 수 있습니다.
이를 sum types인 열거형을 통해 리팩토링한다면 3가지 변화를 가지는 데이터로 바꿀 수 있습니다.

```swift
enum PaymentStatus {
  case invoice(paymentDate: Date?, isRecurring: Bool)
  case creditcard(paymentDate: Date?, isRecurring: Bool)
  case cash(paymentDate: Date?, isRecurring: Bool)
}
```

위와 같이 수정한다면 PaymentStatus 타입 하나만 다룰 수 있다는 장점이 있습니다. 

## A safer use of strings

string과 열거형을 함께 사용하는 경우는 흔합니다.
열거형에서 raw value를 추가할 수 있습니다. 
이때 모든 케이스스는 raw value를 가져야 합니다. 
raw value로는 String, Char, Int, float만 가능합니다.

```swift
enum Currency: String {
  case euro = "euro"
  case usd = "usd"
  case gbp = "gbp"
}
```

열거형을 raw value로 사용할 때는 아래와 같이 간략히 사용할 수도 있습니다.

```swift
enum Currency: String {
  case euro
  case usd 
  case gbp
}
```

지금까지 보던 열거형의 연관 값(튜플)들은 런타임에 정의됩니다. 하지만 열거형을 raw values로 사용하면 해당 값은 컴파일 타임에 정의됩니다.

열거형을 raw values와 사용할 때 코드 작성자에 의한 버그에 주의해야 합니다.
열거형을 raw values로 사용하면 올바르지 않은 값을 넣어도 컴파일러가 알아차리지 못합니다. 

아래 코드와 같이 euro를 eur로 잘못 적었다면 컴파일러는 euro를 eur로 인식해서 버그가 숨어들게 됩니다.

```swift
enum Currency: String {
  case euro = "eur"
  case usd 
  case gbp
}

let parameters = ["filter": currency.rawValue]
```

다음과 같은 버그를 방지하기 위해 currency 열거형형의 raw values를 무시하고 raw values가 필요할 때 데이터를 다시 생성하도록 아래와 같이 구현합니다.

```swift
enum Currency: String {
  case euro = "eur"
  case usd 
  case gbp
}

let parameters = [String: String]
switch currency {
  case .euro: parameters = ["filter": "euro"]
  case .usd: parameters = ["filter": "usd"]
  case .gbp: parameters = ["filter": "gbp"]
}
```

위 코드와 같이 열거형의 raw value를 무시하고 해당 값이 필요할 때마다 switch로 얻는다면 버그를 방지할 수 있습니다. 
다시 말해 열거형의 raw value를 사용하면 결과적으로 컴파일러 타임 안전성을 잃게 됩니다.

열거형을 사용할 때 string과 패턴 매칭하는 경우도 빈번합니다.

```swift
func iconName(for fileExtension: String) -> String {
  switch fileExtension {
  case "jpg": return "assetIconJpeg"
  case "bmp": return "assetIconBitmap"
  case "gif": return "assetIconGif"
  default: return "assertIconUnknown"
  }
}

iconName(for: "jpg")
iconName(for: "JPG")
```

위에서 jpg를 iconName 함수에 넣었을 때는 성공적으로 패턴 매칭이 됩니다. 
하지만 JPG에 매칭되는 결과는 없기에 버그가 발생하게 됩니다. 코드 작성자가 실수할 가능성까지 버그로 봅니다. 
이를 enum을 통해 개선할 수 있습니다. 

```swift
enum ImageType: String {
  case jpg
  case bmp
  case gif
	
  init?(rawValue: String) {
    switch rawValue.lowercasted() {
    case "jpg", "jpeg": self = .jpg
    case "bmp", "bitmap": self = .bmp
    case "gif", "gifv": self = .gif
    default: return nil
    }
  }
}

func iconName(for fileExtension: String) -> String {
  guard let imageType = ImageType(rawValue: fileExtension) else {
    return "assertIconUnknown"
  }

  switch fileExtension {
  case "jpg": return "assetIconJpeg"
  case "bmp": return "assetIconBitmap"
  case "gif": return "assetIconGif"
  default: return "assertIconUnknown"
  }
}

iconName(for: "jpg")
iconName(for: "JPG")
iconName(for: "JPEG")
```

이제는 ImageType에 case가 추가되면 컴파일러가 알려줍니다. 위의 코드는 대문자에 대응하고 다른 jpg, jpeg와 같이 다른 명칭에도 대응하도록 개선한 코드입니다.

## Summary

1. Enums are sometimes an alternative to subclassing, allowing for a flexible architecture.
2. Enums give you the ability to catch problem at compile time instead of runtime.
3. You can use enums to group properties together.
4. Enums are sometimes called sum types, based on algebratic data types.
5. Structs can be distributed over enums.
6. When working with enum's raw values, you forego catching problems at compile time.
7. Handling strings can be made safer by converting them to enums.
8. When converting a string to an enum, grouping cases and using a lowercased strings makes conversion easier.
