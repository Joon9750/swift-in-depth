# Demystifying initializers

## This chpater overs
- Demystifying Swift's initializer rules
- Understanding quirks of struct initializers
- Understanding complex initializer rules when subclassing
- How to keep the number of initializers low when subclassing
- When and how to work with required initializers

## Struct initializer rules
구조체는 subclassing을 지원하지 않기 때문에 생성자가 상대적으로 정직합니다.
그럼에도 특별한 규칙이 몇 가지 있습니다.
기본적으로 스위프트에서는 구조체와 클래스의 모든 프로퍼티가 초기화 시키는 것을 원칙으로 여깁니다.
따라서 구조체와 클래스의 프로퍼티가 초기화되지 않았다면 컴파일되지 않습니다.

구조체는 아래 코드와 같이 생성자를 따로 구현하지 않았다면 모든 프로퍼티를 초기화하는 default init을 제공합니다.


























