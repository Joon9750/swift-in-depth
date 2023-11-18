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

enum은 "or" 타입으로 여겨집니다. 데이터를 모델링 할 때 "or"이 눈에 보인다면 enum을 고려합시다.

chat application 서비스의 message 데이터를 모델링 해봅시다. 
message는 본인이 보낸 text, 채팅방 참여 text, 채팅방 퇴장 text, 풍선 이모티콘 중 하나씩 보낼 수 이는 chat application 서비스입니다.

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
	isSendingBalloons: false)

let brokenMessage = Message(
	userId: "1", 
	contents: "Hi there", 
	date: Date(), 
	hasJoined: true,
	hasLeft: true, 
	isBeingDrafted: false, 
	isSendingBalloons: false)
```

위와 같이 구조체로 모델을 만드는 방식이 일반적으로 떠올리는 Message 데이터 모델입니다.
우리가 만드는 Message 데이터는 유저가 보내는 text 또는 채팅방 참여 text 또는 채팅방 퇴장 text 또는 풍선 이모티콘이어야 합니다.
하지만 구조체는 여러 형태의 값을 만들 가능성이 있습니다. 이 가능성은 버그로 이어집니다. 

enum이 "or" 개념이라면 구조체는 "And" 개념인 것입니다.

brokenMessage처럼 hasJoined와 hasLeft가 동시에 true일 수 있는 말도 안되는 상황이 생길 수 있습니다.
심지어 구조체 타입의 message 유효성을 검사하더라도 유효하지 않다면 이를 런타임에 알게 됩니다.
이럴 때 데이터 모델링에 enum을 사용하면 버그를 줄이고 유효하지 않는 message는 컴파일 타임 체크할 수 있습니다.

데이터 모델링에서 상호 배타적인(or) 성격을 가졌다면 구조체보다는 enum을 고려해봅시다.
enum과 함께 tuple을 사용하면 더 복잡한 데이터를 표현하기 쉽습니다. 

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

// 2. with value
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

enum과 tuple을 함께 사용하여 각 case들이 associated value(연관값)을 갖게 됩니다. 이는 프로퍼티가 어떤 케이스에 어울릴지 명확히 보여줍니다.

enum으로 데이터를 만들면 당연히 switch문이 등장할 것입니다. 하지만 switch문은 하나의 데이터를 얻기 위해 모든 case에 해당하는 코드를 적어야 합니다. 이는 중복되는 코드를 발생시킵니다.

if case let 구문을 사용해 반복적인 switch문을 피할 수 있습니다.

```swift
import Foundation

if case let Message.text(userId: id, contents: contents, date: date) = {
	print("Received: \(contents)")
}
```

위 코드에서 userId와 date 변수는 사용하지 않기 때문에 와일드 카드(_)로 신경 쓰지 않을 수 있습니다.

```swift
import Foundation

if case let Message.text(_, contents: contents, _) = {
	print("Received: \(contents)")
}
```

enum은 컴파일 타임에 이점이 있습니다. 하지만 enum의 case가 하나뿐이라면 구조체가 더 좋은 선택일 수 있습니다.

구조체를 사용한다면 구조체의 프로퍼티를 그룹화 해봅시다. 그룹화가 된다면 enum으로 구조체를 수정해봅시다. enum이 더 좋은 선택지일것 입니다.

## Enums for polymorphism

Polymorphism(다형성) means that a single function, method, array, dictionary - you name it - can work with different types.

```swift
let arr: [Any] = [Date(), "aa", 789]
```

위 코드와 같이 Any 타입 배열로 다형성을 만들 수도 있습니다. 하지만 이상적이지 못합니다.
Any가 어떤 타입으로 표현될지 컴파일 타임에 알 수 없고 런타임에 알게 됩니다. 
Any 타입이 필요한 경우는 서버로부터 unknown data를 받을 때 정도입니다. 

enum을 사용하면 다형성을 구현함과 동시에 complie-time safety를 보장받을 수 있습니다.
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

enum을 통해 다른 타입을 배열에 넣음과 동시에 compile-time safety를 얻을 수 있습니다. 
여기서 말하는 compile-time safety는 enum에 있는 case의 대응 여부를 컴파일러가 체크하는 것입니다. 
만약 enum에 새로운 case가 추가된다면 컴파일러가 switch문의 missing case를 체크해줍니다. 

## Enums instead of subclassing

class를 사용하여 subclassing으로 데이터 계층을 만드는 경우가 많습니다. 
하지만 이때의 한계는 자식 클래스에 어울리지 않는 코드를 부모 클래스에서 강제하는 경우가 생깁니다. 
만약 자식 클래스가 추가될 필요가 생겼을 때 부모 클래스가 추가될 자식 클래스와 어울리지 않을 수 있습니다. 
subclassing보다 enum을 사용하면 새로운 요구사항에 대응하기 쉽습니다. 

Workout 부모 클래스에 Run, Cycle, Pushups가 자식 클래스로 있다고 가정해봅시다.
새로운 abs가 추가될 때 subclassing은 부모 클래스 Workout의 자식 클래스로 abs가 추가됩니다.
부모클래스 Workout에서 제공하는 기능들이 abs와 어울리지 않을 수 있습니다. 
따라서 subclassing은 변경에 용이하지 못합니다.

enum을 사용하면 아래와 같이 변경에 용이합니다. 
아래와 같이 subclassing이 아닌 enum을 사용해 구현했습니다.

```swift
enum Workout {
	case run(Run)
	case cycle(Cycle)
	case pushups(Pushups)
}
```

이때 abs가 추가된다면 아래 코드와 같이 간단히 추가할 수 있습니다.

```swift
enum Workout {
	case run(Run)
	case cycle(Cycle)
	case pushups(Pushups)
	case abs(Abs)
}
```

물론 subclassing을 사용한다면 공통 프로퍼티나 함수를 부모 클래스에 넣어 자식 클래스들에서의 중복을 줄일 수 있습니다. 
하지만 기능 확장이 생기면 부모 클래스를 리팩터링 해야합니다. 
enum을 사용한다면 기능 확장에 추가적인 리팩터링이 필요 없습니다. 
앞에서 봤듯이 class와 마찬가지로 enum을 통해 추상화도 가능합니다.

These are trade-offs you'll have to make. If you can lock down your data model to a fixed, manageable number of cases, enums can be a good choice.

## Algebraic data types

Algebraic data에는 sum types와 product types이 있습니다. 
sum types은 "or"로 설명되고 enum이 해당됩니다. product types은 "and"로 설명되고 struct, tuple이 해당됩니다. 
sum types의 enum은 고정된 개수의 데이터를 표현합니다. product types은 여러 개수의 데이터를 표현합니다. 

데이터 모델링할 때 데이터가 표현할 수 있는 상태의 수를 체크하는게 중요합니다. 
데이터가 표현 가능한 상태가 많을수록 표현 가능한 타입을 추론하기 어렵습니다.

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

위와 같이 PaymentStatus를 만든다면 2가지(Bool)과 3가지(PaymentType)를 곱하여 6가지 변화를 가지는 데이터로 볼 수 있습니다.
이를 sum types인 enum을 통해 리팩토링한다면 3가지 변화를 가지는 데이터로 바꿀 수 있습니다.

```swift
enum PaymentStatus {
	case invoice(paymentDate: Date?, isRecurring: Bool)
	case creditcard(paymentDate: Date?, isRecurring: Bool)
	case cash(paymentDate: Date?, isRecurring: Bool)
}
```

위와 같이 수정한다면 PaymentStatus 타입 하나만 다룰 수 있다는 장점이 있습니다. 

## A safer use of strings

string과 enum을 사용하는 경우는 흔합니다.
enum에서 raw value를 추가할 수 있습니다. 
이때 모든 case는 raw value를 가져야 합니다. 
raw value로는 String, Char, Int, float만 가능합니다.

```swift
enum Currency: String {
	case euro = "euro"
	case usd = "usd"
	case gbp = "gbp"
}
```

enum을 raw value로 사용할 때는 아래와 같이 간략히 사용할 수도 있습니다.

```swift
enum Currency: String {
	case euro
	case usd 
	case gbp
}
```

지금까지보던 enum의 연관값들은 런타임에 정의됩니다. 하지만 enum을 raw values로 사용하면 해당 값은 컴파일 타임에 정의됩니다.

enum을 raw values와 사용할 때 코드 작성자에 의한 버그에 주의해야 합니다.
enum을 raw values로 사용하면 올바르지 않은 값을 넣어도 컴파일러가 알아차리지 못합니다. 
예를 들어 euro를 eur로 잘못 적었다면 컴파일러는 euro를 eur로 인식해서 버그가 숨어들게 됩니다.

```swift
enum Currency: String {
	case euro = "eur"
	case usd 
	case gbp
}

let parameters = ["filter": currency.rawValue]
```

위와 같은 버그를 방지하기 위해 currency enum의 raw values를 무시하고 raw values가 필요할 때 데이터를 다시 생성하도록 아래와 같이 구현합니다.

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
```

위 코드와 같이 enum의 raw value를 무시하고 해당 값이 필요할 때마다 switch로 얻는다면 버그를 방지할 수 있습니다. 
다시 말해 enum의 raw value를 사용하면 결과적으로 컴파일러의 검사를 잃게 됩니다.

enum을 사용할 때 string과 매칭하는 경우도 빈번합니다.

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

위에서 jpg를 함수에 넣었을 때는 성공적으로 매칭이 됩니다. 
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

이제는 ImageType에 case가 추가돠면 컴파일러가 알려줍니다. 위의 코드는 대문자에 대응하고 다른 명칭에도 대응하도록 개선한 코드입니다.

## 정리

1. Enums are sometimes an alternative to subclassing, allowing for a flexible architecture.
2. Enums give you the ability to catch problem at compile time instead of runtime.
3. You can use enums to group properties together.
4. Enums are sometimes called sum types, based on algebratic data types.
5. Structs can be distributed over enums.
6. When working with enum's raw values, you forego catching problems at compile time.
7. Handling strings can be made safer by converting them to enums.
8. When converting a string to an enum, grouping cases and using a lowercased strings makes conversion easier.
