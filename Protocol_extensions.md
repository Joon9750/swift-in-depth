# Protocol extensions

## This chapter covers
- Flexibly modeling data with protocols instead of subclasses
- Adding default behavior with protocol extensions
- Extending regular types with protocols
- Working with protocol inheritance and default implementations
- Applying protocol composition for highly flexible code
- Showing how Swift prioritizes method calls
- Extending types containing associated types
- Extending vital protocols, such as Sequence and Collection

## Class inheritance vs. protocol inheritance

객체지향 프로그래밍은 서브 클래싱을 사용해 전형적으로 데이터를 모델링하는 방법입니다.
하지만 서브 클래싱의 수직적 데이터 모델링은 다소 엄격합니다.
클래스 기반의 상속을 대신해서 스위프트는 프로토콜 기반의 상속을 제공합니다.

클래스 기반의 상속은 수직적 데이터 구조를 만들고 프로토콜 기반의 상속은 수평적 데이터 구조를 이룹니다.

클래스 기반의 상속에서 부모 클래스를 상속한 자식 클래스에 새로운 함수를 추가하거나 부모 클래스의 함수를 오버라이드 할 수 있습니다.

만약, 네트워크 호출을 위한 URLRequest 타입을 생성하는 RequestBuilder 타입을 서브 클래싱 방식으로 만들어 봅시다.

RequestBuilder 클래스를 최상위 부모 클래스로 두고, RequestBuiler 클래스를 상속하는 RequestHeaderBuilder 자식 클래스를 만들어 RequestHeaderBuilder 클래스에서 header를 붙이는 작업을 추가합니다. 
또한 RequestHeaderBuilder 클래스를 상속한 EncryptedRequestHeaderBuilder 자식 클래스를 만들고 네트워크 요청의 데이터를 암호화하는 기능까지 추가합니다.

이처럼 클래스 기반 상속은 수직적(vertical) 데이터 구조를 형성합니다.

그렇다면 프로토콜 기반의 상속을 통해 RequestBuilder 타입을 만든다면 아래와 같습니다.
























