# Delivering quality Swift code

## This chapter covers
- Documenting code via Quick Help
- Writing good comments that don't distract
- How style isn't too important
- Getting consistency and fewer bugs with SwiftLint
- Splitting up large classes in a Swifty way
- Reasoning about making types generic

## API documentation
드디어 마지막 챕터입니다. ^~^

마지막 챕터(Delivering quality Swift code)에서는 코드적인 부분보다 프로젝트에 유용하게 쓸 수 있는 도구들을 살펴볼 예정입니다.

프로젝트는 주로 여러 동료들과 함께합니다. 따라서 자연스럽게 동료들이 만든 코드를 해석해야 합니다.
이때 API documentation을 만들어 프로젝트의 코드를 문서화 할 수 있습니다.

프로젝트의 코드를 문서화하는 방법은 Quick Help를 사용하거나 Jazzy 패키지를 사용해 문서화된 웹 페이지를 만드는 방법이 있습니다.

먼저 Quick Help를 사용하는 방법을 살펴봅시다.

Quick Help는 짧은 마크다운으로 코드상에서 아래와 같이 표시됩니다.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/0004055c-4ed8-467d-b25d-48d0a9d9fffa)

위와 같이 코드상에서 마우스를 통해 표시 되며 아래와 같이 sidebar에서도 표시됩니다.

![image](https://github.com/hongjunehuke/Swift-in-depth/assets/83629193/92721cc4-2f3e-48b8-b493-5f6855be4e81)


위와 같은 Quick Help는 '///'를 통해 작성할 수 있습니다.
아래 코드로 살펴봅시다.



























