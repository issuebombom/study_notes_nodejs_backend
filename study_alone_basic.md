# 자바스크립트 기본기

> 📌 너무 기초적인 부분은 생략하고, 개인적으로 리마인딩이 필요할 것 같은 부분 위주로 정리합니다. (es6 문법 포함)

## Case

선택지에 따른 결과 출력을 달리하기 위해 사용

```javascript
myNumber = 2;
switch (myNumber) {
  case 1:
    console.log(`${myNumber}번을 선택했음`);
    break;
  case 2:
    console.log(`${myNumber}번을 선택했음`);
    break;
  default:
    console.log("해당내역 없음");
}
```

## 연산자 (Math Object)

> 항목: abs, max, min, sqrt, pow, round, floor, ceil, random 등등

```javascript
// 예시
Math.abs(-10); // 절댓값 (10)
```

## 진법 표기법

다음은 10진법의 255에 해당하는 값들이다.

```javascript
console.log(0xff) // 16진법 표기
console.log(0o377) // 8진법 표기
console.log(0b(11111111)) // 2진법 표기
```

## 반올림

toFixed를 쓰면 문자열로 변한다...(왜 이렇게 만들었어??)

```javascript
let myNumber = 255.667;

// 2자리수까지 표기(반올림)
console.log(Number(myNumber.toFixed(2)));

// 위와 같은 결과
console.log(+myNumber.toFixed(2));
```

## 문자열 다루기

```javascript
// 문자열 인덱싱
console.log(myString.charAt(3)); // 해당 인덱스값 출력
console.log(myString[3]); // 위랑 같음..charAt 왜쓰는거야?

myString.toUpperCase(); // 대문자 변경
myString.toLowerCase(); // 소문자 변경
myString.trim(); // 양쪽공백제거
myString.slice(0, 2); // myString[0: 2]
```

## 배열(Array) 주요 메서드

파이썬 문법과 비교

```javascript
let array = [1, 2, 3, 4, 5];

array.push(1); // array.append(1)
array.pop(); // array.pop()
array.shift(); // deque.popleft()
array.unshift(1); // deque.appendleft(1)

// idx 1 포함 2개 지우고 그 자리에 99, 99를 넣는다.
array.splice(1, 2, 99, 99);
array.slice(1, 2); // array[1 : 2]

// 해당 요소의 idx값 출력 (못찾으면 -1 출력)
array.indexOf(3); // 앞에서부터 3 찾기
array.lastIndexOf(3); // 뒤에서부터 3 찾기

// 포함 여부
array.includes(3); // true

// 뒤집기
array.reverse();
```

## 정렬(Sorting)

자바스크립트에서의 sort 메서드는 실행 메서드이기도 하지만 리턴값이 있어 정렬 결과를 새로운 변수에 할당도 가능하다.

파라미터 a와 b를 받아서 a - b의 값을 반환한다.  
이 비교 함수는 두 값의 차이를 계산하고,

1\. 결과가 양수이면 b를 a보다 앞에 위치시키고,  
2\. 결과가 음수이면 a를 b보다 앞에 위치시키며,  
3\. 결과가 0이면 위치를 변경하지 않습니다.

```javascript
// 정렬(숫자)
array.sort((a, b) => a - b); // ascending
array.sort((a, b) => b - a); // descending

// 정렬(문자)
const myArray4 = ["a", "ab", "ca", "dab"];
myArray4.sort((a, b) => a.localeCompare(b));
```

## 객체(Object) 주요 메서드

객체에 프로퍼티로 함수를 담을 수 있다.  
key값을 문자열로 적지 않아도 된다.  
key값 접근은 `.key` 또는 `['key']` 둘 다 가능

```javascript
let obj = {
  name: "apple",
  age: 6,
  sum: (a, b) => a + b,
};

console.log(obj.name); // apple
console.log(obj.sum(1, 2)); // 3

console.log(Object.keys(obj)); // key값 배열
console.log(Object.values(obj)); // value값 배열
console.log(Object.entries(obj)); // [key, value] 배열

// shallow copy, 프로퍼티 추가 안해도 됨
let newObj = Object.assign({}, obj, { gender: "female" });
console.log(newObj); // copy + gender 프로퍼티 추가
```

> 🚨 `Caution`  
> 얉은 복사(swallow copy)는 객체 내 객체(2-depth)부터는 메모리를 공유한다.

## null 병합 연산자

`??` 연산은 왼쪽값이 `null` 또는 `undifined`라면 오른쪽값, 그 외 값이면 왼쪽값을 출력한다.

```javascript
let whoAmI = null;
console.log(whoAmI ?? "alternative"); // alternative 출력
```

```javascript
let myObj = {
  name: "apple",
  age: 6,
};

const propertyChecker = (obj, prop) => {
  console.log(obj[prop] ?? `There is no such property: ${prop}`);
};

// There is no such property: gender
propertyChecker(myObj, "gender");
```

병합 연산자를 활용하여 object에 특정 property 있는지 체크하는 함수를 만들었다.

## 조건부 연산자

`?` 연산을 통해 `true` or `false`에 따른 결과를 구분할 수 있다.

```javascript
const CUT_OFF = 80;
let score = 75;
let result = score > CUT_OFF ? "pass" : "fail";
console.log(result); // "fail"
```

```javascript
let myObj = new Object();
let strArray = ["a", "b", "a", "b", "c", "d", "a", "b"];

strArray.forEach((num) => {
  myObj[num] = myObj[num] ? ++myObj[num] : 1;
});

console.log(myObj); // { a: 3, b: 3, c: 1, d: 1 }
```

문자열 배열에 들어있는 알파벳 각각의 총 개수를 오브젝트로 나타낼 때 응용할 수 있다.

## 즉시 실행 함수

함수를 생성 즉시 실행하는 기능

```javascript
// 즉시 실행 함수
const testFunc = (() => {
  console.log("This is Function");
})();

testFunc; // 콘솔에 "This is Function" 출력
```

testFunc변수에 함수 실행 명령인 `()` 없이도 함수가 실행된다.

```javascript
const testFunc = (() => {
  return subFunc() {
    console.log("I'm sub function")
  }
})
```

```javascript
const testFunc = (() => {
  return function subFunc() {
    console.log("I'm sub function");
  };
})();

testFunc(); // 콘솔에 "I'm sub function" 출력
```

위 예시는 고차함수에 즉시 실행 함수를 적용한 예시이다.  
testFunc 함수의 리턴값이 subFunc함수 자체이므로 testFunc함수가 즉시 실행되면 결국 subFunc함수를 쥐고 있는 것과 동일하다.  
이 상태에서 `()`를 통해 함수를 실행하므로 subFunc함수가 실행된 결과를 얻게 된다.

## spread 구문

배열과 오브젝트를 spread했을 때 효과를 살펴본다.  
객체 변수 앞에 `...`을 붙여 사용한다.

```javascript
const numbers = [1, 2, 3];

// 오브젝트 전환
const newObject = { ...numbers };
console.log(newObject); // { '0': 1, '1': 2, '2': 3 }

// 오브젝트 concat
const newObject2 = {
  ...newObject,
  3: 4,
};
console.log(newObject2); // { '0': 1, '1': 2, '2': 3, '3': 4 }

// 배열 활용
const numbers2 = [...numbers]; // swallow copy
const numbers3 = [...numbers2, 4]; // [1, 2, 3, 4]
const numbers4 = [...numbers, ...numbers3]; // [1, 2, 3, 1, 2, 3, 4]
```

## 모던 프로퍼티 표기법

const 변수를 객체에 담으면 변수를 key, 할당값을 value로 인식하여 오브젝트의 프로퍼티로 추가할 수 있다.  
함수 프로퍼티를 아래와 같이 축약해서 입력도 가능하다.

```javascript
const title = "Apple";
const birth = 2023;

const userObject = {
  title,
  birth,
  getFullName() {
    return `${this.title} ${this.birth}`;
  },
};

console.log(userObject.getFullName());
```

> 💡 `Tip`  
> `this`는 this를 품은 객체 자체를 의미한다. 그래서 위 코드 `this.title`은 곧 `userObject.title`과 동일하다.

## 옵셔널 체이닝 연산자

기본적으로 없는 프로퍼티를 호출할 경우 에러가 발생한다.  
이로 인해 코드 작동이 중지되는 것을 방지하기 위해 에러 대신 undefined를 출력하게 하는 연산자이다.

> 💡 `Tip`  
> Node에서 실행했을 때는 `?` 연산자가 없이 존재하지 않는 프로퍼티에 접근해도 undefined를 잘 출력해준다.

```javascript
const catName = {
  cat: {
    name: "Tom",
  },
};

console.log(catName?.cat); // { name: 'Tom' }
console.log(catName.cat?.name); // 'Tom'
console.log(catName.cat?.age); // undefined
```

## 구조분해 (Destructuring)

객체의 프로퍼티를 꺼내서 여러 변수에 병렬로 간단하게 할당하는 방법이다.

```javascript
// a, b, c, d, e에 각 값이 할당된다.
const [a, b, c, d, e] = [1, 2, 3, 4, 5];
console.log(a, b, c, e, d); // 1 2 3 4 5

// d에는 배열 [4, 5]가 할당된다.
const [a, b, c, ...d] = [1, 2, 3, 4, 5];
console.log(d); // [4, 5]
```

```javascript
// key를 변수로 value가 할당된다. (이름 일치 필수)
let obj = {
  nickname: "jack",
  age: 6,
  gender: "female",
};

const { nickname, age, gender } = obj;
console.log(nickname, age, gender); // jack 6 female
```

## 에러 발생시키기

```javascript
// 인위적으로 애러를 발생시킬 때 throw를 쓸 수 있다.
const error = new TypeError("타입 에러가 발생했다.");
throw error;
```

## try, catch, finally

try에서 시도한 코드가 에러를 일으키면 catch에 입력된 코드를 실행한다.  
finally는 에러유무와 상관없이 무조건 실행된다.

> 💡 `Tip`  
> catch문을 쓰면 에러 발생 시 코드 실행 자체가 멈추는 것을 방지할 수 있다. 코드 실행이 멈추는 것은 곧 서버가 다운되는 것을 의미한다.

```javascript
// 실행 순서를 익혀보자

let result = null;

try {
  const apple = "apple";

  // const는 재할당 안됨 (에러 발생) -> catch
  apple = "another apple";

  // 정상 작동 시 해당 영역 실행
  result = "GOOD";
} catch (err) {
  // 에러 발생 시 에러 객체가 catch로 전달됨
  console.error(err); // 에러 객체 확인
  console.error(err.name); // 에러 타입 확인
  console.error(err.message); // 에러 메시지 확인
  result = err.name;
} finally {
  const finalResult = "최종 결과: ";
  console.log(finalResult + result); // 최종 결과: TypeError
}
```

## map 메소드

return을 받아 변수 할당이 가능하다는 점이 forEach와 차이점이다. 각 배열 요소를 꺼내 수정할 때 쓰인다.

```javascript
const myArray = ["apple", "ball", "cat", "doll"];

const fixedArray = myArray.map((name, i) => {
  return `${i}. ${name}`;
});

console.log(fixedArray); // ["1. apple", "2. ball"...]
```

## filter, find 메소드

filter: 배열 출력, find: 오브젝트 출력  
조건이 없을 경우:  
(filter: 빈 어레이 출력, find: undefined 출력)

```javascript
const devices = [
  { name: "ipad", brand: "APPLE" },
  { name: "galaxy", brand: "SAMSUNG" },
  { name: "gram", brand: "LG" },
];

let answer = devices.filter((device, i) => device.brand === "APPLE");
console.log(answer); // [ { name: 'ipad', brand: 'APPLE' } ]

answer = devices.find((device, i) => device.brand === "LG");
console.log(answer); // { name: 'gram', brand: 'LG' }
```

## some, eveery 메소드

some: 조건을 만족하는 요소가 하나라도 있으면 True  
every: 조건을 만족하지 않는 요소가 하나라도 있으면 False  
empty 어레이 적용 시

- some은 false
- every는 true를 출력한다.

```javascript
const myArray = [1, 3, 5, 7, 9];

// some: 조건을 만족하는 요소가 하나라도 있으면 True
const someAnswer = myArray.some((i) => i > 5);
console.log(someAnswer); // true

// every: 조건을 만족하지 않는 요소가 하나라도 있으면 False
const everyAnswer = myArray.every((i) => i > 5);
console.log(everyAnswer); // false
```

## reduce 메소드

accumulate 연산을 떠올리면 된다.  
acc는 myInit을 통해 초기화값을 맨 먼저 받고, 다음부터는 각iteration마다의 리턴값을 받는다.  
el(element)는 배열의 요소를 처음부터 순차적으로 받는다.

myInit 영역에 값을 입력하지 않을 경우 첫번째 idx값을 초기값으로 선정  
이에 따라 idx도 0이 아닌 1부터 시작한다는 점 주의

```javascript
const myArray = [1, 2, 3, 4, 5];
const myInit = 0;

// reduce는 함수와 초기값을 파라미터로 받는다
const myResult = myArray.reduce((acc, el, idx) => {
  console.log(idx); // 0, 1, 2...
  return acc + el;
}, myInit);

console.log(myResult); // 15가 출력
```

## Map, Set 함수

Map

```javascript
const myMap = new Map();

myMap.set("name", "apple");
myMap.set("age", "19");
myMap.set("gender", "male");
console.log(myMap);

console.log(myMap.get("name")); // apple
console.log(myMap.has("name")); // true
myMap.delete("name"); // 해당 key가 제거됨, 콘솔 출력 시 true 반영됨
console.log(myMap.size); // 사이즈 확인
myMap.clear(); // 모두 제거
console.log(myMap); // Map(0) {}
```

Set  
집합을 의미한다. Set도 Map과 거의 동일하게 동작한다.  
데이터 추가 시 .add를 쓴다는 점이 Map과의 차이

```javascript
const mySet = new Set([1, 2, 3]); // 필요 시 괄호 안에 배열 입력 가능
mySet.add("name");
console.log(mySet); // Set(4) { 1, 2, 3, 'name' }

// Set을 Array로 전환
console.log(new Array(...mySet)); // [ 1, 2, 3, 'name' ]
```

## 모듈 스코프

자바스크립트도 여러 파일을 만들어서 모듈화 할 수 있다.  
html에서 `<script src='index.js'></srcipt>`와 같은 형식으로 불러올 텐데 `<script type="module" scr='index.js'></srcipt>` 와 같이 type을 module로 설정해야 한다.  
이렇게 해야 각 js파일 내 변수가 외부 js파일과 공유되지 않기 때문이다.

### 모듈 변수 공유받기

외부의 js파일에 있는 특정 변수에 export를 붙여주면 아래와 같이 불러 쓸 수 있다.  
이를 통해 html이 직접적으로 주목할 main js파일을 하나로 정하고, 나머지 js파일들은 main js와 export, import로 엮어서 사용한다.

```javascript
// variable.js
const myVar = "APPLE BALL CAT DOLL EGG";
const myVar1 = "apple";
const myArray = [1, 2, 3, 4, 5];
// 개별 또는 묶어서 export도 가능
export { myVar };
export { myVar1, myArray };

// main.js
import { myVar } from "./variable.js";
console.log(myVar); // 'APPLE BALL CAT DOLL EGG'

// import한 변수의 이름을 바꿔서 쓰고 싶을 때 as를 쓴다.
import { myVar as myVar2 } from "./modern_test.js";
console.log(myVar2); // 'APPLE BALL CAT DOLL EGG'

// 모듈 변수 한꺼번에 가져오기 (*)
import * as modernJS from "./modern_test.js";
console.log(modernJS.myVar1, modernJS.myArray);
```

## 클래스

### 클래스 생성

아래와 같이 클래스 객체 생성 시 아래와 같이 constructor로 파라미터를 `__init__`하는 과정을 넣는다.

```javascript
class User {
  constructor(name, age, gender) {
    this._name = name;
    this._age = age;
    this._gender = gender;
  }
}
```

### Getter, Setter

`Getter`: 생성된 객체의 프로퍼티값에 접근하면 프로퍼티의 get 함수로 접근한다.  
`Setter`: 생성된 객체의 프로퍼티값을 변경하고자 할 때 해당 프로퍼티의 set 함수로 접근하도록 한다. 이를 활용하여 검증 절차를 추가할 수 있다.

```javascript
// name 프로퍼티에 대한 호출 시 get 또는 set으로 보내진다.
get name() {
  return this._name
};

set name(value) {
  if (typeof value !== 'string') {
    console.log('스트링 타입을 넣을 것!')
    return
  } else if (value.trim().length === 0) {
    console.log('이름을 입력해 주세요.')
    return
  }
  this._name = value;
};
```

개인적으로 아쉬운 점은 최초 클래스 객체 생성 시 지정되는 각 파라미터값에 대해서도 검증할 수 있었다면 더 좋았을 것 같다.

```javascript
class User {
  constructor(name, age, gender) {
    if (typeof name !== "string") {
      throw new Error("이름은 문자열이라야 합니다.");
    }
    if (typeof age !== "number" || age < 0) {
      throw new Error("나이는 양수의 숫자여야 합니다.");
    }
    if (typeof gender !== "string") {
      throw new Error("성별은 문자열이라야 합니다.");
    }

    this._name = name;
    this._age = age;
    this._gender = gender;
  }
}
```

최초 객체 생성 시 유효성 검증을 하기 위해서 지금 당장 떠오르는 방법은 위 코드처럼 constructor 내부에서 검증 과정을 넣는 것인데 이렇게 set 함수와 함께 이중으로 검증을 위한 코드를 작성하는게 올바른 방법인가 싶다.

### 상속

자식클래스가 부모클래스의 `__init__`을 받아 내 것처럼 사용하는 것을 말한다.

```javascript
class Man extends User {
  constructor(name, age, nickName) {
    super(name, age, "male");
    this._nickName = nickName;
  }

  speak() {
    console.log(`I'm ${this._name}. Just call me ${this._nickname}`);
  }
}
```

위 코드에서는 User의 초기설정값을 상속받아 쓸 수 있으므로 `this._name`, `this._age`, `this._gender`의 사용이 가능하다.  
위 케이스처럼 자식클래스의 특성 상 gender가 고정적일 경우 위처럼 super 함수를 사용하여 User 클래스에 고정 파라미터값을 전달해줄 수도 있다.  
상속받은 프로퍼티 외에 추가된 파라미터에 대해서도 프로퍼티를 정의해 주면 된다.

```javascript
class Man extends User {
  speak() {
    console.log(`My name is ${this._name}`);
    console.log(`I'm ${this.age}`);
    console.log(`I'm ${this.gender}`);
  }
}
```

특별히 파라미터 초기값에 변화가 없다면 constructor나 super 사용을 하지 않아도 된다.

### Static Method

클래스 내부 함수 중 인스턴스 생성 과정 없이 바로 사용하고 싶은 함수의 경우 static method로 지정하여 활용할 수 있다.

```javascript
class User {
  constructor(name, age, gender) {
    this._name = name;
    this._age = age;
    this._gender = gender;
  }

  static now = () => new Date();
}

console.log(User.now());
```

클래스에서 now 함수를 static으로 설정하였으므로 인스턴스 생성 없이 바로 클래스 객체에 메소드를 적용하여 함수 결과를 얻을 수 있다.

### 클로저

클로저는 특정 함수 내 중첩된 함수를 실행될 때 중첩 함수가 바깥에 있는 함수의 변수를 지속적으로 참조하는 기능을 말한다.

```javascript
const increase = (() => {
  let n = 0;
  return () => ++n;
})();
console.log(increase()); // 1
console.log(increase()); // 2
console.log(increase()); // 3
```

increase라는 변수에 함수가 할당되어 함수 내에서 n을 선언한 뒤 리턴값으로 또 다른 함수를 내보내는 코드다.  
또한 increase변수는 즉시 실행 함수가 할당되었으므로 결론적으로 increase변수에는 `(() => ++n)` 함수가 할당된다.  
그러므로 increase는 곧 함수이며 이를 실행하면 -> `(() => ++n)` 함수를 실행하는 것과 같다. 그런데 `(() => ++n)` 함수의 관점에서 n은 정의되지 않은 변수이므로 에러 발생이 예상된다.

하지만 신기하게도 `(() => ++n)` 함수를 품고 있던 함수 내에서 선언된 변수 n을 가져온다.  
더 신기한 점은 변수 n값이 increase함수를 실행할 때 마다 증가한다는 점이다.  
이런 현상을 클로저라고 하며, 이 기능을 통해 변수 n은 건드릴 수 없는 `은닉 상태`가 된다.

---

개인적으로 이해한 바로는 클로저가 특정 변수에 대해서 접근 자체가 불가하도록 조치하는 방법으로 보여진다. 그래서 예시에서도 출력만 할 뿐 실제 출력 대상의 변수에 접근할 방법은 없도록 짜여져있다.  
어떤 상황에서 클로저 기능이 필요할까? 아무도 접근할 수 없지만 값을 볼 수는 있어야 하는 상황을 생각해보니 *자동차 계기판에 있는 총 주행거리*가 떠오른다.
