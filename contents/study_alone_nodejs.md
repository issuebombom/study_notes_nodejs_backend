# 자바스크립트 Node.js 기본기

## console.log 즉시 실행

node.js가 설치되어있다면 아래와 같은 명령어로 console.log를 터미널에서 실행이 가능하다.
참고로 js확장자는 명시해주지 않아도 잘 작동한다.

```bash
node nodefile.js
```

## REPL 모드

Read, Eval, Print, Loop  
터미널에서 node를 실행하면 자바스크립트 언어에 대한 결과값을 바로 확인할 수 있다.  
python이나 mysql 입력해서 즉석해서 코드를 실행하는 것과 같은 형식이다.  
종료는 `.exit`를 입력한다.

## 모듈

자바스크립트는 require("js파일경로")를 변수 지정하여 모듈로 활용할 수 있다. 파이썬의 from import와 같은 개념인 것 같다.

> 🚨 `주의`  
> 모듈로서 활용할 js파일은 외부에서 해당 메소드를 쓸 수 있도록 아래 코드에서처럼 export를 해줘야 한다.

```javascript
function add(a, b) {
  return a + b;
}

exports.add = add; // exports뒤에 이름(add)이 외부에서 사용할 이름으로 결정난다.
```

또한 사용하는 입장에서도 해당 모듈을 불러오기 위한 require함수를 써야한다.

```javascript
let a = require("./math.js");
console.log(a.add(3, 5));
```

> `Tip`  
> exports를 쓰면 함수 뿐 아니라 변수까지도 모듈로 쓸 수 있다는 점 참고

만약 외부로 보낼 함수가 많아질 경우에는 object형태로 담아서 전달할 수 있다. 이때는 `module.exports`라는 코드를 써야한다.

```javascript
let caculator = {
  let PI = 3.14,
  add: (a, b) => a + b,
  subtract: (a, b) => a - b
};

module.exports = caculator;
```

위와 같이하면 외부에서 PI, add, subtract 모두 사용 가능해진다. `require("js_file_path")`과 같은 형태로 받는 것은 동일하다.

### 코어모듈

node.js에서 기본적으로 이미 만들어 놓은 모듈이 있다.  
python의 itertools, os, datetime 등등과 같다.  
require로 직접 이름을 명시해서 사용하면 된다.  
코어모듈의 예시를 몇 가지 보자

```javascript
// 현재 디렉토리에 있는 파일명을 리스트에 담는다.
const fs = require("fs"); // file system
let fileList = fs.readdirSync(".");
console.log(fileList) >>
  ["math.js", "study_alone_nodejs.js", "study_alone_nodejs.md"];

// new 파일 생성 후 hello 내용을 기록한다.
fs.writeFileSync("new", "hello!");

// cpu 성능을 확인한다.
const os = require("os"); // operating system
console.log(os.cpus());
```

> `Tip`  
> 코어 모듈 목록 확인  
> https://nodejs.org/dist/latest-v12.x/docs/api/

### 서드파티모듈

python에서 pip를 쓴다면 node.js에서는 npm  
서드파티모듈을 설치하면 자체적으로 `package-lock.json`파일과 `node_modules`라는 디렉토리가 생성된다.

`package-lock.json`: 해당 디렉토리에 설치된 서드파티를 확인할 수 있다. 당연하지만 서드파티모듈을 설치하면 의존성을 지닌 모든 서드파티모듈이 다함께 설치된다.  
재밌는 부분은 각 모듈이 어떤 모듈과의 의존성을 지니고 있는지 json 파일로 확인할 수 있다는 점이다.  
`node_modules`: 실제 모듈이 저장되는 디렉토리이다. 파이썬에서 site-packages와 같은 역할

## 비동기 실행

비동기 함수를 활용한 실행을 말한다. 예시를 보자

```javascript
const fs = require("fs");

console.log("start");

fs.readFile("./new", "utf8", (error, data) => {
  console.log(data);
});

console.log("end");
```

위 경우 readFile은 비동기 함수로 작동하여 start, end 다음에 출력된다.  
비동기 함수의 경우 항상 뒤에 `콜백 함수` 입력이 요구된다. 위 예시에서 arrow function으로 성공 유무에 따라 error과 data로 값을 받는 함수가 위치하고 있는데 이렇게 결과를 받아서 처리할 함수를 말한다.  
이와 같이 비동기 실행을 활용한 코딩을 `비동기 프로그래밍`이라 부른다.

> `Tip`  
> Node.js를 잘 하려면 비동기 프로그래밍을 잘하는 것이 중요하다.  
> Why? 아직은 모르겠다...

## 프로세스와 스레드

Node.js의 비동기 실행을 이해하기 위해서는 프로세스와 스레드의 개념을 알아야 한다.  
프로세스와 스레드를 비유하자면 식당과 직원들에 비유할 수 있겠다.  
메뉴를 주문 받아 음식을 내어주는 `프로세스`를 수행하기 위해서 10명의 직원이 대기 중이라고 했을 때 이들을 각각 스레드라고 볼 수 있다. 이때 한 명의 `메인스레드`가 요청을 받아 '요리'와 같은 시간이 오래 걸리는 작업을 `나머지 9개의 스레드`에게 넘겨주고, 이 스레드의 업무가 완료되면 메인스레드가 최종 결과를 전달한다.  
위 예시는 Node.js에서 File I/O 작업에 대한 비동기 실행 방식이다. 어떠한 작업이냐에 따라 실행 방식은 다양하다.

> 🚨 `주의`  
> 메인스레드의 주요 업무는 CPU 연산과 요청, 응답 작업이다. 여기서 만약 무거운 CPU 연산 작업이 들어올 경우 메인스레드의 처리속도가 급저하 되기 때문에 성능이 떨어지게 된다. 그러므로 무거운 연산이 요구되는 `CPU intensive job`을 처리하는 서버를 구축해야 한다면 Node.js는 적합하지 않을 수 있다. 이러한 job은 이미지 처리, 딥러닝 등 CPU가 GPU에게 처리를 토스하는 작업들이 되겠다.

## 이벤트

이벤트 실행의 기본 동작이 어떤지 살펴보자

```javascript
const EventEmitter = require("events");
const myEmitter = new EventEmitter();
myEmitter.on("test", () => {
  console.log("Success");
});

myEmitter.emit("test");
```

events 모듈을 불러오면 클래스가 불러와지며 이를 new 객체 생성을 통해 활용하는 방식이다.
이벤트 객체의 메소드에서 `on` 메소드를 통해 이벤트를 설정한다.  
여기서는 이벤트 명을 'test'로 지정하여 'Success'라는 문자열을 콘솔에 출력하는 콜백함수(이를 `이벤트 핸들러` 또는 `리스너`라고도 부른다.)를 등록했다.  
`emit` 메소드를 통해 'test'라는 이벤트를 발생시키면 'Success' 문자열이 콘솔에 출력된다.

> 🚨 `주의`  
> 당연한 이야기지만 특정 event가 사전에 등록되지 않은 상태에서 emit 메소드를 실행하면 아무 일도 일어나지 않는다.

_이벤트는 여러 개 등록할 수 있다. 가령 아래 처럼 말이다._

```javascript
const EventEmitter = require("events");
const myEmitter = new EventEmitter();

myEmitter.on(
  "test",
  (callback1 = () => {
    console.log("Success");
  })
);
myEmitter.on(
  "test",
  (callback2 = () => {
    console.log("Nice");
  })
);
myEmitter.on(
  "test",
  (callback3 = () => {
    console.log("Good Job");
  })
);
```

동일한 이벤트에 대한 콜백함수는 순차적으로 쌓이게 되며 이벤트 발생시 순차적으로 실행된다.

_이벤트 객체의 또다른 메소드를 알아보자_

- .once: 딱 한번만 발생시킬 이벤트 핸들러를 등록한다. 해당 이벤트가 두번째로 emit되면 반응이 없다.

- .listeners: 이벤트에 등록된 이벤트 핸들러들을 리스트에 담아 출력해준다.
  ```javascript
  console.log(myEmitter.listeners('test'));
  >> [ [Function: callback1], [Function: callback2], [Function: callback3] ]
  ```
- .off: on 메소드와 반대되는 개념으로 특정 이벤트 핸들러를 제외할 때 쓴다. 제외를 위해서는 위 코드와 같이 이벤트 핸들러가 변수로 명시되어 있거나 기타 방법으로 지정이 가능한 상태여야 한다.

_이벤트에 파라미터 전달하기_  
args를 하나씩 나열할 수도 있지만 주로 아래와 같이 오브젝트 또는 리스트로 전달한다.

```javascript
const EventEmitter = require("events");
const myEmitter = new EventEmitter();

myEmitter.on("test", (info) => {
  console.log(info["name"]);
});

const obj = { name: "apple", age: "50" };
myEmitter.emit("test", obj) >> "apple";
```

## 비동기 함수의 실행 순서

1. 비동기 함수를 만나면 일단 콜백 함수를 등록한다. (정확히 어디에 등록되는건지는 모르겠지만 '들고 있는다.' 라고 생각하면 좀 더 편할 것 같다.)
2. 스크립트 코드를 모두 실행한 뒤 `Event Loop`라는 로직이 시작된다.
   - `Event Loop`는 지금까지 들고 있었던 콜백 함수의 실행 조건이 충족되었는지 확인하고, 실행하는 로직을 무한반복한다. 콜백 함수를 다 털어낼 때 까지
3. 들고 있던 콜백 함수 중 조건이 충족된 콜백 함수는 `Queue`에 넣는다.
4. `Queue`에 들어온 함수는 즉각 실행된다.
   - 큐에 들어온 함수가 실행되는 동안 또 다른 콜백 함수가 조건을 충족한다면 Queue에 들어와 줄을 서게 된다.

> `비동기 실행의 특징을 명확히 파악하기`  
> Event Loop는 크게 아래 두 가지를 수행한다고 볼 수 있다.  
> `1. 조건을 확인한다.`  
> `2. 실행한다.`  
> 이 수행과제에서 집중할만한 점은 아래와 같다.  
> `1. 콜백함수의 실행 조건이 언제 달성되느냐?`  
> `2. 콜백함수의 실행은 언제 끝나느냐?`  
> 1번의 경우 API 통신에서 response가 얼마나 빠르냐가에 따라 콜백 함수 실행 우선권이 달라짐을 알 수 있다.  
> 즉 조건 달성이 빠른 순으로 Queue에 들어가 줄세워진다. 이 시점에서 실행 순서가 결정된 셈이다.  
> 2번의 경우 Queue에 들어간 순서대로 콜백 함수가 수행되므로 _뒤쪽 함수가 앞쪽 함수보다 계산량이 적다 하더라도 앞쪽 함수가 끝날 때까지 기다려야 한다._

하지만 테스트를 통해 미묘한 차이에서는 위 로직이 항상 같은 결과를 주지 않을 수도 있다는 점을 발견했다.

```javascript
console.log("1");

setTimeout(() => {
  for (let i = 0; i < 10; i++) {}
  console.log("4");
}, 301);

console.log("2");

setTimeout(() => {
  for (let i = 0; i < 1000000000; i++) {}
  console.log("3");
}, 300);
```

위에서 설명한 비동기 함수의 실행 순서대로라면 1, 2, 3, 4 순으로 출력되는게 맞다.  
비록 3번이 출력되는데 필요한 계산량이 4번보다 월등히 높지만 Queue에 들어간 순서가 3번이 먼저이기 때문이다.  
하지만 해당 코드를 반복적으로 실행해보면 가끔씩 4번이 3번보다 먼저 나올 때가 있었다. 왜그럴까?

nodejs.org의 가이드 문서에 따르면 프로세스의 성능에 따라 달라질 여지가 있는 것으로 보인다.

```javascript
// timeout_vs_immediate.js
setTimeout(() => {
  console.log("timeout");
}, 0);

setImmediate(() => {
  console.log("immediate");
});
```

위 코드를 보면 실상 실행 조건 달성 시간이 똑같아 보인다. 하지만 출력 순서는 확정적이지 않다고 한다. (non-deterministic)  
이에 대해 timing will be bound by the performance of the process (which can be impacted by other applications running on the machine) 라고 말한다.

## 웹서버 구현

노드는 Django처럼 웹 서버 구축을 위한 프레임워크가 아니다. 현재로서는 Flask에 더 가까운 느낌이다. 왜냐하면 Django프레임워크처럼 프로젝트 생성하는 순간 MVT 구조가 펼쳐지는게 아니라 하나부터 열까지 빌드업해야 할 것 같이 생겼기 때문이다.

### 웹 서버 구현 기본

```javascript
const http = require("http");

let server = http.createServer((request, response) => {
  response.end("<h1>Hello</h1>");
});
server.listen(3000); // 포트번호
```

위 코드와 같이 http 코어모듈을 사용하여 간단하게 웹 서버를 구축할 수 있다.  
위 코드의 이미를 해석하자면 아래와 같다.

> 개인 컴퓨터에서(127.0.0.1) 3000번 포트 번호를 가지고 실행되고 있던(:3000) 서버 프로그램과  
> http라는 프로토콜로 통신을 시작하겠다는 뜻

> `Tip`  
> URL에서 `www.naver.com` 영역을 `Host` 영역을 `도메인 네임`으로 나타냈다고 한다.  
> 도메인 영역은 `Root Domain`, `Top-Level Domain(TLD)`, `Second-Level Domain`, Third, Fourth까지 쭉쭉 나타낼 수 있지만 보통 Third 정도까지 사용한다.  
> `Root Domain`은 도메인 네임에서 가장 오른쪽에 위치하지만 생략되어 있다. 사실 www.naver.com 뒤에 '.'점을 하나 더 찍으면 그 뒷부분에 해당한다. root domain은 주로 빈 문자열을 쓰므로 기왕 쓰는 김에 .도 같이 생략해서 www.naver.com까지만 쓰는 것이다.  
> `TLD`는 Root Domain의 왼쪽영역 즉 com에 해당한다. 이렇게 오른쪽에서 시작하여 왼쪽으로 가면서 Root->Top->Second->Third->Fourth로 부른다.

### Domain Name Resolution

도메인 네임(naver.com)을 가지고 서버 IP를 어떻게 찾아가는지 알아두자

1. 일단 내 컴퓨터는 기본적으로 설정된 네임 서버(Name Server)에 naver.com의 IP 주소를 알려달라는 요청을 보낸다. 여기서 네임 서버는 도메인 네임을 IP 주소로 변환하는 과정에 참여하는 서버를 말한다. 내 컴퓨터에 설정된 네임 서버는 OS마다 다르며 변경도 가능하다.
2. 네임 서버는 이제 루트 네임 서버로 이동하여 '.com'으로 끝나는 네입 서버의 주소를 요청하게 되고, 루트 네임서버는 '.com' 네임 서버의 IP를 알려준다.
3. '.com' 네임 서버에 가서는 'naver.com' 네임 서버의 IP를 요청하게 되고, 해당 네임 서버는 'naver.com'의 IP를 응답으로 받게 된다.
4. 이 과정을 통해 네임 서버가 얻은 최종 IP주소를 내 컴퓨터에 알려주게 되고, 그제서야 본격적으로 내 PC는 naver.com과 통신하게 된다.
   위 과정은 최초 사이트 접속 시 발생하는 과정이며 이후에는 OS에서 해당 도메인에 대한 IP주소를 기억해두고, 또한 OS과 가까운 네임서버에서도 캐시에 IP를 저장해두기 때문에 이후부터는 요청 과정이 짧아진다.

### URL 특정 부분 가져오기

```javascript
const url = new URL("https://www.issuebombom.store/comment?id=345&cat_id=3645");

const protocol = url.protocol;
const host = url.host;
const pathname = url.pathname;
const query = url.search;

console.log({ protocol, host, pathname, query });

/*
>> {
protocol: 'https:',
host: 'www.issuebombom.store',
pathname: '/comment',
query: '?id=345&cat_id=3645'
}
*/
```

### 포트번호 상세

아래와 같이 특정 프로토콜로 통신을 하는 서버 프로그램은 특정 포트 번호를 사용하도록 정해져 있다. 이로 인해 일반적으로 www.naver.com과 같은 https 프로토콜은 사실 도메인 네임 뒤에 `:443`을 붙어줘야 하나 nginx와 같은 웹 서버 소프트웨어에서 http와 https와 같이 포트 번호가 정해져 있는 경우에는 포트번호를 생략하더라도 백엔드에서 붙여주도록 설정이 되어 있다. 기술적으로 포트번호를 바꾼다고 해서 문제될 것은 없지만 규약이므로 지키는 것이 일반적이다.
!['port_num'](./img/port_num.png)

### URL 라우팅

도메인의 pathname에 따라 html 페이지로 연결해 주는 것을 라우팅이라고 한다.  
아래와 같이 코드를 작성하여 간단한 라우팅 구현이 가능하다.  
하지만 앞으로 if문을 덕지덕지 붙일 생각을 하니 결코 좋은 방법으로 보이지는 않는다.

```javascript
const users = ["apple", "ball", "cat", "doll", "egg"];
const http = require("http");

const server = http.createServer((request, response) => {
  if (request.url === "/") {
    response.end("<h1>Hello</h1>");
  } else if (request.url === "/users") {
    response.end(`<h1>${users}</h1>`);
  } else {
    response.end("<h1>Page Not Available</h1>");
  }
});

server.listen(3000);
```

> `Tip`  
> 자바스크립트 코드 작성 시 스타일 가이드를 지켜주는 습관을 들이는 것이 좋다.  
> https://github.com/airbnb/javascript  
> 위 주소는 에어비앤비에서 사용하는 자바스크립트 스타일 가이드이므로 참고하면 좋겠다.  
> 대표적인 스타일 가이드 예시로 아래의 목록이 있다.  
> -함수는 Arrow Function으로 쓸 것  
> -변수의 변동성이 없다면 const를 쓸 것  
> -주석은 "// 주석입니다." 와 같이 띄어쓰기를 지켜줄 것  
> -문자열은 double-quote(")가 아닌 single-quote(')를 쓸 것

### Express 맛보기

당연하겠지만 위에서 본 것처럼 라우팅을 if, else문으로 작성할 리가 없다.  
npm의 express 패키지를 설치해서 Express 프레임워크로 라우팅을 해보자

먼저 터미널에서 express 설치를 먼저 진행한다.

```bash
npm install express
```

위에서 본 코드를 express 패키지를 활용하여 수정해보자

```javascript
const express = require("express");
const users = ["apple", "ball", "cat", "doll", "egg"];
const app = express();

app.get("/", (request, response) => {
  response.end("<h1>Hello</h1>");
});

app.get("/users", (request, response) => {
  response.end(`<h1>${users}</h1>`);
});

app.get("/users/:id", (request, response) => {
  const userName = users[request.params.id - 1];
  response.end(`<h1>${userName}</h1>`);
});

app.get("*", (request, response) => {
  response.end("<h1>Page Not Available</h1>");
});

app.listen(3000);
```

> `Tip`  
> asterisk('\*')는 '모든것'을 뜻하는 의미로 여러 곳에서 쓰인다.  
> 그렇기에 asterisk로 라우팅된 get 요청도 모든페이지에 대해서 'Not Available' 응답을 했어야 할텐데 그렇지 않았던 이유는 `요청에 대한 확인을 코드의 윗줄에서 아래로 진행하기 때문이다.` 만약 '\*' 요청이 가장 윗줄에 있었다면 모든 path에 대해서 'Not Available'을 응답했을 것이다.

### nodemon 패키지 사용

Express만 사용하면 코드를 수정한 뒤 웹 상에서 확인하기 위해 서버를 껐다가 다시 켜야 하는 번거로움이 있는데 이를 해결해주는 패키지가 nodemon이다.  
노드몬은 코드가 변하면 이를 감지하여 자동으로 서버를 재실행 해주는 기능을 제공한다.  
설치는 아래와 같이 터미널에 입력한다.

```zsh
sudo npm install -g nodemon
```

위 명령어에서 -g 옵션은 global 설치를 의미한다. 글로벌 설치를 하는 이유는 nodemon 실행을 앞으로 node나 npm처럼 경로 상관 없이 항상 실행할 수 있는 환경에 놓기 위해서이다.  
정리하자면 앞으로 기존에 웹 서버를 돌리기 위해 `node main.js`를 입력하는 대신 `nodemon main.js`을 입력하여 서버를 돌릴 예정이기 때문에 글로벌 설치를 진행했다고 볼 수 있다.

### 글로벌 설치와 환경 변수

`글로벌(전역) 설치`가 의미하는 바는 간단히 말해서 현재 디렉토리의 node_modules에 설치되는 것이 아닌 MacOS 기준으로 `usr/local/lib/node_modules`경로에 설치함을 뜻한다. 여기서 `usr` 폴더는 시스템 레벨 폴더이므로 설치하기 위해 sudo 권한이 요구된다.  
또한 해당 경로 내 `/nodemon/bin/nodemon.js`파일을 실행하기 위한 symbolic link(바로가기) nodemon 파일을 `usr/local/bin` 폴더에 설치한다.  
위 두가지는 `npm config list --json`을 통해 전역 설치 경로인 prefix의 설정과 nodemon의 `package.json` 내 bin 필드의 설정을 따른다.

위에서 언급한 `usr/local/bin`은 MacOS에서 `$PATH` 환경 변수로 등록된 경로에 해당하는데, 환경 변수는 터미널에서 특정 프로그램을 실행할 때 (가령 node, npm, pip, python 명령어 입력 등) 가장 먼저 뒤지는 경로는 뜻한다. 쉽게 말해 우리가 터미널에 'node'라고 입력하면 OS는 `$PATH` 경로에 가서 'node'라는 파일이 있는지 찾는다. 그런데 `usr/local/bin`에는 대부분 심볼릭 링크 파일(바로가기)를 넣어두어 특정 프로그램의 실행이 간편하도록 하는 용도로 쓴다.

> `Tip`  
> `echo $PATH` 명령어를 입력하면 환경변수로 등록된 디렉토리의 목록을 확인할 수 있다.

결론적으로 터미널 창의 어떤 위치에서든지 nodemon을 입력하면 nodemon이라는 심볼릭 링크를 환경 변수 내에서 찾기 시작하고 이를 찾으면 심볼릭 링크가 가리키는 원본 파일의 경로를 따라가 해당 파일을 실행하는 순서이다. (추가적으로 nodemon.js 원본 파일을 열어보면 코드 상단에 node로 실행해야 한다는 코드가 있어 사실상 `node /usr/local/lib/node_modules/nodemon/bin/nodemon.js`명령을 실행하는 것과 같다.)

### 환경 변수 응용

만약 내가 만든 뉴스 스크랩 파이썬 코드를 심볼릭 링크로 만들어 환경변수에 아래와 같이 저장할 수 있다.

```zsh
ln -s /my-project/project01/newscrape.py /usr/local/bin/newscrape
```

저렇게되면 newscrape.py 파일의 심볼릭 링크인 newscrape 파일이 $PATH 환경변수에 생성되고, 앞으로 터미널에서 newscrape만 입력해도 newscrape.py파일이 실행된다.

### 패키지에 대하여

npm install을 통해 패키지를 설치하면 해당 디렉토리에 node_modules 폴더가 생성되고 해당 폴더 내 저장된다.

#### 서드파티 모듈은 패키지라고 부른다.

모듈은 직접 js 파일로 생성한 일반 모듈과 코어모듈, 그리고 서드파티 모듈로 나뉜다.  
이 중에서 서드파티 모듈은 항상 package.json 파일이 포함된 디렉토리명을 require해서 사용하는데 이러한 이유로 서드파티 모듈을 패키지라고 부른다.

#### what is package?

node_modules 폴더 내 다양한 폴더가 있을 텐데 각 폴더 내에는 package.json 파일이 존재할 것이다. 이렇게 package라는 json파일을 품고 있는 폴더는 모두 `패키지`라고 부를 수 있다. 그래서 npm으로 설치한 서드 파티 모듈은 그냥 패키지라고 부를 수 있다.

#### package.json

각 패키지에 존재하는 package.json에는 해당 패키지의 각종 정보를 얻을 수 있는데 대표적으로 아래와 같은 정보를 얻을 수 있다.

- 패키지 이름, 설명, 버전정보, 제작자, 컨트리뷰터, 라이센스, 레포지토리 주소, 홈페이지, 해시태그와 같은 키워드 등

기본적인 모듈 정보 외에도 해당 모듈이 의존하고 있는 또 다른 모듈의 목록도 확인할 수 있다. 이러한 모듈을 디펜던시라고 부른다.

#### 버전에 대한 이해

디펜던시 모듈을 보면 각 모듈의 버전 정보가 함께 표기되어 있다. 주로 X.X.X형태의 숫자로 구성되어 있는데 이를 `Semantic Version`이라고 부른다. '의미론적'이라는 의미를 지닌 Semantic은 어떠한 의미의 차이에 따라 구분짓는 곳에서 종종 등장하는 단어인 것 같다. 여기서는 숫자의 각 자리가 의미하는 바가 다르기 때문이라고 생각한다.  
왼쪽부터 major-version, minor-version, patch-version이라고 부른다.

- `patch-version`: bug fix나 기능의 효율성 향상 등을 개선했을 때 patch-version 숫자를 올린다. 이는 기존 API기능 사용자에게 있어 변화가 없는 수준을 말한다.
- `minor-version`: 신규 API 기능이 추가된 경우 버전의 숫자를 올린다. 기능이 추가되었지만 기존 기능에 변화가 없기 떄문에 기존 사용자가 API를 사용하는데 지장을 주지 않는다.
- `major-version`: 기존에 사용하던 API 기능이 삭제되거나 기존 디펜던시와의 호환성을 보장할 수 없을 수준의 업데이트일 경우 숫자를 올린다. 이는 기존 사용자에게 '기능 삭제', '바뀐명령어' 등 변경된 부분을 명확하게 명시하는 것이 중요하다.

#### Version Range Syntax

모듈에 대한 버전 명시를 범위로 표기하는 경우가 있는데 이때 사용하는 기호를 `Version Range Syntax`라고 부른다.

- "~3.8.1": Tilde로 표기된 버전은 minor-version이 바뀌지 않는 선에서 patch 버전 1 이상의 버전을 지닌 패키지는 허용함을 의미한다.
  - "~3.8" 또는 "~3"과 같이 표기될 수도 있다. 이 경우는 3.8.x부터 3.9.0미만의 범위, 그리고 3.0.0부터 4.0.0미만의 범위를 뜻한다.
- "^3.8.1": Caret 기호로 표기된 버전은 major-version이 바뀌지 않는 선에서 8.1 이상의 버전을 지닌 패키지는 허용함을 의미.
  - "^0.8.1" 또는 "^0.0.1"과 같이 표기될 수 있다. 이 경우는 0.8.1부터 0.9.0미만의 범위, 그리고 0.0.1부터 0.1.0미만의 범위를 뜻한다.
- "3.8.x": 'x'자가 들어간 자리의 버전은 무엇이든 상관없음을 의미.
- 그 밖에 부등호나 or 연산자, 하이푼 등으로 표기하기도 한다.

#### 패키지 만들기

패키지는 누구나 자유롭게 만들어서 npm에 공개할 수 있다.

현재 위치한 디렉토리를 하나의 패키지로 만드는 명령을 `npm init` 입력을 통해 실행 가능하다. 그러면 위에서 언급한 패키지 이름, 설명, 버전정보 등 항목별로 정보를 입력하도록 가이드한다.  
이후 npm사이트 가입후 직접 만든 패키지를 publish할 수 있다.

#### 패키지 공유하기

npm 사이트를 통해 내가 만든 패키지를 publish할 수도 있겠지만 다른 사람에게 일반적으로 공유할 때는 패키지를 생성한 디렉토리에서 node_modules폴더만 제외하고 전달한다. 이는 파이썬에서 requirements.txt만 전달하는 느낌과 비슷하다. 모든 모듈을 전달하기 보다는 어떤 모듈이 필요한지에 대한 디펜던지 정보가 담긴 package.json파일이 전달되면 이를 근거로 `npm install`로 모듈 설치가 가능하기 때문이다.  
여기서 package-lock.json은 현재 디렉토리에 설치된 모듈의 정확한 버전을 표기한다. 얘가 필요한 이유는 package.json파일에 설치가 필요한 패키지 정보가 표기되어 있으나 `Version Range Syntax`가 표기된, 즉 버전의 범위가 표기된 경우가 있기 때문이다. 이 경우 명확하게 몇 버전의 패키지를 설치해야 하는지 명시하지 않으므로 package-lock.json파일을 통해 해당 디렉토리가 패키지로 생성되던 시기에 설치되어있던 디펜던시의 정확한 버전을 확인할 수 있기 때문이다.  
만약 package.json파일에 모든 디펜던시에 대한 버전 정보가 명확히 표기되어 있다면 package-lock.json파일은 없어도 상관 없다.

#### 패키지의 취약성

패키지 하나 설치하는데 동봉되는 의존성 모듈이 다수 설치되는 것을 확인했을 것이다. 이 중에 악성 코드가 담겨 있을 가능성이 제로라고 할 수 없다. 그러니 반드시 공신력 있는 패키지를 설치하는 것이 안전하다.

현재 설치된 패키지의 버전을 정기적으로 확인 및 업데이트하자. 터미널에서 `npm outdated`를 입력하면 최신버전이 아닌 패키지에 대한 목록을 확인할 수 있으며 `npm update`를 통해 의존성 모듈을 모두 최신 버전으로 업데이트할 수 있다. 이 때 package.json파일에서 허용하는 범위 중 최신 버전을 채택하여 설치된다.

`npm audit` 명령을 통해 취약성 검사를 진행할 수 있다. 취약성이 발견된다면 터미널창에 리포트가 발생하고 `npm install fix` 명령을 실행하여 취약성이 해결된 최신 버전으로 업데이트한다.
