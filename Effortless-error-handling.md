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

첫번째 방법은 함수가 외부의 상태를 조작하지 않도록 만드는 것입니다.

"Make func immutable"

함수의 인자로 들어온 값만 함수가 조작해 리턴한다면 외부의 상태를 조작하지 않는 함수입니다.
외부의 값을 변경하지 않으면 에러를 던지더라도 애플리케이션 상태을 유지할 수 있습니다.

두번째 방법은 작업이 에러 없이 끝났다면 작업의 결과(새로운 상태)를 저장하는 것입니다.

"use temporary value"

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

위 코드는 에러가 for 루프 중에 발생하지 않는다면 문제가 없지만, for 루프 중 에러가 발생해 에러를 던질 경우 애플리케이션 상태는 에러 발생 이전의 상태와 달라집니다.
만약 세 번째 for 루프에서 에러가 발생하면 첫 번째와 두 번째의 trimmedString이 TodoList의 values에 추가되며 애플리케이션 상태를 유지하지 못합니다.

아래 코드처럼 임시 변수를 만들어 애플리케이션 상태를 유지합시다.

```swift
enum ListError: Error {
  case invalidValue
}

struct TodoList {
  private var values = [String]()

  mutating func append(strings: [String]) throws {
    var tempValues = [String]()
    for string in strings {
      let trimmedString = string.trimmingCharacters(in: .whitespacesAndNewlines)
    
      if trimmedString.isEmpty {
        throw ListError.invalidValue
      } else {
        tempValues.append(trimmedString)
      }
    }
    values.append(tempValues)
  }
}
```

임시 변수 tempValues를 선언해 모든 for 루프에서 에러를 던지지 않을 때 작업의 결과를 리턴하며 에러 발생 상태에서 애플리케이션 상태를 유지할 수 있게 됩니다.
for 루프 중간에 에러가 발생하면 임시 변수는 사라지고 임시 변수를 실제 값에 대입하지 않습니다.
이로써 todoList를 에러가 발생하기 전 상태로 유지할 수 있습니다.

마지막 방법은 defer 클로저를 사용하는 방법입니다.

"Recovery code with defer"

에러가 발생했을 때 에러 발생 이전의 변경사항을 되돌리는 방식입니다. defer 클로저는 함수가 끝나면 실행됩니다. 함수에서 에러가 발생되었는지 유무와 관계없이 defer 클로저는 함수 끝에 실행됩니다.
defer 클로저에서 에러 발생 여부를 판단하고 에러가 발생했다면 에러 발생 이전의 상태로 되돌려 애플리케이션 상태를 유지합니다.

아래 코드에서는 파일의 저장 개수를 함수 입력으로 들어온 data 개수와 비교하여 에러 발생 여부를 판단하고 있습니다.

```swift
import Foundation

func writeToFiles(data: [URL: String]) throws {
  var storedUrls = [URL]()
  defer {
    if storedUrls.count != data.count {
      for url in storedUrls {
        try! FileManager.default.removeItem(at: url)
      }
    }
  }

  for (url, contents) in data {
    try contents.write(to: url, atomically: true, encoding: String.Encoding.utf8)
    storedUrls.append(url)
  }
}
```

defer 클로저는 writeToFiles 함수의 종료가 정상적인 종료인지 비정상적인 종료(에러 던짐)인지 구분해야 합니다.
writeToFiles 함수는 데이터 개수를 비교해 구분했습니다.

defer 클로저를 사용하면 에러가 발생하기 이전의 상태를 정확하게 유지하기 유리합니다. 하지만 여러 상황이 섞여있다면 defer 클로저가 대응해야 할 상황이 많아져 오히려 복잡성을 
높일 수 있습니다.

## Error propagation and catching

"My favorite way of dealing with problems is to give them to somebody else."

에러는 전달됩니다. 보통은 하위 수준의 함수에서 위로 에러를 전달합니다.
함수 호출은 상위 함수에서 하위 함수로 내려가고 하위 함수에서 발생한 에러는 함수 호출을 거슬러 상위 함수로 전달됩니다.
하위 수준 함수들은 에러 핸들링 방법을 모르고 발생하는 에러를 상위 함수로 던질 뿐입니다. 상위 함수에서 에러를 핸들링합니다.

아래 코드로 확인해봅시다.

```swift
struct Recipe {
  let ingredients: [String]
  let steps: [String]
}

enum ParseRecipeError: Error {
  case parseError
  case noRecipeDetected
  case noIngredientsDetected
}

struct RecipeExtractor {
  let html: String

  func extractRecipe() -> Recipe? {
    do {
      return try parseWebpage(html)
    } catch {
      print("Could not parse recipe")
      return nil
    }
  }

  private func parseWebpage(_ html: String) throws -> Recipe {
    let ingredients = try parseIngredients(html)
    let steps = try parseSteps(html)
    return Recipe(ingredients: ingredients, steps: steps)
  }

  private func parseIngrediants(_ html: String) throws -> [String] {
    // ... Parsing happens here

    // .. Unless an error is thrown
    throw ParseRecipeError.noIngredientsDetected
  }

  prviate func parseSteps(_ html: String) throws -> [String] {
    // ... Parsing happens here

    // .. Unless an error is thrown
    throw ParseRecipeError.noRecipeDetected
  }
}
```

위 코드로 함수 호출의 흐름과 에러 전달 흐름을 확인할 수 있습니다.

하지만 위와 같이 에러를 상위 계층으로 전달하면 발생하는 단점이있습니다.
에러에 대한 정보는 에러가 발생한 하위 계층에서 더욱 자세히 알 수 있습니다. 예를 들어 어떤 동작의 실패로 발생한 에러이고 각 변수의 상태가 어떤지 에러가 발생한 하위 계층에서는 명확이 알 수 있습니다.
하지만 에러를 상위 계층으로 전달하면 에러에 대한 자세한 정보 없이 어떤 에러가 발생했다는 사실만 전달됩니다.
유용한 정보를 잃는것 입니다.

따라서 에러를 상위 계층으로 전달할 때 에러와 함께 유용한 정보를 함께 전달해야 합니다.
에러에 대한 유용한 정보는 에러를 핸들링하는 상위 계층에 유용합니다.

아래 ParseRecipeError의 parseError 케이스처럼 enum의 케이스에 튜플을 추가하는 방식으로 에로와 같게 에러에 대한 정보를 전달할 수 있습니다.

```swift
enum ParseRecipeError: Error {
  case parseError(line: Int, symbol: String)
  case noRecipeDetected
  case noIngredientsDetected
}

struct RecipeExtractor {
  let html: String

  func extractRecipe() -> Recipe? {
    do {
      return try parseWebpage(html)
    } catch let ParseRecipeError.parseError(line, symbol) {
      print("Parsing failed at line: \(line) and symbol: \(symbol)")
      return nil
    } catch {
      print("Could not parse recipe")
      return nil
    }
  }
    
  // ...snip
}
```

위와 같이 에러에 정보를 추가할 때 enum에 튜플로 구현할 수 있지만, LocalizedError 프로토콜을 사용할 수도 있습니다.
LocalizedError 프로토콜은 에러의 정보를 추가하는 역할을 합니다. 

LocalizedError 프로토콜은 네 가지 프로퍼티를 지원하며 에러에 대한 정보를 추가합니다.
네 가지 프로퍼티는 아래와 같습니다. 네 가지 프로퍼티는 필수로 구현할 필요는 없습니다.

- var errorDescription: String?   에러 정보를 추가합니다.
- var failureReason: String?      에러 발생 이유를 설명합니다.
- var helpAnchor: String?         apple's help viewer 링크로 연결합니다.
- var recoverySuggestion: String? 에러에 대처하는 방법을 설명합니다.

보통은 LocalizedError 프로토콜의 errorDescription, recoverySuggestion 프로퍼티 정도로 충분합니다.
아래 코드는 LocalizedError 프로토콜을 채택하여 에러에 정보를 추가한 코드입니다.

```swift
extension ParseRecipeError: LocalizedError {
  var errorDescription: String? {
    switch self {
    case .parseError:
      return NSLocalizedString("The HTML file had unexpected symbols.", comment: "Parsing error reason unexpected symbols")
    case .noIngredientsDetected:
      return NSLocalizedString("No ingredients were detected.", comment: "Parsing error no ingredients")
    case .noRecipeDetected:
      return NSLocalizedString("No recipe was detected.", comment: "Parsing error no recipe")
    }
  }

  var failureReason: String? {
    switch self {
    case let .parseError(line: line, symbol: symbol):
      return String(format: NSLocalizedString("Parsing data failed at line: %i and symbol: %@, comment: "Parsing error line symbol"), line, symbol)
    case .noIngredientsDetected:
      // ...snip
    case .noRecipeDetected:
      // ...snip
    }
  }

  var recoverySuggestion: String? {
    return "Please try a different type"
  }
}
```

에러에 human-readable(by LocalizedError)을 추가하여 안정적으로 에러를 전달할 수 있습니다.





































