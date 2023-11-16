# Modeling data with enums

## Or vs. and

enum은 "or" 타입으로 여겨집니다. 데이터를 모델링 할 때 "or"이 눈에 보인다면 enum을 고려할 수 있습니다.

chat application 서비스의 message 데이터를 모델링 해봅시다. 
message는 본인이 보낸 text, 채팅방 참여 text, 채팅방 퇴장 text일 수 있습니다.
추가적으로 text 하나만 보낼 수 있고 풍선 이모티콘도 보낼 수 있는 chat 서비스입니다.

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

let joinMessage = 
Message(
userId: "1", 
contents: nil, 
date: Date(), 
hasJoined: true,
hasLeft: false, 
isBeingDrafted: false, 
isSendingBalloons: false
)

let brokenMessage = 
Message(
userId: "1", 
contents: "Hi there", 
date: Date(), 
hasJoined: true,
hasLeft: true, 
isBeingDrafted: false, 
isSendingBalloons: false
)
```

위와 같이 구조체로 모델을 만드는 방식이 일반적으로 떠올리는 Message 데이터 모델입니다.

우리가 만드는 Message 데이터는 유저가 보내는 text 또는 채팅방 참여 text 또는 채팅방 퇴장 text 또는 풍선 이모티콘이어야 합니다.

하지만 구조제는 여러 형태의 값을 만들 가능성이 있다. 하지만 이 가능성은 버그로 이어집니다. enum이 "or" 개념이라면 구조체는 "And" 개념인 것입니다.

brokenMessage처럼 hasJoined와 hasLeft가 동시에 true일 수 있는 말도 안되는 상황이 생길 수 있기 때문에 버그를 유발합니다.
심지어 구조체 타입의 message 유효성을 검사하더라도 유효하지 않다면 이를 런타임에 알게 됩니다.

이럴 때 데이터 모델링에 enum을 사용하면 버그를 줄이고 유효하지 않는 message를 컴파일 타임에 알 수 있습니다.

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

enum과 tuple을 함께 사용하며 각 case들이 associated value(연관값)을 갖게 됩니다. 이는 프로퍼티가 어떤 케이스에 어울릴지 명확히 보여줍니다.

enum으로 데이터를 만들면 당연히 switch문이 등장할 것입니다. 하지만 switch문은 하나의 데이터를 얻기 위해 모든 case에 해당하는 코드를 적어야합니다. 이는 중복되는 코드를 발생시킵니다.
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
하지만 구조체를 사용한다면 구조체의 프로퍼티를 그룹화 해봅시다. 그룹화 된다면 enum을 사용할 가능성이 생깁니다.

## Enums for polymorphism
 