# Writing cleaner properties

## This chapter cover
- How to create getter and setter computed properties
- When(not) to use computed properties
- Improving performance with lazy properties
- How lazy properties behave with structs and mutability
- Handling stored properties with behavior

## Computed properties
연산 프로퍼티는 프로퍼티로 가장한 함수로 여겨집니다. 어떤 값을 저장하지 않지만 저장프로퍼티와 형태는 유사합니다.
연산 프로퍼티에 접근할 때마다 동작합니다. 따라서 "always up to date" 성질을 가집니다.

함수보다 연산프로퍼티가 가독성이 높습니다. 
함수중에서도 인자 없이 return 값은 있는 함수가 연산 프로퍼티로 바꿀만한 함수입니다.

