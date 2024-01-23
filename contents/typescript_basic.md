# 타입스크립트 기본 활용

## Background

타입스크립트는 Microsoft에서 개발한 오픈 소스 프로그래밍 언어입니다. 이는 JavaScript의 단점을 보안하기 위해 등장했습니다.

### 자바스크립트의 주요 단점

- 자바스크립트는 코드가 실행되는 시점에서 변수의 타입이 결정됩니다. 그렇기 때문에 실행하기 전에는 타입으로 인한 오류 확인을 할 수 없었습니다. 이러한 현상이 만약 서비스 중 발생하게 된다면 서버는 곧장 다운될 수 있습니다.
- 위에서도 언급했지만 타입 체크에 어려움이 있습니다. 특정 프로그래밍 언어에서는 아예 모든 변수 지정 시 타입을 함께 지정하도록 하기도 하지만 자바스크립트는 그런 기능이 없기 때문에 자칫 잘못하면 특정 변수가 언제 Number에서 String타입으로 변경되었는지 파악하는데 오랜 시간이 소요되기도 합니다.
- 객체 접근에 제한을 걸 수 없습니다. 만약 클래스에서 Contructor로 설정한 init 변수가 외부에서 접근할 수 없도록 만들고 싶다면, 그렇게 할 수 없습니다. 특별한 접근 제한 기능이 없기 때문에 외부에서 얼마든지 클래스 내부 프로퍼티에 접근할 수 있으며 심지어 값을 변경할 수도 있습니다.

### 타입스크립트의 역할

위 주요 단점을 보완한다고 볼 수 있습니다.

- 코드 실행 전 에러 메시지를 받아볼 수 있습니다. 즉 코드 입력 단계에서부터 틀리지 않고 잘 작성할 수 있도록 돕는다고 볼 수 있습니다.
- 생성한 변수의 처음 지정된 타입을 내부에서 기억해 둡니다. 그래서 변수에 할당된 값이 항상 같은 타입을 유지할 수 있도록 돕습니다.
- 클래스에서 private 기능을 사용하여 프로퍼티의 외부 접근을 방지할 수 있습니다.

### 타입스크립트에 적용된 컴파일러

타입스크립트 컴파일러인 tsc는 우리가 앞으로 입력할 타입스크립트 코드를 자바스크립트 코드로 변환해주는 역할을 합니다. 그래서 언어 번역을 해주는 기능을 컴파일러 라고 부릅니다.  
tsc의 변환 기능이 있어 자바스크립트 코드의 실행 전 언어 변환 과정에서 에러 체크를 하게되므로 정적 타입 검사 수행이 가능해진 것입니다.

## 타입스크립트 설치 및 설정

> 이미 node.js가 설치되어 있다는 것을 전제로 합니다.

```zsh
# 설치
npm i typescript -g

# 타입스크립트 초기화 (tsconfig.json 생성)
tsc --init

# ts파일(index.ts) 컴파일
tsc index.ts

# @types 패키지를 위한 .d.ts파일 생성 명령
tsc index.js --declaration --emitDeclarationOnly
```

### tsconfig.json

타입스크립트 프로젝트의 설정파일입니다.  
컴파일러가 `es5`를 기준으로 번역할지 `es2016` 기준으로 번역할지 설정할 수 있습니다.  
해당 파일에서 `compilerOptions - strict`를 `true`로 설정하는 것을 권장하며 개발 단계에서는 `compilerOptions - sourceMap`을 `true`로 설정하는 것을 권장합니다.  
`include`와 `exclude` 옵션을 통해 특정 파일 또는 디렉토리를 컴파일에서 포함 또는 제외시킴을 의미합니다.

여기서 compilerOptions를 strict로 설정한다는 것은 엄격한 타입 검사를 수행함을 의미합니다. 가령 잠재적으로 undefined가 될 수 있는 값들에 대해서 검사하거나 함수의 파라미터 또는 변수의 타입이 명시되어 있지 않으면 반드시 명시하도록 경고합니다.

sourceMap 설정은 코드 디버깅에 도움을 주는 옵션으로 코드 실행 시 에러가 발생하면 해당 에러가 타입스크립트 코드 상에서 어느 소스 코드에 해당하는지 알려줍니다.

`.d.ts`파일은 자바스크립트 라이브러리를 타입스크립트 코드에서 사용 가능하도록 해주는 파일입니다. 구체적으로는 `@types` 폴더에 존재하며 이 파일들은 자바스크립트의 외부 라이브러리에 대한 타입 정보를 제공하는 역할을 합니다. 그래서 프로젝트에 적용하고자 하는 라이브러리는 그에 해당하는 .d.ts파일이 함께 제공되어야 합니다.

#### .d.ts 생성 실습

개인 용도로 사용하기 위해 특정 함수를 export하는 js파일을 생성하였고, 이를 사용하기 위해 .d.ts를 생성하는 과정을 실습합니다.

먼저 tsconfig.json 설정을 변경합니다.

```JSON
"allowJs": true // TypeScript 프로젝트에 JavaScript 파일 허용 여부
"checkJs": true // JavaScript 파일 타입 체크 여부
```

```javascript
// 덧셈 함수

/**
 * @param {number} a
 * @param {number} b
 * @returns {number}
 */
export function add(a, b) {
  return a + b;
}
```

`JSDoc` 주석을 통해 함수의 파라미터와 리턴값의 타입을 알려주고 있는 것을 확인할 수 있습니다. 해당 파일명이 `test.js`라고 한다면 아래 명령어를 통해 .d.ts를 생성할 수 있습니다.

```zsh
npx tsc add.js --declaration --allowJs --emitDeclarationOnly --outDir types
```

이후 types/add.d.ts파일을 확인하면 아래와 같이 생성된 것을 확인할 수 있습니다.

```typescript
/**
 * @param {number} a
 * @param {number} b
 * @returns {number}
 */
export function add(a: number, b: number): number;
```

그래서 이후 add 함수를 타입스크립트 코드에서도 import해서 사용이 가능해 집니다.

```typescript
// test.ts
import { add } from './add';
console.log(add(1, 2)); // 3
```

```zsh
# 실행 (터미널에 3이 출력됩니다.)
npx ts-node test.ts
```

## 타입스크립트의 기본 실행 흐름

### 초기 설정

```zsh
# 글로벌 설치
npm i typescript -g

# Node 프로젝트 생성
npm init -y

# 타입스크립트 초기화 (tsconfig.json 생성)
tsc --init --rootDir ./src --outDir ./dist --esModuleInterop --module commonjs --strict true --allowJS true --checkJS true

mkdir src
```

package.json의 script 설정

```JSON
"scripts": {
    "start": "tsc && node ./dist/index.js",
    "build": "tsc --build",
    "clean": "tsc --build --clean"
},
```

### 개발 단계

타입스크립트로 개발 단계에서는 아래와 같은 흐름으로 코드를 테스트 합니다.

> -> 타입스크립트로 코드를 작성합니다.  
> -> `tsc --build` 명령을 실행하여 타입스크립트 코드를 dist 폴더 안에 js 언어로 컴파일 합니다. (해당과정에서 컴파일 에러를 발견합니다.)  
> -> `tsc && node ./dist/index.js` 명령을 통해 생성된 index.js파일을 실행합니다.

위 코드에서는 아래 내용을 전제로 합니다.

1. 메인 실행파일이 `index.js`임을 전제로 하고 있습니다.
2. tsconfig.json에서 `"include": ["src/**/*"]`와
   `"exclude": ["./dist"]`로 설정이 되어 있습니다.
3. 컴파일 대상인 ts파일은 `src` 폴더 내 위치해야 합니다.

### 타입스크립트의 타입 적용법

코드를 통해 타입이 어떻게 적용되는지 직접 확인하자

```typescript
//* 변수에 타입 지정
const name: string = 'apple';
const age: number = 25;

//* 함수 (파라미터: 타입): 리턴 타입
// arrow function
const validName = (name: string): boolean => name.length >= 4;
// 선언형
const validPassword = function (password: string): boolean {
  return password.length >= 8;
};
// 대입형
function validAge(age: number): boolean {
  return age >= 20;
}
// 함수에 리턴값이 없다면 void
const printName = (name: string): void => {
  console.log(name);
};

//* 배열
// 배열(내부 타입이 하나로 고정일 경우
const array: number[] = [1, 2, 3, 4, 5];
const array2: string[] = ['apple', 'ball', 'cat'];
// 배열(내부 타입이 두 개일 경우)
const array3: (string | number)[] = [1, 'apple', 2]; // union이라 부른다.

//* 객체
interface Client {
  name: string;
  age: number;
  role: 'admin' | 'client'; // 선택지를 줄 수도 있다.
}
const client: Client = { name: 'apple', age: 99, role: 'admin' };

//* 변수에서의 union
let value: string | number = 3;
value = 'apple'; // 가능

//* 튜플 (튜플은 내부 데이터가 고정되는 것이 원칙이다)
const tuple: [string, number, string] = ['apple', 3, 'ball'];

//* enumerate (User 객체의 프로퍼티를 직접 명시한다.)
// string과 number만 사용 가능
// 클래스처럼 첫글자 대문자로 쓰는 경향
// 상수를 그룹화하여 관리하기 위한 수단으로 주로 사용
// 상수값은 변하지 않는 것이 원칙
enum User {
  USER_KEY = 'apple',
  API_KEY = 28,
  STATUS = 1,
}
// enum의 프로퍼티를 꺼내어 사용하는 느낌이다.
// enum은 상
const key: User = User.USER_KEY;

//* readonly (최초 지정한 값을 변경할 수 없다.)
interface Client {
  readonly name: string;
  readonly age: number;
}
const client: Client = { name: 'apple', age: 18 };
// client.name = 'ball' // 에러 발생

//* any와 unknown
/*
  any는 그냥 쓰면 안된다고 생각하자
  unknown은 타입 명시 전이라는 표시에 사용
*/
const yourname: unknown = 'ball';
console.log(yourname); // ball
// let myname: string = yourname // unknown타입은 string이 아님(오류)
// alias로 임시 타입 지정 가능
let myname: string = yourname as string;

//* object literal ( 자바스크립트 객체랑 뭐가 다르지?? )
// obj.b -> [1, 2, 3]이 출력되어 그냥 쓰면 된다.
const obj = {
  a: 1,
  b: [1, 2, 3],
  c: 'apple',
};

// -- 유틸리티 타입
//* Partial<T>
// interface 내 프로퍼티 중 일부만 사용 가능하도록 하는 기능
// 아래처럼 프로퍼티 업데이트에 사용할 때 주로 쓰이는 듯 하다
interface Todo {
  title: string;
  description: string;
}
function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}
const todo1 = {
  title: 'organize desk',
  description: 'clear clutter',
};
const todo2 = updateTodo(todo1, {
  description: 'throw out trash',
});

//* Required<T>
interface Client {
  name: string;
  age: number;
  address?: string; // '?'는 있어도 없어도 된다는 뜻
}
type RequiredClient = Required<Client>;
const client: Client = { name: 'apple', age: 999 }; // 가능
// 아래 코드는 address도 입력 필수
// const requiredClient: RequiredClient = { name: 'apple', age: 999 };

//* Readonly<T>
interface Client {
  name: string;
  age: number;
}

const immutableClient: Readonly<Client> = {
  name: 'apple',
  age: 990,
};
// immutableClient.name = 'ball' // 재할당 안됨(읽기 전용)

//* Pick<T, K>
// 타입 T에서 선택한 K 프로퍼티만 사용 가능
interface Client {
  name: string;
  age: number;
  address: string;
}

type pickedClient = Pick<Client, 'name' | 'age'>;
// const client: pickedClient = {name: 'cat', age: 4, address: 'aaa'} // address는 입력 불가

//* Omit<T, K>
// 타입 T에서 선택한 K 프로퍼티는 사용 불가
interface Client {
  name: string;
  age: number;
  address: string;
}

type omittedClient = Omit<Client, 'name' | 'age'>;
// const client: omittedClient = {name: 'cat', age: 4, address: 'aaa'} // name과 age는 입력 불가
```

위에서 소개한 유틸리티 타입 외 타입은 아래 링크 참조  
[유틸리티 타입](https://www.typescriptlang.org/ko/docs/handbook/utility-types.html)

## 클래스에서의 타입스크립트

```typescript
class Person {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  sayHello() {
    console.log(`안녕하세요! 제 이름은 ${this.name}이고, 나이는 ${this.age}살입니다.`);
  }
}

const person = new Person('Spartan', 30);
person.sayHello();
```

Person의 프로퍼티로 사용되는 name과 age이 constructor보다 상단에 위치하여 타입 지정이 되고 있다.  
또한 Person이라는 클래스의 인스턴스를 생성할 때 필요한 name과 age 변수 즉 constructor에 정의된 변수의 타입도 지정되어 있다.  
얼핏 보기에는 동일한 변수가 중복 타입 지정되는 것으로 보여질 수 있으나 class 최상단에 정의된 변수는 this.name과 this.age에 대한 것이며, constructor 내부에 정의된 변수는 인스턴스 생성 시 필요한 파라미터에 대한 타입 정의를 의미한다.

### 클래스 접근 제한자

타입스크립트의 강점 중 하나는 클래스 프로퍼티의 접근을 제한할 수 있다는 점이다.

- `public`
  - **클래스 외부에서도 접근이 가능한 접근 제한자**입니다.
  - 접근 제한자가 선언이 안되어있다면 **기본적으로 접근 제한자는 public** 입니다.
- `private`
  - **클래스 내부에서만 접근이 가능한 접근 제한자**입니다.
  - 보통은 클래스의 **속성은 대부분 private으로 접근 제한자를 설정**합니다.
  - 클래스의 속성을 보거나 편집하고 싶다면 별도의 getter/setter 메서드를 마련합니다.
- `protected`
  - **클래스 내부와 해당 클래스를 상속받은 자식 클래스에서만 접근**이 가능한 접근 제한자입니다.

```typescript
class Person {
  private name: string;
  private age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  public sayHello() {
    console.log(`안녕하세요! 제 이름은 ${this.name}이고, 나이는 ${this.age}살입니다.`);
  }
}
```

### upcasting, downcasting

부모 클래스의 상속을 받은 자식 클래스 이 둘의 관계에서 부모가 지닌 타입과 자식이 가진 타입을 각각 `슈퍼 타입`과 `서브 타입`으로 부른다.  
이는 클래스 자체가 하나의 타입으로서 활용될 수 있기 떄문이다. 그런 의미에서 아래와 같이 쓰인다.

```typescript
const person: Person = new Person('apple', 99);
```

Person은 클래스명이지만 타입을 정의하는 자리에 비치되어 있음을 알 수 있다. 사실 위 코드에서 타입 자리에 Person을 입력하지 않아도 오류가 발생하지 않는다. 왜냐하면 new Person을 통해 인스턴스를 생성하므로 이미 타입이 정의된 바와 같기 때문이다.  
하지만 upcasing과 downcasting과 같은 경우에는 확실한 타입 지정을 필요로 한다.

먼저 upcasting 케이스를 살펴보자.

```typescript
class Person {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  sayHello() {
    console.log(`이름: ${this.name} 나이: ${this.age}살`);
  }
}

class Admin extends Person {
  role: string;

  constructor(name: string, age: number) {
    super(name, age);
    this.role = 'admin';
  }

  sayRole = () => {
    console.log(this.role);
  };
}

const admin: Admin = new Admin('apple', 99);
admin.sayRole();

// admin이 Person 클래스
const person: Person = admin; // upcasting
// person.sayRole(); // 작동 불가
```

admin이라는 인스턴스가 Person 타입으로 재정의 되고 있다. admin은 super메서드를 통해 person의 상속을 받고 있었기 때문에 Person 타입이 입력받아야할 값을 그대로 지니고 있어 person 인스턴스 생성의 재료로 사용되는데 문제가 없다.  
다만 이렇게 될 경우 person이 제아무리 admin을 그대로 받았다 할지라도 Admin 클래스에서 지니고 있던 sayRole 함수는 더이상 사용이 불가하다. 이제 admin이 부모 클래스를 기준으로 재정의 되었기 때문이다.  
admin은 자신과 부모 클래스가 지닌 함수 모두에 접근이 가능한데 upcasting을 쓸 경우 자식 클래스의 함수 접근이 불가해지고, 이러한 의도가 필요한 곳에 쓰일 수 있겠다.

downcasting은 그 반대다. 부모가 자식 클래스를 타입으로 받아들이므로 부모가 지니지 못했던 자식 클래스의 함수에 접근이 가능해진다.

```typescript
const person: Person = new Admin('apple', 99);
// person.sayRole() // 불가능
const adminPerson: Admin = person as Admin; // downcasting
adminPerson.sayRole(); // 가능
```

Admin 클래스로 인스턴스를 생성할 때 슈퍼 타입으로 생성되었다. 즉 Person 클래스의 인스턴스가 생성된 것이다. 그러니 당연히 자식 클래스에 존재하는 sayRole 함수에는 접근이 불가하다.  
하지만 person 인스턴스의 타입을 일시적으로 서브 타입으로 변경하여 자식 클래스 인스턴스로 생성한다면(downcasting한다면) sayRole 함수에 접근이 가능하다.  
이는 다소 이상한 그림이기는 하다. 왜냐하면 저렇게 할 바에 그냥 처음부터 자식 클래스인 Admin 클래스 인스턴스를 생성하는 것과 차이가 없기 때문이다.  
자식 클래스는 부모 클래스의 상속을 받으므로 부모 클래스의 함수에도 접근이 가능하기 때문이다.

## 추상 클래스

자식 클래스에서 공통적으로 갖고 있는 함수(이름은 갖으나 기능이 다른)를 부모 클래스에서 일괄 처리해주기 위한 기능 구현에서 활용된다. 예시를 보자.

```typescript
// 추상클래스와 추상함수
abstract class Shape {
  abstract getTotal(): number;

  printTotal() {
    console.log(this.getTotal());
  }
}

class Case1 extends Shape {
  age: number;

  constructor(age: number) {
    super();
    this.age = age;
  }

  getTotal() {
    return this.age * 10;
  }
}

class Case2 extends Shape {
  name: string;

  constructor(name: string) {
    super();
    this.name = name;
  }

  getTotal() {
    return this.name.length * 10;
  }
}

const case1 = new Case1(30);
const case2 = new Case2('apple');
case1.printTotal();
case2.printTotal();
```

Case1클래스와 Case2 클래스는 공통적으로 `getTotal 함수`를 지니고 있다. 하지만 보다시피 작동 방식(기능)이 다르다.  
만약 이 함수의 실행 결과를 프린트해주는 함수도 필요하다면 Case1과 Case2에 각각 동일한 형태의 프린트 함수를 생성해야 할 것이다.  
하지만 이러한 코드 중복을 방지하는 방법으로 위 코드와 같은 추상 클래스를 도입한다. Case1과 Case2를 자식태그로 만들어 그 부모 태그인 Shape에서 getTotal함수를 추상 함수로 지정한다.  
여기서 `abstract`라는 키워드를 사용하며 `추상 함수`로 등록된 getTotal을 지니고 있는 이상 Shape 클래스도 추상 클래스로의 지정이 필요하다.  
이렇게하면 추상 클래스에서의 getTotal은 Shape 클래스 내에서 참조용으로서 표기되며 이를 기반으로 추가 기능 구현이 가능해 진다. 그 결과가 printTotal 함수이다.  
이제 자식 클래스 인스턴스는 부모 클래스의 `printTotal 함수`에 접근이 가능하게 되므로 불필요한 중복 코드 작성을 줄일 수 있게 되었다.
