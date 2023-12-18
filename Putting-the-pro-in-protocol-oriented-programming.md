# Putting the pro in protocol-oriented programming

## This chapter covers
- The relationship and trade-offs between generics and using protocols as types
- Understanding associated types
- Passing around protocols with associated types
- Storing and constraining protocols with associated types
- Simplifying your API with protocol inheritance

## Runtime versus compile time
프로토콜은 크게 두 분류로 나뉩니다.
제네릭을 제약할 때의 프로토콜과 단일 타입으로 쓰이는 프로토콜로 나뉩니다.
제네릭을 제약할 때의 프로토콜은 컴파일 타임에 타입을 결정하고 단일 타입으로 쓰이는 프로토콜의 경우 런타임에 타입을 결정합니다.

지금부터 코인 포토폴리오를 구현하며 살펴봅시다.
요구 사항으로 다양한 코인에 대응하는 포토폴리오를 구현해야 합니다.

코인 포토폴리오를 구현하기 위해 열거형을 떠올릴 수 있습니다.
하지만 모든 코인에 대응해야하기 때문에 너무 많은 case들이 나오기 때문에 좋은 선택지는 아닙니다.

이때 우리는 프로토콜을 통해 구현하여 코인 포토폴리오를 추상화 할 수 있습니다.
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

CryptoCurrency 프로토콜을 통해 코인이라면 공통적으로 가지는 프로퍼티를 선언해 중복을 줄입니다.
또한 프로토콜의 프로퍼티는 항상 var로 선언되어야 합니다.
프로토콜을 따르는 구현부에서 프로퍼티를 var 또는 let으로 설정 가능합니다.
만야 프로토콜의 프로퍼티가 { get set }이라면 구현부에서는 해당 프로퍼티를 var로 구현해야 합니다.

그렇다면 코인 프로퍼티를 가지는 portfolio 클래스를 만들어 봅시다.
코인 프로퍼티는 CrypoCurrency를 채택한 상태여야 하고 여러 코인 타입을 함께 담고 싶습니다.

이때 만약 제네릭을 제약하도록 프로토콜을 사용하면 어떨까요?
아래 코드로 살펴 봅시다.

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

프로토콜을 통한 제네릭 제약과 함께 서브 클래싱이 발생하는 상황입니다.
제네릭으로 사용되는 타입은 커스텀 타입일 경우 상속 관계를 잃습니다.

따라서 Portfolio 객체에 CrytoCurrency 프로토콜을 따르는 타입 중 하나를 넣게 되면, 처음 넣은 타입으로 제네릭의 데이터 타입이 컴파일 타임에 고정됩니다.
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
print(type(of: portfolio))  // Portfolio<Ethereum>, 프로토콜을 제네릭 제약에 사용했을 때 컴파일 타임에 제네릭의 타입이 하나로 고정됩니다. 
print(type(of: portfolio.coins))  // Array<Ethereum>, 프로토콜을 제네릭 제약에 사용했을 때 컴파일 타임에 제네릭의 타입이 하나로 고정됩니다. 
```

제네릭과 프로토콜을 함께 사용했을 때 제네릭이 어떤 타입을 다루는지 컴파일 타임에 알 수 있다는 장점이 있습니다.
컴파일러의 성능도 최적화할 수 있습니다.

하지만 우린 여러 코인 타입을 함께 담고 싶습니다. 제네릭과 프로토콜을 함께 사용하면 요구사항을 충족할 수 없습니다.
그렇다면 어떻게 프로토콜을 통해 여러 코인 타입을 함께 담을 수 있을까요?

제네릭 없이 프로토콜을 하나의 타입으로 사용하면 여러 코인 타입에 대응하도록 Portfolio 클래스를 만들 수 있습니다.
제네릭 없이 쓰이는 프로토콜은 런타임에 타입이 결정됩니다.

아래 코드는 제네릭의 제약으로 쓰이던 프로토콜을 하나의 타입으로 쓰이는 프로토콜로 변경한 코드입니다.

```swift
// Before
final class Portfolio<Coin: CryptoCurrency> {
  var coins: [Coin]
  // ...생략
}

// After
final class Portfolio {
  var coins: [CryptoCurrency]
  // ...생략
}
```

제네릭 없이 타입으로 쓰이는 프로토콜은 런타임에 동작하며 서브 타입까지 모두 대응할 수 있습니다.
보다 유연한 코드로 생각됩니다.

아래 코드는 서브 타입까지 대응 가능한 '타입으로 쓰이는 프로토콜'의 코드입니다.

```swift
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

앞서 '제네릭을 제약하는 프로토콜'은 coins의 타입이 고정되었지만 '타입으로 쓰이는 프로토콜'의 경우 Array<CryptoCurrency>로 추상화 되어있습니다. 
만약 런타임에 동작하는 프로토콜을 원하다면 프로토콜을 타입이나 인터페이스로 사용하면 됩니다.

하지만 '타입으로 쓰이는 프로토콜'로 추상화 할 경우에도 단점이 있습니다.

만약 portfolio.coin에 추가 되는 Bitcoin 타입 객체가 CryptoCurrency protocol에서 제공하는 함수 외에 추가적인 함수가 있을 때, 
Array<CryptoCurrency>로 추상화 된 상태에서는 Bitcoin 타입의 추가적인 함수에 접근할 수 없습니다.
해당 함수에 접근하기 위해서는 CryptoCurrency 타입을 Bitcoin 타입으로 형 변환해야 합니다.

하지만 이는 안티패턴으로 여겨질 수 있고 CryptoCurrency 타입을 따르는 코인의 종류가 많아지면 대응하기 어려워집니다.
































