# Effortless error handling

## This chapter covers
- Error-handling best practices (and downsides)
- Keeping your application in a proper state when throwing
- How errors are propagated
- Adding information for customer-facing applications (and for troubleshooting)
- Bridging to NSError
- Making APIs easier to use without harming the integrity of an application

## Errors in Swift

에러는 여러 종류로 나뉩니다.
크게 세 종류로 나눠보면 Programming errors, User errors, Errors revealed at runtime으로 나눌 수 있습니다.

Programming errors는 배열의 잘못된 인덱스 접근, 오버플로, 0으로 나누었을 때 발생하는 에러로 코드 레벨에서 충분히 고칠 수 있는 에러입니다.
User errors는 쉽게 말해 사용자가 서비스를 사용할 때 발생되는 에러입니다.
Errors revealed at runtime은 네트워크 상태가 불안정하거나 만료된 인증서를 사용하는 등 런타임에서 발생되는 에러입니다.

에러를 던지고 다루는 부분도 중요하지만 에러를 던질 때 애플리케이션을 예측 가능한 상태로 유지하는 것도 굉장히 중요합니다.

스위프트는 에러를 처리할 때 Error 프로토콜을 제안합니다. Error 프로토콜은 필수로 구현해야 할 요구사항이 없습니다.

enum은 각 case 별로 독립적이기 때문에 에러를 enum으로 만들기 적합합니다.
하지만 모든 에러를 enum으로 만들 필요는 없습니다. 
주로 만들진 않지만, 구조체로도 충분히 에러를 만들 수 있습니다. 구조체는 error에 더 많은 정보를 추가할 때 어울립니다.  

아래 코드는 enum으로 에러를 만든 예입니다.

```swift
enum ParesLocationError: Error {
  case invalidData
  case locationDoesNotExist
  case middleOfTheOcean
}
```

에러에 더 많은 정보를 추가해야 할 때 구조체를 사용합니다.

```swift
struct MultipleParseLocationErrors: Error {
  let parsingErrors: [ParseLocationError]
  let isShownToUser: Bool
}
```

에러는 던져지고 처리되기 위해 존재합니다.
throws 키워드를 함수에 붙여 해당 함수가 에러를 던질 수 있다는 사실을 표현합니다.

아래 코드는 함수에 throws 키워드를 붙인 예입니다.

```swift
struct Location {
  let latitude: Double
  let longitue: Double
}

func parseLocation(_ latitude: String, _ longitude: String) throws -> Location {
  guard let latitude = Double(latitude), let longitude = Double(longitude) else {
    throw ParseLocationError.invalidData  // 에러를 던지고
  }
  return Location(latitude: latitude, longitude: longitude)
}

do {
  try parseLocation("I am not a double", "4.899431")
} catch {  // 에러를 받아 처리합니다.
  print(error)  // invalidData
}
```

하지만 스위프트에서는 함수가 던질 에러의 정보를 드러내지 않습니다.
위 코드에서 parseLocation의 함수 정의문을 봤을 때 throws 키워드로 함수가 에러를 던질 수 있다는 사실은 알지만 어떤 종류의 에러를 던질 수 있을지 알 수 없습니다.
함수 내부를 봐야지만 ParseLocationError.invalidData 에러를 던진다는걸 알 수 있습니다.

따라서 가능하다면 어떤 에러를 던질지 몇가지 정보를 제공하는걸 추천합니다.

"Quick Help"를 사용해 함수가 어떤 에러를 던질지 정보를 제공할 수 있습니다.
에러를 던지는 함수에 커서를 올리고 cmd-Alt-/를 누르면 Quick Help templete을 만들 수 있습니다.

아래 코드처럼 Quick Help로 함수가 던지는 에러의 정보를 함수 구현부를 보지 않고도 알 수 있습니다.

```swift
/// Turns two strings with a latitude and longitude value into a Location type
///
/// - Parameters:
///   - latitude: A string containing a latitude value
///   - longitude: A string containing a longitude value
/// - Returns: A Location struct
/// - Throws: Will throw a ParseLocationError.invalidData if lat and long can't be converted to Double
func parseLocation(_ latitude: String, _ longitude: String) throws -> Location {
  guard let latitude = Double(latitude), let longitude = Double(longitude) else {
    throw ParseLocationError.invalidData  // 에러를 던지고
  }
  return Location(latitude: latitude, longitude: longitude)
}
```

위에서 이야기 했듯이 에러를 처리하는것 만큼 에러 상황에서 애플리케이션 상태를 예측 가능한 상태로 유지하는것도 중요합니다.

에러가 발생해도 애플리케이션 상태는 기존과 동일하게 유지되어야 합니다. 변경되어서는 안됩니다.
함수가 에러를 던진다면 애플리케이션의 상태(환경 & 인스턴스 상태)는 유지되어야 합니다.

앞으로 에러를 던진 이후 애플리케이션 상태를 유지하기 위한 세 가지 방법을 살펴보겠습니다.

Make func immutable

첫번째 방법은 함수가 외부의 상태를 조작하지 않도록 만드는 것입니다.
함수의 인자로 들어온 값만 함수가 조작해 리턴한다면 외부의 상태를 조작하지 않는 함수입니다.
외부의 값을 변경하지 않으면 에러를 던지더라도 애플리케이션 상태을 유지할 수 있습니다.

use temporary value

두번째 방법은 작업이 에러 없이 끝났다면 작업의 결과(새로운 상태)를 저장하는 것입니다.
작업이 에러 없이 끝나기 전까지 작업의 결과는 temporary value(임시 변수)에 저장하는 방법입니다.

아래 코드는 temporary value를 사용하지 않은 코드와 사용한 코드입니다.

```swift
enum ListError: Error {
  case invalidValue
}

struct TodoList {
  private var values = [String]()

  mutating func append(strings: [String]) throws {
    for string in strings {
      let trimmedString = string.trimmingCharacters(in: .whitespacesAndNewlines)

      if trimmedString.isEmpty {
        throw ListError.invalidValue
      } else {
        values.append(trimmedString)
      }
    }
  }
}
```

위 코드는 에러가 for 루프 중에 발생하지 않는다면 문제가 없지만, for 루프 중 에러가 발생해 에러를 던질 경우 애플리케이션 상태를 

























