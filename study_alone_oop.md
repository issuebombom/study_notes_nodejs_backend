# 자바스크립트 객체지향 프로그래밍

## 기본

```javascript
class User {
  constructor(email, birthdate) {
    this.email = email;
    this.birthdate = birthdate;
  }

  buy(item) {
    console.log(`${this.email} buys ${item.name}`);
  }
}

const user1 = new User('chris123@google.com', '1992-03-21');
const user2 = new User('jerry99@google.com', '1995-07-19');
const user3 = new User('alice@google.com', '1993-12-24');

const item = {"name": "pickle"}
user1.buy(item)

>> chris123@google.com buys pickle
```

- this가 파이썬에서 self와 동일한 기능을 갖는다.
- 클래스 내 constructor는 **init**과 비슷하게 사용되는 것 같다. js에서는 클래스의 기본 프로퍼티 설정을 보통 constructor에 담는다고 한다.
- 클래스 내 생성된 buy메소드도 객체 생성 및 변수 지정해서 해당 변수의 메소드 형태로 사용한다는 점은 파이썬과 거의 일치한다.

## 상속

```typescript
class Animal {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  introduction() {
    console.log(`저는 ${this.name}입니다.`);
  }
}

class Dog extends Animal {
  age: number;

  constructor(name: string) {
    super(name);
    this.age = 5;
  }

  makeSound() {
    console.log('멍멍!'); // 부모의 makeSound 동작과 달라요!
  }
}

const dog = new Dog('멍멍이');
console.log(dog.introduction());
```

Dog 클래스에 introduction 메서드가 없지만 Animal 클래스를 부모클래스로 두고 있어 접근이 가능하다.  
하지만 Animal 클래스가 makeSound 메서드에 접근할 수는 없다. 이를 해결하기 위해서는 upcasting이라는 방법을 써야한다.

## 클래스 설계 원칙 SOLID

### 단일 책임 원칙 (SRP)

하나의 클래스는 하나의 책임만을 지녀야 한다는 원칙을 의미한다.  
가령 User 클래스를 설계한다면 유저라는 속성과 관련된 액션 또는 프로퍼티만을 지니도록 해야 한다. 예를 들어 예약하기, 포스터 작성하기 등 유저가 취할 수 있는 액션으로 보이지만 사실 유저의 어떠한 상태가 변경되는 것이 아니므로 이를 User클래스에 담아서는 안된다. 유저의 정보를 변경하거나, 유저의 정보를 조회하거나 하는 등의 액션이 User클래스에 담길 수 있을 것이다.

### 개방 폐쇄 원칙 (OCP)

클래스는 확장에 있어서 개방적이며, 수정에 대해서는 폐쇄적이어햐 한다는 원칙이다. 즉 확장성을 고려한 클래스로 설계되어야 한다는 의미이다. 이는 User 클래스를 생성한 이후 해당 유저의 게시글 작성 및 지금까지 작성한 게시글 수를 조회하는 기능을 추가한다고 했을 때 User클래스에 수정을 가하는 것이 아니라 Post클래스를 따로 생셩하여 이를 조합하거나 상속을 활용할 것을 고려해야 한다는 의미이다. 새로운 기능이 추가될 떄 마다 기존에 설계한 User클래스를 손보게 될 수 밖에 없다면 이는 클래스를 잘못 설계한 것에 해당한다.

### 리스코트 치환 원칙 (LSP)

자식 클래스를 생성할 때 부모클래스의 기능을 수정하지 않고도 부모 클래스와 호환이 되어야 하며 그럼으로 인해 자식 클래스가 부모 클래스로 대체될 수 있어야 한다는 원칙이다. 즉 부모 자식간 계층(hierarchy)이 논리적으로 정립되어야 함을 의미한다.  
가령 User라는 부모 클래스 아래 Admin이라는 자식 클래스를 둔다면 이는 User의 개념이 Admin보다 상위 계층으로 취급이 되어야 한다는 말인데 일반적으로 User는 클라이언트 즉 고객에 해당하므로 관리자에 해당하는 Admin 클래스의 부모가 될 수 없다. Admin 클래스는 User클래스를 대체할 수 없고, User 클래스의 정의가 모호해 진다.  
차라리 Human 클래스를 부모 클래스로 두고 자식 클래스로 User와 Admin을 각각 둔다면 좀 더 계층 정립이 잘 되었다고 볼 수 있을 것이다. 왜냐하면 Human 클래스가 User 또는 Admin클래스로 대체가 가능하며 이 또한 자식 클래스를 둘 수 있는 확장성을 지니기 때문이다. 이와 같은 부분을 신경써야 한다는 말이다.

### 인터페이스 분리 원칙 (ISP)

클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 합니다. 여기서 인터페이스는 클래스에서의 프로퍼티에 해당한다 볼 수 있다. 의존하지 말아야 한다는 말은 즉 의존성을 제거해야 함을 의미한다. User 클래스의 특정 메서드를 작동하는데는 Admin 클래스의 특정 프로퍼티값이 활용된다고 가정해보자. 이를 보고 User클래스가 Admin 클래스를 의존한다고 하는데, 이럴 경우 Admin 클래스의 수정이 User 클래스에게 영향을 끼칠 수 있음을 의미한다.  
이는 유지 보수에 있어 문제가 될 수 있기 때문에 각 클래스의 인터페이스를 최소 단위로 구현하여 해당 인터페이스의 변경이 미치는 영향이 최소가 되도록 할 것을 권장하는 원칙이다.

### 의존성 역전 원칙 (DIP)

하위 수준의 모듈(구현 클래스)은 상위 수준의 모듈에 의존해야 하며 이 둘 모두는 추상 클래스에 의존해야 한다는 원칙을 말한다.

```typescript
interface MyStorage {
  save(data: string): void;
}

class MyLocalStorage implements MyStorage {
  save(data: string): void {
    console.log(`로컬에 저장: ${data}`);
  }
}

class MyCloudStorage implements MyStorage {
  save(data: string): void {
    console.log(`클라우드에 저장: ${data}`);
  }
}

class Database {
  // 상위 수준 모듈인 MyStorage 타입을 의존!
  // 여기서 MyLocalStorage, MyCloudStorage 같은 하위 수준 모듈에 의존하지 않는게 핵심!
  constructor(private storage: MyStorage) {}

  saveData(data: string): void {
    this.storage.save(data);
  }
}

const myLocalStorage = new MyLocalStorage();
const myCloudStorage = new MyCloudStorage();

const myLocalDatabase = new Database(myLocalStorage);
const myCloudDatabase = new Database(myCloudStorage);

myLocalDatabase.saveData('로컬 데이터');
myCloudDatabase.saveData('클라우드 데이터');
```

위 코드에서 database 클래스의 상위 클래스는 MyLocalStorage와 MyCloudStorage로 보일 수 있지만 더 상위인 MyStorage 인터페이스에 의존하고 있는 모습을 볼 수 있다.  
그래서 설명대로 MyStorage 인터페이스 즉 추상 클래스의 역할을 하는 인터페이스를 database, MyLocal, MyCloud 모두가 의존하고 있는 구조로 설계되어 있는 모습을 확인할 수 있으며 이와 같은 상위, 하위 클래스 계층을 고려해야 한다.
