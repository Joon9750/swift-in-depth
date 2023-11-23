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
















