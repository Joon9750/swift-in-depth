# Generics

## This chapter covers
- How and when to write generic code
- Understanding how to reason about generics
- Constraining generics with one or more protocols
- Making use of the Equatable, Comparable and Hashable protocols
- Creating highly reusable types
- Understanding how subclasses work with generics

## The benefits of generics

제네릭은 스위프트에 필수적이고 자주 등장합니다.
다형성을 위해서는 제네릭과 프로토콜은 필수적입니다.

제네릭 없이 여러 타입을 하나의 함수가 대응하기에는 어렵습니다.
물론 Any 타입을 사용해 여러 타입에 대응할 수 있지만, Any를 사용하면 런타임에 Any를 특정 타입(String, Int 등)으로 다운 캐스팅해야 합니다.
이런 보일러플레이트 코드를 피하기 위해 제네릭이 사용됩니다.

제네릭으로 컴파일 타입에 다형성을 이룰 수 있습니다.

아래 코드는 제네릭 없이 사용하던 함수를 제네릭 함수로 고친 코드입니다.






















