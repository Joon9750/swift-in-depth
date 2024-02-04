# Swift patterns

## This chapter covers
- Mocking types with protocols and associated types
- Understanding conditional conformance and its benefits
- Applying conditional conformance with custom protocols and types
- Discovering shortcomings that come with protocols
- Understanding type erasure
- Using alternatives to protocols

## Dependency injection

이번 챕터에서는 스위프트 개발에 유용한 패턴들을 살펴볼 예정입니다.
전통적인 객체지향 프로그래밍과 다른 스위프트에 집중한 패턴들을 알아보겠습니다.

먼저 살펴볼 것은 의존성 주입입니다.

의존성 주입에서 말하는 의존성이란 서로 다른 객체 사이에 의존 관계가 있다는 것입니다.
다시 말해, 의존하는 객체가 수정되면 다른 객체도 영향을 받게 됩니다.

또한 의존성 주입에서 주입이란 외부에서 생성자 등의 방법으로 미리 생성한 객체를 넣는것을 뜻합니다.

의존성을 가진 코드가 많으면 코드의 재사용성이 떨어집니다.
재사용을 위해 코드를 수정하면 매번 의존성을 가진 객체들을 함께 수정해야하기 때문에 재사용성이 떨어집니다.

의존성을 강하게 가지는 코드를 해결하기 위해 의존성 주입으로 해결할 수 있습니다.

또한 의존성 주입을 통해 아래와 같은 이점을 얻을 수 있습니다.

1. Unit Test가 용이해진다.
2. 코드의 재활용성을 높여준다.
3. 객체 간의 의존성(종속성)을 줄이거나 없엘 수 있다.
4. 객체 간의 결합도이 낮추면서 유연한 코드를 작성할 수 있다.

의존성 주입은 DIP(의존 관계 역전 법칙)와도 관련이 깊습니다.

의존 관계 역전 법칙은 객체 지향 프로그래밍 설계의 다섯가지 기본 원칙(SOLID) 중 하나입니다. 

추상화 된 것은 구체적인 것에 의존하면 안되고 구체적인 것이 추상화된 것에 의존 해야합니다.
즉, 구체적인 객체는 추상화된 객체에 의존 해야 한다는 것이 핵심입니다.

스위프트에서 말하는 추상화된 객체는 프로토콜로 생각할 수 있습니다.

**Swapping an implementation**

지금부터 의존성 주입을 활용하여 구현부를 교환할 수 있는 WeatherAPI를 구현할 것입니다.
여기서 구현부를 교환하는 것은 의존성 주입에 의해 실현됩니다.

WeatherAPI는 실제 네트워크 세션, 오프라인 네트워크 세션 그리고 테스트 세션을 모두 대응하도록 구현합니다.
이때 세 가지 세션은 주입 받는 객체가 됩니다.

WeatherAPI에 Session 프로토콜을 따르는 타입을 주입 받습니다.

주입 받는 세션의 정확한 타입과 상관없이 Session 프로토콜의 함수를 사용할 수 있습니다. 
의존성 주입을 통해 WeatherAPI는 Session 타입과 의존 관계를 만들지 않게 됩니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/c0e48dae-9db6-48a0-a787-0575b6ed8b96)

WeatherAPI가 가진 Session 프로토콜에 의해 Session 프로토콜을 따르는 URLSession, OfflineSession, MockSession 타입을 주입 받을 수 있습니다.
이를 구현부를 교환한다고 볼 수 있습니다.

Session 프로토콜을 코드로 구현해봅시다.






































