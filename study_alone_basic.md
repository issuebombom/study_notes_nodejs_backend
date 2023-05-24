# 자바스크립트 기본기

## 클래스
### 클래스 생성
아래와 같이 클래스 객체 생성 시 아래와 같이 constructor로 파라미터를 `__init__`하는 과정을 넣는다.
```javascript
class User {
  constructor(name, age, gender) {
    this._name = name;
    this._age = age;
    this._gender = gender;
  };
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
    if (typeof name !== 'string') {
      throw new Error('이름은 문자열이라야 합니다.')
    }; 
    if (typeof age !== 'number' || age < 0) {
      throw new Error('나이는 양수의 숫자여야 합니다.')
    };
    if (typeof gender !== 'string') {
      throw new Error('성별은 문자열이라야 합니다.')
    }; 

    this._name = name;
    this._age = age;
    this._gender = gender
  };
};
```
최초 객체 생성 시 유효성 검증을 하기 위해서 지금 당장 떠오르는 방법은 위 코드처럼 constructor 내부에서 검증 과정을 넣는 것인데 이렇게 set 함수와 함께 이중으로 검증을 위한 코드를 작성하는게 올바른 방법인가 싶다.

### 상속
자식클래스가 부모클래스의 `__init__`을 받아 내 것처럼 사용하는 것을 말한다.
```javascript
class Man extends User {
  constructor(name, age, nickName) {
    super(name, age, 'male');
    this._nickName = nickName
  }

  speak() {
    console.log(`I'm ${this._name}. Just call me ${this._nickname}`)
  }
}
```
위 코드에서는 User의 초기설정값을 상속받아 쓸 수 있으므로 `this._name`, `this._age`, `this._gender`의 사용이 가능하다.  
위 케이스처럼 자식클래스의 특성 상 gender가 고정적일 경우 위처럼 super 함수를 사용하여 User 클래스에 고정 파라미터값을 전달해줄 수도 있다.  
상속받은 프로퍼티 외에 추가된 파라미터에 대해서도 프로퍼티를 정의해 주면 된다.
```javascript
class Man extends User {
  speak() {
    console.log(`My name is ${this._name}`)
    console.log(`I'm ${this.age}`)
    console.log(`I'm ${this.gender}`)
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

  static now = () => new Date()
}

console.log(User.now())
```
클래스에서 now 함수를 static으로 설정하였으므로 인스턴스 생성 없이 바로 클래스 객체에 메소드를 적용하여 함수 결과를 얻을 수 있다.

### 클로저
클로저는 특정 함수 내 중첩된 함수를 실행될 때 중첩 함수가 바깥에 있는 함수의 변수를 지속적으로 참조하는 기능을 말한다.
```javascript
const increase = (() => {
  let n = 0;
  return () => ++n
})()
console.log(increase()) // 1
console.log(increase()) // 2
console.log(increase()) // 3
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