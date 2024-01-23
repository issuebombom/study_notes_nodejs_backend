# 테스트 코드

## 테스트 코드의 개념

테스트 코드를 작성하는 이유는 '내 코드에는 오류가 없다.' 라는 것을 증명하기 위함이 아니라 `내 코드가 멀쩡하다면 이렇게 결과가 나와야 한다.` 라는 관점에서 테스트 코드를 작성해야 함을 기억하자

## 테스트 코드의 종류

### 단위 테스트

Unit Test라고 부르며 가장 작은 규모의 기능을 테스트한다. 가령 findAll 메서드를 만들었을 때 원하는 형태의 배열을 잘 가지고 오는지 테스트한다.

### 통합 테스트

Integration Test라고 부르며 여러가지 기능을 합쳤을 때 생기는 문제를 방지하기 위한 테스트이다. 가령 스트리밍 서비스에서 유저 회원 가입 기능을 추가했을 때 회원 정보가 DB에 잘 저장되었는지, 가입과 동시에 생성되는 유저의 채널 정보도 함께 잘 생성되었는지 등 여러 기능이 잘 연계되었는지 확인하는 테스트를 말한다.

### E2E 테스트

End-to-end Test라고 부르며 백엔드에서 웹페이지까지 총 과정을 통틀어 테스트하는 것을 의미한다.

## Jest 사용하기

Node.js 환경에서 가장 많이 활용되는 테스터 모듈이다.

### 기본 설정

```zsh
# 설치
npm init -y
npm i jest -D
```

```json
// package.json 수정
{
...
  "script": {
    "test": "jest"
  },
...
}
```

### 단위 테스트 코드 작성

먼저 아주 간단한 예시를 통해 단위 테스트 코드 적용 방식을 살펴보자.

아래 예시에서는 이메일 주소 양식이 올바로 작성되었는지 validation하는 기능에 대한 테스트 코드를 먼저 작성하고, 이후 실제 이메일 검증 기능을 구현하여 해당 기능이 내가 작성한 테스트 코드를 통과하는지 확인한다.

> 정리하자면
>
> 1. 테스트하고 싶은 항목에 대해 테스트 코드로 작성
> 2. 실제 기능 구현을 통해 테스트 코드를 통과하는지 확인

테스트 코드는 아래와 같이 작성한다.

```javascript
// validation.spec.js

const { isEmail } = require('./validation');

test('"@"이 하나만 있어야 한다.', () => {
  expect(isEmail('test@gmail.com')).toEqual(true);
  expect(isEmail('test@@gmail.com')).toEqual(false);
  expect(isEmail('testgmail.com')).toEqual(false);
  expect(isEmail('testgmail.com@')).toEqual(true);
});

test('공백이 존재해서는 안된다.', () => {
  expect(isEmail('test@gmail.com')).toEqual(true);
  expect(isEmail('test @gmail.com')).toEqual(false);
  expect(isEmail('test @gma il.com')).toEqual(false);
});

test('이메일 주소 맨 앞에 하이푼이 있으면 안된다.', () => {
  expect(isEmail('test@gmail.com')).toEqual(true);
  expect(isEmail('-test@gmail.com')).toEqual(false);
  expect(isEmail('-----test@gmail.com')).toEqual(false);
  expect(isEmail('tes-t@gm-ail.c-om')).toEqual(true);
});
```

validation.isEmail 메서드에 대한 테스트 코드이다.  
`test` 메서드는 테스트 주제에 대한 스크립트, 테스트 코드 실행 함수가 입력되고,  
`expect` 메서드를 통해 isEmail함수의 인풋값에 대한 기대값으로 true, false값을 예상하는 테스트 코드가 작성되었다.  
쉽게말해 test는 테스트 주제, expect 코드는 `우리가 정한 규정을 따랐을 경우와 그렇지 않았을 경우에 대한 예상 답안을 적어두는 곳`으로 이해하면 쉽다.

이제 우리가 내세운 조건을 성립하는 코드를 구현한다.

```javascript
// validation.js

module.exports = {
  isEmail: (value) => {
    const email = value || '';

    // "@"이 하나만 있어야 한다.'
    if (email.split('@').length !== 2) {
      return false;
      // 공백이 존재해서는 안된다.
    } else if (email.includes(' ')) {
      return false;
      // 이메일 주소 맨 앞에 하이푼이 있으면 안된다.
    } else if (email[0] === '-') {
      return false;
    }

    return true;
  },
};
```

구현한 코드가 과연 주석으로 입력된 조건을 만족시킨다면 테스트 코드는 모두 통과가 되어야 할 것이다.

```zsh
npm run test

>>>
 PASS  ./validation.spec.js
  ✓ "@"이 하나만 있어야 한다. (3 ms)
  ✓ 공백이 존재해서는 안된다. (1 ms)
  ✓ 이메일 주소 맨 앞에 하이푼이 있으면 안된다. (1 ms)

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        0.625 s, estimated 1 s
Ran all test suites.
```

모두 성공한다면 위와 같은 결과가 발생한다. 하지만 테스트 결과로 failed가 뜬다면 어떤 조건의 어떤 테스트 코드가 예상 답안과 다른지, 즉 통과 되지 못했는지 확인할 수 있고, 그 뜻은 내가 구현한 코드가 놓친 검증 기능이 있음을 의미한다.

아래 결과와 같이 테스트 실패의 경우 어떤 주제의 테스트 코드가 통과하지 못했는지 확인할 수 있다.

```zsh
● "@"이 하나만 있어야 한다.

    expect(received).toEqual(expected) // deep equality

    Expected: true
    Received: undefined

      2 |
      3 | test('"@"이 하나만 있어야 한다.', () => {
    > 4 |   expect(isEmail('test@gmail.com')).toEqual(true);
        |                                     ^
      5 |   expect(isEmail('test@@gmail.com')).toEqual(false);
      6 |   expect(isEmail('testgmail.com')).toEqual(false);
      7 |   expect(isEmail('testgmail.com@')).toEqual(true);

      at Object.toEqual (validation.spec.js:4:37)

```
