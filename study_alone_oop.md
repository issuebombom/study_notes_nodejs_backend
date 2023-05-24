# 자바스크립트 객체지향 프로그래밍
파이썬과 크게 다르지 않다.
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
- 클래스 내 constructor는 __init__과 비슷하게 사용되는 것 같다. js에서는 프로퍼티를 보통 constructor에 담는다고 한다.
- 클래스 내 생성된 buy메소드도 객체 생성 및 변수 지정해서 해당 변수의 메소드 형태로 사용한다는 점은 파이썬과 거의 일치한다.