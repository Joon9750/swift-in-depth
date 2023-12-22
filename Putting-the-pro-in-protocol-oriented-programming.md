# Putting the pro in protocol-oriented programming

## This chapter covers
- The relationship and trade-offs between generics and using protocols as types
- Understanding associated types
- Passing around protocols with associated types
- Storing and constraining protocols with associated types
- Simplifying your API with protocol inheritance

## Runtime versus compile time
프로토콜은 크게 런타임과 컴파일 타임에 동작하는 것으로 나뉩니다.
'제네릭을 제약할 때의 프로토콜'과 '단일 타입으로 쓰이는 프로토콜'로 나뉩니다.
'제네릭을 제약할 때의 프로토콜'은 컴파일 타임에 타입을 결정하고 '단일 타입으로 쓰이는 프로토콜'의 경우 런타임에 타입을 결정합니다.

지금부터 코인 포토폴리오를 구현하며 프로토콜에 대해 살펴봅시다.
코인 포토폴리오는 다양한 코인에 대응할 수 있어야 합니다.

다양한 코인에 대응하는 코인 포토폴리오를 구현하기 위해서는 열거형을 떠올릴 수 있습니다.
하지만 모든 코인에 대응해야하기 때문에 열거형에 너무 많은 case들이 나오기 때문에 열거형은 좋은 선택지는 아닙니다.

이때 우리는 프로토콜을 통해 코인 포토폴리오를 구현하여 여러 종류의 코인에 대응하도록 추상화 할 수 있습니다.
아래 코드로 살펴봅시다.

```swift
import Foundation

protocol CryptoCurrency {
  var name: String { get }
  var symbol: String { get }
  var holdings: Double { get set }
  var price: NSDecimalNumber? { get set }
}

struct Bitcoin: CryptoCurrency {
  let name = "Bitcoin"
  let symbol = "BTC"
  var holdings: Double
  var price: NSDecimalNumber?
}

struct Ethereum: CryptoCurrency {
  let name = "Ethereum"
  let symbol = "ETH"
  var holdings: Double
  var price: NSDecimalNumber?
}
```

CryptoCurrency 프로토콜을 통해 코인들이 공통적으로 가지는 프로퍼티를 선언합니다.

프로토콜의 프로퍼티는 항상 var로 선언되어야 합니다.
프로토콜을 따르는 구현부에서 프로퍼티를 var 또는 let으로 구현이 가능합니다.
만약 프로토콜의 프로퍼티가 { get set }이라면 조회/수정이 가능한 변수이기 때문에 구현부에서 해당 프로퍼티를 var로 구현해야 합니다.

코인 포토폴리오는 코인 프로퍼티를 가지고 있습니다.
코인 프로퍼티를 가지는 Portfolio 클래스를 만들어 봅시다.
코인 프로퍼티는 CrypoCurrency를 채택한 상태이고 코인 포토폴리오는 여러 종류의 코인에 대응해야 합니다.

먼저 '프로토콜을 제네릭 제약'으로 사용할 때의 코드를 살펴봅시다.
아래 코드를 확인해 봅시다.

```swift
final class Portfolio<Coin: CryptoCurrency> {
  var coins: [Coin]

  init(coins: [Coin]) {
    self.coins = coins
  }

  func addCoin(_ newCoin: Coin) {
    coins.append(newCoin)
  }
}
```

위 코드에 문제가 보이십니까?
Chapter 6. Generics에서 다루었던 내용입니다.
프로토콜을 통한 제네릭 제약과 함께 서브 클래싱이 발생하는 상황에서는 제네릭 타입으로 쓰이는 타입이 커스텀 타입일 경우 상속 관계를 잃게 됩니다. 

따라서 Portfolio 클래스의 제네릭 타입으로 CrytoCurrency 프로토콜을 따르는 타입 중 하나를 넣게 되면, 처음 넣은 데이터 타입으로 제네릭 타입이 컴파일 타임에 고정됩니다.
제네릭의 데이터 타입이 고정된 이후에는 CrytoCurrency 프로토콜을 따르더라도 고정된 타입과 같은 타입이 아닌 이상 컴파일 에러가 발생합니다.

```swift
let coins = [
  Ethereum(holdings: 4, price: NSDecimalNumber(value: 500)),
  // if we mix coins, we can't pass them to Portfolio
  // Bitcoin(holdings: 4, price: NSDecimalNumber(value: 6000))
]
let portfolio = Portfolio(coins: coins)

let btc = Bitcoin(holdings: 3, price: nil)
portfolio.addCoin(btc)  // error: cannot convert value of type 'Bitcoin' to expected argument type 'Ethereum'

// 타입을 확인해 봅시다.
print(type(of: portfolio))  // Portfolio<Ethereum>, 프로토콜을 제네릭 제약에 사용했을 때 컴파일 타임에 제네릭의 타입이 Ethereum로 고정됩니다. 
print(type(of: portfolio.coins))  // Array<Ethereum>, 프로토콜을 제네릭 제약에 사용했을 때 컴파일 타임에 제네릭의 타입이 Ethereum로 고정됩니다. 
```

제네릭과 프로토콜을 함께 사용했을 때 제네릭이 어떤 타입을 다루는지 컴파일 타임에 알 수 있다는 장점이 있습니다.
컴파일러의 성능도 최적화할 수 있습니다.

하지만 우린 여러 코인 타입을 함께 담고 싶습니다. 제네릭과 프로토콜을 함께 사용하면 컴파일 타임에 타입이 고정되고 상속 관계도 잃기 때문에 요구사항을 충족할 수 없습니다.
그렇다면 어떻게 프로토콜을 통해 여러 코인 타입을 함께 담을 수 있을까요?

제네릭 없이 프로토콜을 하나의 타입으로 사용하면 여러 코인 타입에 대응하도록 Portfolio 클래스를 만들 수 있습니다.
제네릭 없이 쓰이는 프로토콜은 런타임에 타입이 결정되기 때문입니다. 또한 상속 관계도 잃지 않습니다.

아래 코드는 제네릭의 제약으로 쓰이던 프로토콜을 하나의 타입으로 쓰이는 프로토콜로 변경한 코드입니다.

```swift
// Before : 제네릭 제약으로 쓰인 프로토콜
final class Portfolio<Coin: CryptoCurrency> {
  var coins: [Coin]
  // ...생략
}

// After : 타입으로 쓰인 프로토콜
final class Portfolio {
  var coins: [CryptoCurrency]
  // ...생략
}

let portfolio = Portfolio(coins: [])
let coins: [CryptoCurrency] = [
  Ethereum(holdings: 4, price: NSDecimalNumber(value: 500)),
  Bitcoin(holdings: 4, price: NSDecimalNumber(value: 6000))
]
portfolio.coins = coins  // '타입으로 쓰이는 프로토콜'로 서브 타입까지 수용 가능합니다.

// 타입을 확인해 봅시다.
print(type(of: portfolio))  // Portfolio
let retrievedCoins = porfolio.coins
print(type(of: rerievedCoins))  // Array<CryptoCurrency>
```

제네릭 없이 타입으로 쓰이는 프로토콜은 런타임에 동작하며 서브 타입까지 모두 대응할 수 있습니다.
보다 유연한 코드로 생각됩니다.

앞서 '제네릭을 제약하는 프로토콜'은 coins의 타입이 고정되었지만 '타입으로 쓰이는 프로토콜'의 경우 Array<CryptoCurrency>로 추상화 되어있습니다. 
만약 런타임에 동작하는 프로토콜을 원하다면 프로토콜을 타입이나 인터페이스로 사용하면 됩니다.

하지만 '타입으로 쓰이는 프로토콜'로 추상화 할 경우에도 단점이 있습니다.

만약 portfolio.coin에 추가 되는 Bitcoin 타입 객체가 CryptoCurrency protocol에서 제공하는 함수 외에 추가적인 함수가 있을 때, 
Array<CryptoCurrency>로 추상화 된 상태에서는 Bitcoin 타입의 추가적인 함수에 접근할 수 없습니다.
해당 함수에 접근하기 위해서는 CryptoCurrency 타입을 Bitcoin 타입으로 형 변환해야 합니다.
하지만 이는 안티패턴으로 이어질 수 있고 CryptoCurrency 타입을 따르는 코인의 종류가 많아지면 대응하기 어려워질 수 있으니 주의해야 합니다.

앞에서 보았듯이 '제네릭 타입 제약에 쓰이는 프로토콜'은 컴파일 타임에 타입이 결정되며 결정된 타입이 외에 어떤 타입도 허용하지 않습니다.
그렇다면 '제네릭 타입 제약에 쓰이는 프로토콜'은 어떤 경우에 더 좋은 선택지가 될까요?

비트 코인을 함수에 넘겨 가장 최신 가격 동일한 타입의 코인을 리턴 받는 상황에 '제네릭 타입 제약에 쓰이는 프로토콜'의 사용해 유리한 점을 살펴봅시다.

```swift
// 타입으로 쓰이는 프로토콜
func retrievePriceRunTime(coin: CryptoCurrency, completion: ((CryptoCurrency) -> Void)) {
  // ...생략
  var copy = coin
  copy.price = 6000
  completion(copy)
}

// 제네릭 타입 제약으로 쓰이는 프로토콜
func retrievePriceCompileTime<Coin: CryptoCurrency>(coin: Coin, completion:((Coin) -> Void) {
  // ...생략
  var copy = coin
  copy.price = 6000
  completion(copy)
}

let btc = Bitcoin(holdings: 3, price: nil)
retrievePriceRunTime(coin: btc) { (updatedCoin: CryptoCurrency) in  // 런타임 전까지 updatedCoin의 정확한 타입을 모릅니다.
  print("Updated value runtime is \(updatedCoin.price?.doubleValue ?? 0)")
}

retrievePriceCompileTime(coin: btc) { (updatedCoin: Bitcoin) in  // 컴파일 타임에 updatedCoin의 타입이 Bitcoin으로 결정되었다. 
  print("Updated value runtime is \(updatedCoin.price?.doubleValue ?? 0)")
}
```

'제네릭 제약으로 사용된 프로토콜'은 컴파일 타임에 타입이 결정되어 클로저로 들어올 타입을 이미 알고 있습니다.
'타입으로 사용된 프로토콜'은 런타임에서야 타입이 결정되기 때문에 Bitcoin 타입에 새로 추가된 프로퍼티나 함수에 접근하기 위해서는 CryptoCurrency 타입에서 Bitcoin 타입으로 형 변환을 해야 합니다.
하지만 '제네릭 제약으로 사용된 프로토콜'의 경우 이미 컴파일 타임에 Bitcoin 타입으로 확정되기 때문에 Bitcoin 타입의 새로운 프로퍼티나 함수에 추가적인 형 변환 없이 접근 가능합니다.

결과적으로 '제네릭 제약으로 사용된 프로토콜'과 '타입으로 사용된 프로토콜' 모두 trade-off가 존재합니다.
'타입으로 사용된 프로토콜'은 여러 타입에 대응할 수 있지만 런타임에 타입이 결정되어 특정 자식 타입에 추가된 함수나 프로퍼티에 접근하기 위해서는 형 변환을 필요로 합니다.
그에 반해 '제네릭 제약으로 사용된 프로토콜'은 컴파일 타임에 타입이 결정되어 상속 관계를 잃으며 여러 타입에 대응하기는 어렵지만 컴파일 타임에 타입이 결정되어 성능적 이점과 추가적인 형 변환을 필요로 하지 않습니다.

'제네릭 제약으로 사용된 프로토콜'이 생각하기 어렵지만, 더 좋은 선택입니다.

## The why of associated types

프로토콜은 추상화에 자주 쓰입니다. 
하지만 프로토콜과 함께 연관 값(associated types)을 사용하면 더 추상적인 코드를 구현할 수 있습니다.
우린 연관 값(associated types)을 가진 프로토콜을 PATs로 줄여 부릅니다.

연관 값 없이 프로토콜 만으로 데이터 모델링을 할 수 있습니다.
하지만 한계가 있습니다.
프로토콜 만으로 데이터 모델링을 했을 때 발생하는 문제를 먼저 살펴봅시다. (해당 문제를 프로토콜의 연관 값을 통해 해결할 것입니다.)

아래 예시는 이메일을 손님에게 보내거나 데이터 베이스를 다루거나 이미지 사이즈를 조절하는 등 성격들이 다른 '일'을 추상화하는 프로토콜로 만들었습니다.

```swift
protocol Worker {
  @discardableResult
  func start(input: String) -> Bool
}

class MailJob: Worker {
  func start(input: String) -> Bool {
    // send mail to email address
  }
}
```

위 코드에서 @discardableResult 키워드는 함수의 리턴 값을 무시할 수 있도록 만듭니다.
@discardableResult 키워드가 붙은 함수의 리턴은 사용하지 않아도 경고를 띄우지 않습니다.

Worker 프로토콜의 start 함수는 입력과 출력 데이터 타입이 String과 Bool로 고정되어 있습니다. 
하지만 '일'들은 start 함수를 공통으로 가질 수 있지만 start 함수의 입력과 출력 데이터 타입이 다양할 것입니다.
위의 Worker 프로토콜의 start 함수로는 다양한 타입에 대응하지 못합니다.
만약 파일을 삭제하는 FileRemover 객체가 URL을 입력 받아 [String]을 리턴하는 start 함수가 필요하다면 위와 같은 Worker 프로토콜의 start 함수로는 대응할 수 없습니다.

단순한 프로토콜(연관 값이 없는 프로토콜)은 적용 가능한 범위가 넓지 못합니다.

위와 같이 함수(start) 동작의 과정은 동일하지만, 동작에 사용되는 타입이 다를 경우 프로토콜에 연관 값을 추가해서 구현할 수 있습니다.
프로토콜과 연관 값을 함께 사용하는 방법을 살펴보기 전에 불완전한 두 가지 방법을 먼저 살펴봅시다.
불완전한 방법이지만 생각해볼만한 접근 방법입니다.

첫 번째 방법은 Worker 프로토콜 start 함수의 입력과 출력 데이터 타입을 모두 프로토콜로 만드는 방법입니다.  
Input, Output 프로토콜을 만들고 start 함수의 모든 입력과 출력 데이터 타입이 Input 또는 Output 프로토콜을 따르도록 합니다.
그리고 Worker 프로토콜의 start 함수 입력에 쓰이는 데이터 타입은 Input 프로토콜을 따르고 출력에 쓰이는 데이터 타입은 Output 프로토콜을 따르도록 만듭니다.

물론 동작은 할 것입니다. 아래 코드를 확인해봅시다.

```swift
protocol Input {}
protocol Output {}

protocol Worker {
  @discardableResult
  func start(input: Input) -> Output
}
```

하지만 start 함수에 쓰이는 모든 데이터 타입이 Input 또는 Output 프로토콜을 따르도록 하는 방법은 boilerplate 코드를 유발합니다.
심지어 Input, Output 프로토콜에 새로운 프로퍼티나 함수가 추가된다면 이를 따르는 모든 타입에서 추가적인 구현이 필요합니다.

두 번째 방법은 프로토콜에 제네릭을 추가하는 방법입니다.
아래 코드로 살펴봅시다.

```swift
protocol Worker<Input, Output> {
  @discardableResult
  func start(input: Input) -> Output
}
```

프로토콜에 제네릭을 추가하여 Worker 프로토콜을 채택한 구현부에서 Input과 Output이 어떤 타입이 될지 결정하도록 구현했습니다.
굉장히 좋은 접근이지만 아쉽게도 스위프트에서 프로토콜과 제네릭의 사용을 허락하지 않습니다.
위 코드는 다음과 같은 컴파일 에러를 발생시킵니다.

error: protocols do not allow generic parameters; use assciated types instead

스위프트에서는 하나의 타입에 특정 프로토콜을 채택하고 요구사항을 구현하는 행위의 중복을 허용하지 않습니다.
다시 말해 MailJob 클래스가 Worker 프로토콜을 채택할 때 한 가지의 구현만 허용합니다.

스위프트에서 프로토콜과 제네릭의 사용을 허락하지 않는 이유도 하나의 타입에 한 번의 프로토콜 채택과 구현만을 허용하기 때문입니다.
만약 프로토콜과 제네릭이 함께 사용된다면 아래와 같은 코드가 작성될 것입니다.

```swift
// Not supported: Implementing a generic Worker.
class MailJob: Worker<String, Bool> {
  // ...생략
}

class MailJob: Worker<Int, [String]> {
  // ...생략
}
```

스위프트에서 같은 타입(MailJob)이 Worker 프로토콜을 단일 채택하고 구현해야 합니다.
위와 같이 같은 타입이 Worker 프로토콜에 대해 여러 구현부를 가진다면 컴파일 에러가 발생합니다.

컴파일 되지는 않는 코드지만 프로토콜과 제네릭을 함께 사용하는 방법은 좋은 접근입니다.
결과적으로 프로토콜과 제네릭스러운 요소(연관 값)가 함께 사용되어야 Worker 프로토콜이 여러 타입의 '일'에 대응할 수 있습니다.
여기서 프로토콜과 함께 쓰일 제네릭스러운 요소가 바로 연관 값(associatedtype)입니다. 

그렇다면 우리의 결론이었던 프로토콜과 연관 값을 사용해 Worker 프로토콜을 만들어 봅시다.
start 함수의 입력과 출력을 연관 값으로 만들어 Worker 프로토콜이 여러 '일'에 대응하도록 만듭니다.
프로토콜에서 associatedtype 키워드로 연관 값을 선언하고 구현부에서는 typealias 키워드를 사용해 연관 값의 타입을 지정합니다.

```swift
// 연관 값을 사용한 프로토콜
protocol Worker {
  associatedtype Input  // just like generics
  associatedtype Output  // just like generics
  
  @discardableResult
  func start(input: Input) -> Output
}

class MailJob: Worker {
  typealias Input = String  // the Input associated type is defined as String
  typealias Output = Bool  // the Output associated type is defined as Bool

  func start(input: String) -> Bool {
    // send mail to email address
  }
}
```

associatedtype은 제네릭과 비슷한 성격을 가지지만 제네릭과 달리 프로토콜 내부에 정의됩니다.
Worker 프로토콜의 associatedtype을 통해 MailJob은 Input을 string 타입, Output을 Bool 타입으로 구현할 수 있고 FileRemover는 Input을 URL 타입, Output을 [string] 타입으로 구현할 수 있습니다.

프로토콜의 연관 값을 통해 프로토콜을 따르는 타입이 특정 프로토콜에 대한 유일한 구현을 가지는 동시에 여러 타입에 대응하도록 만들 수 있습니다.

위 코드의 MailJob 클래스는 typealias 키워드로 프로토콜 연관 값의 타입을 확정하고 있지만, 컴파일러가 MailJob 클래스의 구현부를 통해 타입이 추론 가능하다면 typealias 구문을 생략할 수 있습니다.
아래 코드는 typealies 키워드룰 생략한 코드입니다. 
FileRemover 클래스의 start 함수에서 입력과 출력 데이터 타입을 모두 명시하고 있기 때문에 typealias 키워드 없이도 컴파일러가 Worker 프로토콜의 연관 값을 추론할 수 있습니다.

```swift
class FileRemover: Worker {
  // typealias Input = String 
  // typealias Output = Bool 

  func start(input: URL) -> [String] {
    do {
      var results = [String]()
      let fileManager = FileManager.default
      let fileURLs = try fileManager.contentsOfDirectory(a: input, includingPropertiesForKeys: nil)

      for fileURL in fileURLs {
        tryfileManager.removeItem(at: fileURL)
        results.append(filedURL.absoluteString)
      }
      return results
    } catch {
      print("Clearing direcory failed.")
      return []
    }
  }
}
```

스위프트에서는 프로토콜과 연관 값은 함께 자주 사용됩니다.
IteratorProtocol, Sequence, Collection 프로토콜들이 대표적으로 Element 연관 값을 가지고 있습니다.

또한 여러 프로토콜에서 Self를 볼 수 있는데 Self도 마찬가지로 연관 값입니다.
아래 코드는 Self 연관 값을 가진 Equatable 프로토콜 코드입니다.

```swift
public protocol Equatable {
  static func == (lhs: Self, rhs: Self) -> Bool
}
```

스위프트 내부적으로 프로토콜과 연관 값을 함께 사용한 경우 외에도 다양한 용도에 사용됩니다.
프로토콜을 따르는 타입에서 여러 타입에 대응해야 할 때 프로토콜과 연관 값이 함께 사용됩니다.
프로토콜과 연관 값이 함께 사용되는 상황들을 살펴봅시다.

- A Recording protocol - Each recording has a duration, and it could also suport scrubbing through time via a seek() method, but the actual data could be different for each implementation, such as an audio file, video file, or YouTube stream.
- A Service protocol - It loads data; one type could return JSON data from an API, and another could locally search and return raw string data.
- A Message protocol - It's on a social media tool that tracks posts. In one implementation, a message represents a Tweet; in another, a message represents a Facebook direct message; and in another, it could be a message on WhatsApp.
- A SearchQuery protocol - It resembles database queries, where the result is different for each implementation.
- A Paginator protocol - It can be given a page and offset to browse through a database. Each page could represent some data. Perhaps it has some users in a user table in a database, or perhaps a list of files, or a list of products inside a view.

서브 클래싱 방식의 코드를 프로토콜 방식으로 코드로 고쳐 봅시다.
아래 코드로 확인해 봅시다.

```swift
class AbstractDamage {}

class AbstractEnemy {
  func attack() -> AbstractDamage {
    fatalError("This method must be implemented by subclass.")
  }
}

class Fire: AbstractDamage {}
class Imp: AbstractEnemy {
  override func attack() -> Fire {
    return Fire()
  }
}

class BluntDamage: AbstractDamage {}
class Centaur: AbstractEnemy {
  override func attack() -> BluntDamge {
    return BluntDamage()
  }
}
```

위의 서브 클래싱 기반 코드를 아래 프로토콜 기반 코드로 수정했습니다. 

```swift
protocol AbstractEnemy {
  associatedtype damage

  func attack() -> damage
}

struct Fire {}
class Imp: AbstractEnemy {
  func attack() -> Fire {
    return Fire()
  }
}

struct BluntDamage {}
class Centaur: AbstractEnemy {
  func attack() -> BluntDamage {
    return BluntDamage()
  }
}
```

## Passing protocols with associated types

위에서는 프로토콜의 연관 값을 살펴보았다면, 지금부터는 연관 값을 가진 프로토콜을 함수의 인자로 넘기는 방법을 살펴봅시다.

연관 값을 가진 프로토콜을 함수에 넘길 때 프로토콜의 연관 값에 직접 접근할 수 있습니다.
또한 함수에서 제네릭 제약처럼 프로토콜 연관 값의 타입에 제약을 걸 수도 있습니다.

먼저 함수로 연관 값을 가진 프로토콜을 넘기는 코드를 살펴봅시다.

```swift
func runWorker<W: Worker>(worker: W, input: [W.Input]) {
  input.forEach { (value: W.Input) in
    worker.start(input: value)
  }
}

let mailJob = MailJob()
runWorker(worker: mailJob, input: ["grover@sesamestreetcom", "bigbird@sesamestreet.com"])

let fileRemover = FileRemover()
runWorker(worker: fileRemover, input: [
  URL(fileURLWithPath: "./cache", isDirectory: true),
  URL(fileURLWithPath: "./tmp", isDirectory: true)
])
```

위 코드에서 input이 W.Input 타입으로 지정하듯이 프로토콜의 연관 값에 직접 접근할 수 있습니다. 
input이 W.Input 타입으로 선언했기 때문에 worker.start(input: value)로 값을 전달할 수 있습니다.
프로토콜의 연관 값은 컴파일 타임에 타입이 결정됩니다.

제네릭 함수에서 where 절을 사용했듯이 프로토콜의 연관 값을 입력 받는 함수도 where 절로 입력을 받을 수 있습니다.
또한 where 절에서 연관 값의 타입에 제약을 걸 수 있습니다. 이는 제네릭 제약과 비슷한 느낌입니다.
하지만 문법적으로 차이가 있으니 아래 코드를 살펴봅시다.

```swift
final class User {
  let firstName: String
  let secondName: String
  init(firstName: String, lastName: String) {
    self.firstName = firstName
    self.secondName = secondName
  }
}

func runWorker<W>(worker: W, input: [W.Input])
  where W: Worker, W.Input == User {
    // associatedtype에 제약을 걸어 User Input에 특수화된 함수를 구현할 수 있었습니다.
    input.forEach { (user: W.Input) in
      worker.start(input: user)
      print("Finished processing user \(user.firstName) \(user.lastName)")
    }
}
```

By constraining an associated type, the function is specialized to work only with users as input!!

지금까지는 연관 값을 가진 프로토콜을 함수에 넘기는 방법을 살펴봤고 이제 구조체, 클래스, 열거형과 같은 타입들과 프로토콜의 연관 값을 함께 사용하는 방법을 살펴봅시다.

이미지를 다루는 ImageProcesser 클래스를 만들며 다양한 타입들과 프로토콜 연관 값이 함께 사용되는 상황을 살펴 봅시다.
아래의 예시에서는 Worker 프로토콜을 따르는 ImageCropper 클래스를 프로퍼티로 가진 ImageProcessor 클래스를 만들고 있습니다.
ImageProcessor 클래스가 하는 일은 Worker 프로토콜에 의해 결정됩니다.

아래 코드로 확인해 봅시다.

```swift
protocol Worker {
  associatedtype Input  // just like generics
  associatedtype Output  // just like generics
  
  @discardableResult
  func start(input: Input) -> Output
}

final class ImageCropper: Worker {
  let size: CGSize

  init(size: CGSize) {
    self.size = size
  }

  func start(input: UIImage) -> Bool {
    // 이미지 조작 이후 성공 시 true 리턴
    return true
  }
}

final class ImageProcessor<W: Worker>
  where W.Input == UIImage, W.Output == Bool {
    // where 절을 사용해 프로토콜의 연관 값(Input, Output)에 제약을 걸어 ImageProcessor에 필요한 타입을 확정했습니다. 
    let worker: W

    init(worker: W) {
      self.worker = worker
    }

    private func process() {
      // start batches
      var results = [Bool]()

      let amount = 50
      var offset = 0
      var images = fetchImages(amount: amount, offset: offset)
      var failedCount = 0
      while !images.isEmpty {
        for image in images {
          if !worker.start(input: image) {
            failedCount += 1
          }
        }
        offset += amount
        images = fetchImages(amount: amount, offset: offset)
      }
      print("\(failedCount) images failed.")
    }

    private func fetchImages(amount: Int, offset: Int) -> [UIImages] {
      // return images from database
      return [UIImage(), UIImage()]
    }
}

let cropper = ImageCropper(size: CGSize(width: 200, height: 200))
let imageProcessor = ImageProcessor(worker: cropper)
```

ImageProcessor 클래스에 제네릭을 Worker 프로토콜로 제약하여 Worker 프로토콜을 따르는 image croppers, resizers 등에도 대응하는 클래스를 만들었습니다.
물론 W.Input과 W.Output의 타입은 연관 값 제약에 맞춰 UIImage 타입과 Bool 타입으로 들어와야 합니다.

만약 이미지를 다루는 '일'이 아니라 URL을 다루는 '일'이라면 Worker 프로토콜의 연관 값을 수정해 UrlCropper 클래스를 만들고,
UrlProcessor 클래스의 연관 값 타입 제약을 URL에 어울리도록 구현해주면 됩니다.

이미지와 관련된 작업일 경우 Worker 프로토콜의 연관 값인 Input, Output의 타입은 UIImage와 Bool이어야 합니다.
따라서 이미지와 관련된 작업을 하는 클래스마다 where 절로 연관 값 제약 구문을 반복해서 작성해야 합니다.
다시 말해 프로토콜의 연관 값을 제약하는 코드가 각 클래스마다 중복됩니다.

우리는 프로토콜을 채택한 구현부가 아니라 프로토콜 수준에서 연관 값 제약을 걸어 해당 프로토콜을 따르는 타입의 연관 값에 제약을 줄 수 있습니다.
이 방법은 프로토콜 연관 값 중복을 줄여줍니다.
아래 코드로 확인해 봅시다.

```swift
protocol Worker {
  associatedtype Input  // just like generics
  associatedtype Output  // just like generics
  
  @discardableResult
  func start(input: Input) -> Output
}

protocol ImageWorker: Worker where Input == UIImage, Output == Bool {
  // extra methods can go here if you want
}
```

위와 같이 Worker 프로토콜을 따르는 타입 중 Worker 프로토콜의 연관 값을 UIImage, Bool로 제약해야 하는, 
즉 이미지를 다루는 객체가 여러 개라면 Worker 프로토콜을 채택한 ImageWorker 프로토콜을 만들고 연관 값 제약 구문을 추가해 중복되는 코드를 줄일 수 있습니다.

ImageWorker 프로토콜에 연관 값 제약 코드를 추가한 덕분에 아래와 같이 깨끗한 코드가 나옵니다.
이미지를 다루는 클래스는 ImageWorker 프로토콜을 채택할 수 있습니다.

```swift
// Before :
final class ImageProcessor<W: Worker>
where W.Input == UIImage, W.Output == Bool { ... }

// After
final class ImageProcessor<W: ImageWorker> { ... }
```

연관 값을 가진 프로토콜과 제네릭은 추상적인 코드를 만들어 줍니다.
하지만 연관 값을 가진 프로토콜은 훌륭한 방법이지만 코드를 다소 어렵게 만듭니다. 

You don't always have to make things difficult, however. Sometimes a single generic or concrete code is enough to give you what you want.

## Summary
- You can use protocols as generic constraints. But protocols can also be used as a type at runtime (dynamic dispatch) when you step away from generics.
- Using protocols as a generic constraint is usually the way to go, until you need dynamic dispatch.
- Associated types are generics that are tied to a protocol.
- Protocols with associated types allow a concrete type to define the associated type. Each concrete type can specialize an associated type to a different type.
- Protocols with Self requirements are a unique flavor of associated types referencing the current type.
- Protocols with associate types or Self requirements force you to reason about types at compile time.
- You can make a protocol inherit another protocol to further constrain its associated types.
