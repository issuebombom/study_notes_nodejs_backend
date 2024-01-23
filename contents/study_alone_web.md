# 자바스크립트 웹 개발 기본기

## 비동기 실행

### Fetch 함수

- fetch함수는 promise객체를 리턴 (promise 객체?)
- then 메소드로 리스폰스가 왔을 때 실행할 `콜백`을 등록
- 콜백을 등록한 후 실제 코드 실행은 'End'가 출력된 이후 실행한다.
- 이러한 방식을 `비동기 실행`이라고 부른다.

```javascript
console.log('Start!');

fetch('https://www.google.com')
  .then((response) => response.text())
  .then((result) => {
    console.log(result);
  });

console.log('End');
```

### What is callback?

일반 코드처럼 실행하면 위에서 아래로 순차실행되는 것이 아닌 함수를 비동기 함수라고 하고,  
비동기 함수는 특정 조건이 만족한 경우 실행되는 함수이며 조건을 충족하기 전까지는 실행 대기 상태에 빠진다.  
이에 해당하는 코드를 **콜백**이라고 부른다.

대표적인 비동기 함수

```javascript
setTimeout(() => {
  console.log('b');
}, 2000);
setInterval(() => {
  console.log('b');
}, 2000);
btn.addEventListener('click', function (e) {
  // 해당 이벤트 객체가 파라미터 e로 넘어옵니다.
  console.log('Hello Codeit!');
});
```

- setTimeout: 지정한 밀리세컨만큼의 딜레이를 가진 후 실행
- setInterval: 지정한 밀리세컨만큼의 딜레이를 두고서 반복 실행
- addEventListener: 클릭과 같은 이벤트 발생 시에만 함수가 실행

> **`Tip`**  
> json은 반드시 큰 따옴표로 작성해야 한다.

### What is promise?

fetch함수가 실행되면 promise라 부르는 객체가 생성된다.  
프로미스 객체는 `pending`, `fullfilled`, `rejected` 이 세가지의 작업 상태를 갖는데 이는 서버와의 통신과 연관이 있다.

- fetch함수로 벡엔드에 요청을 보내 대기중인 상태를 pending
- 서버로부터 response를 잘 받아온 상태를 fullfilled
- 어떠한 문제로 response를 받아오지 못한 상태를 rejected

```javascript
fetch('https://www.google.com')
  .then((response) => response.text())
  .then((result) => {
    console.log(result);
  });
```

- `then` 메소드는 promise 객체의 메소드에 해당한다.
- fetch함수가 실행된 이후 promise 객체가 pending에서 fullfilled 상태가 될 경우 성공 결과물이 담긴 promise 객체가 새로 생성되어 response로 전달되는데 이 역할을 then 메소드가 한다.
- then 메소드의 결과가 rejected가 될 경우 실패 결과를 갖는 promise 객체가 생성된다.
- response 객체의 text와 json 메소드는 또 다시 promise 객체를 리턴한다. 그렇기에 then 메소드를 다시 쓸 수 있는 것이다.

### Promise Chaining

fetch 함수는 비동기 함수이지만 여러 fetch 함수 끼리도 순차적인 처리가 요구될 경우가 있는데 이 때 Promise Chaining을 적용한다.

- 체이닝을 할 때는 then 메소드의 성공 결과 데이터를 return문으로 보내주면 해당 데이터를 가진 promise 객체가 생성되므로 다음에 또 then 메소드로의 연계가 가능하다.
- then 메소드의 return값으로 fetch 함수를 연동하여 promise 객체를 체이닝할 수 있다.

```javascript
// Promise Chaining 예시
fetch('https://learn.codeit.kr/api/interviews/summer')
  .then((response) => response.json())
  .then((interviewResult) => {
    const { interviewees } = interviewResult;
    const newMembers = interviewees.filter((interviewee) => interviewee.result === 'pass');
    return newMembers; // Array를 return문으로 보내 promise 객체 생성
  })
  .then((newMembers) =>
    fetch('https://learn.codeit.kr/api/members', {
      method: 'POST',
      body: JSON.stringify(newMembers),
    })
  )
  .then((response) => {
    if (response.status === 200) {
      return fetch('https://learn.codeit.kr/api/members'); // POST 실행 결과를 바로 확인하기 위해 GET 요청 연계
    } else {
      throw new Error('New members not added');
    }
  })
  .then((response) => response.json())
  .then((members) => {
    console.log(`총 직원 수: ${members.length}`);
    console.log(members);
  });
```

### fetch 함수의 결과가 rejected 상태가 되었을 때

위에서는 성공했을 경우에 대한 결과만을 받는 코드를 아래와 같이 작성했었다.

```javascript
fetch('https://google.com')
  .then((response) => response.text())
  .then((result) => {
    console.log(result);
  });
```

하지만 then 메소드는 사실 아래와 같이 fulfilled 상태와 rejected 상태일 때의 결과를 나눠서 받도록 할 수 있다.

```javascript
fetch('https://google.com')
  .then(
    (response) => response.text(),
    (error) => {
      console.log(error);
    }
  )
  .then((result) => {
    console.log(result);
  });
```

이렇게 하면 rejected 상태에 대한 정보를 error 변수로 할당받을 수 있어 콘솔에 에러 메시지가 출력된다. 과정을 살펴보자.

- `fetch ('https://google.com')`  
  fetch 함수를 통해 promise 객체가 성공(fulfilled) 결과 또는 실패(rejected) 정보를 갖고서 리턴한다.
- `.then((response) => response.text(), (error) => {console.log(error)})`  
  성공 결과는 response 변수가 받아 text() 메소드가 적용되어 텍스트가 담긴 promise 객체가 리턴된다.  
  실패 정보는 error 변수가 받아 콘솔 출력을 수행하게 되면서, 더이상 전달할 정보가 없으므로 undefined 정보를 담은 promise 객체가 리턴된다.
- `.then((result) => {console.log(result)})`  
  먼저 해당 then 메소드는 이전 then 메소드로부터 무조건 fulfilled 상태를 전달받는다는 점을 기억하자.  
  response.text()를 result로 받은 경우 텍스트 정보가 콘솔에 출력되고, undefined를 result로 받은 경우 undefined가 그대로 출력된다.

> `Tip`  
> <fetch와 then에서 기억할 점>  
> -리턴값으로 정보가 담긴 promise객체가 생성된다.  
> -promise 객체가 생성되지 않을 경우 체이닝을 할 수 없다.  
> -fetch함수의 promise 상태가 rejected가 되었는데 이를 받는 then 메소드의 콜백 함수가 없다면(error 변수가 없다면) 처음 then에서 받는 promise 객체를 다음 then 메소드에 그대로 전달한다.

### catch함수로 rejected 결과 처리하기

아래와 같이 then 메소드로 fulfilled와 rejected 결과를 나눠 받는 방법 외에 catch 메소드로 rejected 결과를 따로 처리할 수 있다.

```javascript
fetch('https://google.com')
.then((response) => response.text())
.catch((error) => { console.log(error); })
.then((result) => { console.log(result); });

// catch 메소드는 아래 then 메소드와 동일하다.
.then(undefined, (error) => { console.log(error); })
```

- 참고로 실무에서는 catch 메소드를 주로 가장 마지막에 사용하는데 이를 통해 다양한 에러 가능성을 대비해 마지막 단에 처리할 수 있기 때문이다.

### catch함수로 error 상황에 대처하기

아래와 같이 error가 발생했을 경우 대체 항목이 전달되도록 처리할 수도 있다.  
해당 경우는 SNS 사이트에서 오프라인 상태가 되더라도 사전에 저장해 둔 데이터를 갱신하도록 조치한 예시이다.

```javascript
fetch('https://friendbook.com/my/newsfeeds')
  .then((response) => response.json())
  .then((result) => {
    const feeds = result;
    // 피드 데이터 가공 과정을 거친 후...
    return processedFeeds;
  })
  .catch((error) => {
    // error 발생 시 미리 저장해둔 뉴스 데이터 보여주기
    const storedGeneralNews = getStoredGeneralNews();
    return storedGeneralNews;
  })
  .then((result) => {
    /* 화면에 표시 */
  })
  .catch((error) => {
    /* 에러 로깅 */
  });
```

### finally 메소드

fulfilled, rejected 여부와 상관없이 무조건 실행할 코드를 기입할 때 사용한다.  
아래 코드를 통해 비동기 함수의 작동 원리를 최종적으로 알아본다.

```javascript
fetch('https://www.error.www')
  .then((response) => response.text())
  .then((result) => {
    console.log(result);
  })
  .catch((error) => {
    console.log('Hello');
    throw new Error('test');
  })
  .then((result) => {
    console.log(result);
  })
  .then(undefined, (error) => {})
  .catch((error) => {
    console.log('JS');
  })
  .then((result) => {
    console.log(result);
  })
  .finally(() => {
    console.log('final');
  });
```

1. 우선 fetch에서 틀린 URL이 입력되었으므로 error가 발생한다.
1. 에러 발생으로 then 메소드는 넘어가고 catch에서 'Hello'를 출력한 뒤 에러가 발생한다.
1. 5번째 줄 then은 catch 메소드와 기능이 같다. 그래서 promise 객체가 반환되지만 리턴값이 없으므로 undefined값이 전달된다. (fulfilled 상태)
1. then 메소드로 넘어가 'undefined'를 콘솔에 출력한다.
1. finally 메소드는 무조건 실행되므로 'final'이 출력된다.
   최종적으로 콘솔에 Hello, undefined, final이 출력된다.

### Promise 객체 직접 생성하기

Promise 객체를 직접 생성할 일은 거의 없지만 원리를 알아두도록 한다.

```javascript
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success');
  }, 2000);
});
```

resolve와 reject함수는 `executor`함수라고 한다. 즉 두 함수가 트리거 역할을 한다. 등장하는 순간 fulfilled 또는 rejected 상태가 되는 것이다.  
위 예시를 보면 resolve함수가 2초 뒤에 실행이 되는 것을 볼 수 있다. executor 함수는 트리거이므로 2초 뒤에 fulfilled 상태가 될 것이다. 그리고 resolve함수에 담긴 파라미터값이 fulfilled 상태가 되어 생성된 새로운 promise 객체의 데이터가 된다.  
reject는 rejected 상태로 만드는 트리거로 원리는 resolve와 동일하다.

- 정리하자면 executor함수는 등장하는 순간 promise의 상태가 결정되고, executor함수의 파라미터값이 새 promise 객체의 데이터가 된다.

### 추가 Promise 객체 다루기

#### all 메소드

```javascript
// 1번 직원 정보
const p1 = fetch('https://learn.codeit.kr/api/members/1').then((res) => res.json());
// 2번 직원 정보
const p2 = fetch('https://learn.codeit.kr/api/members/2').then((res) => res.json());
// 3번 직원 정보
const p3 = fetch('https://learn.codeit.kr/api/members/3').then((res) => res.json());

Promise.all([p1, p2, p3]).then((results) => {
  console.log(results); // Array : [1번 직원 정보, 2번 직원 정보, 3번 직원 정보]
});
```

all 메소드는 여러 promise 객체를 array로 담아두고, 모든 promise가 fulfilled가 되면 그제서야 다음 then 메소드로 넘겨준다. 이 때 array 안에 있던 promise 객체의 데이터가 array에 담겨져 전달된다.  
만일 하나라도 rejected가 된다면 all 메소드는 rejected 상태가 된다.

#### race 메소드

이름처럼 여러 promise 중 가장 먼저 pending 상태에서 fulfilled 또는 reject 상태가 된 promise 객체를 리턴한다. 상태는 상관 없다. 단지 가장 먼저 pending에서 벗어난 promise가 채택된다.

#### settled 메소드

fulfilled와 rejected 상태를 묶어서 settled 상태라고 부른다.  
그래서 모든 promise가 pending에서 벗어나면 settled는 fulfilled 상태의 promise 객체를 리턴하고, 값은 아래와 같이 표현된다.

```javascript
// 결과
[
   {status: "fulfilled", value: 1},
   {status: "fulfilled", value: 2},
   {status: "fulfilled", value: 3},
   {status: "rejected",  reason: Error: an error}
]
```

각 promise들의 상태와 결과를 딕셔너리로 전달한다.

### async / await

fetch와 then 메소드를 활용한 기본 비동기 함수의 활용을 다소 간단하게 표현하기 위한 기능

```javascript
async function fetchAndPrint() {
  console.log(2);
  const response = await fetch('https://jsonplaceholder.typicode.com/users');
  console.log(7);
  const result = await response.text();
  console.log(result);
}

console.log(1);
fetchAndPrint();
console.log(3);
console.log(4);
console.log(5);
console.log(6);
```

fetchAndPrint 함수 앞에 async라는 함수를 붙이면 해당 함수는 비동기 함수가 된다.  
비동기 함수가 되면 promise 객체를 생성하는 코드 앞에 await를 붙인다.

> `중요`  
> await 함수가 붙으면 그 뒤 함수는 일단 등록한 뒤 비동기 함수 밖으로 나가서 나머지 코드를 실행한다.  
> 즉 await fetch()에서 fetch 함수를 접수받되 실행되지 않고 -> 바깥에 console.log(3) 이후 코드를 실행합니다.  
> -> 외부 코드가 모두 실행된 이후 -> fetch함수를 실행하여 pending 상태에 들어가 fulfilled 또는 rejected를 기다립니다.

> `결론`  
> await가 붙으면 비동기 함수 바깥으로 무조건 나가서 나머지 코드를 실행합니다.  
> 두 번째 await를 만나도 동일하게 작동하나 이전에 바깥코드는 이미 실행되었으므로 또 실행하지는 않는다.  
> await를 async 함수 내에서만 사용 가능하다.

## 즉시실행 함수

함수를 정의함과 동시에 바로 파라미터에 값을 전달하여 실행하는 함수를 의미한다.  
Function을 선언한 경우 함수 전체를 괄호로 감싸고, 전달할 파라미터가 전체 괄호 안에 튜플 형태로 전달된다.  
Arrow Function의 경우 함수와 파라미터가 각각 괄호로 감싸져있다.

```javascript
(function print(sentence) {
  console.log(sentence);
  return sentence;
})('I love JavaScript!');

(function (a, b) {
  return a + b;
})(1, 2);

((a, b) => {
  return a + b;
})(1, 2);

((a, b) => a + b)(1, 2);
```

### 즉시실행 함수의 async 응용

먼저 아래 함수를 보면 urls 배열에서 url을 하나씩 꺼내 request를 한다.  
문제는 첫 url의 fetch 응답을 받은 뒤에서야 비로소 다음 url에 대한 요청을 보낸다는 점이다. 즉 순차적으로 확인한다.

```javascript
async function getResponses(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    console.log(await response.text());
  }
}
```

만약 각 url의 응답이 순차적일 필요가 없다면 아래와 같이 작성하는게 효율적이다.  
리퀘스트 과정을 async 함수가 적용된 즉시실행함수로 정의하여 await fetch(url)에서 요청을 보낸 직후 pending 상태에서는 async 함수를 벗어나 외부 코드를 실행하도록 되어있다.  
그렇기 때문에 for문의 다음 iteration이 실행될 수 있는 것이다. 이렇게 하면 각 url의 응답을 순차적으로 받을 수는 없지만 그럴 필요가 없다면 처리 시간을 단축할 수 있다.

```javascript
async function getResponses(urls) {
  for (const url of urls) {
    (async () => {
      const response = await fetch(url);
      console.log(await response.text());
    })();
  }
}
```

## JSON 파일의 response

```javascript
fetch('https://www.google.com')
  .then((response) => response.text())
  .then((result) => {
    const users = JSON.parse(resullt);
    users.forEach((user) => {
      console.log(user.name);
    });
  });

// 아래와 같이도 적용 가능
fetch('https://www.google.com')
  .then((response) => response.json()) // response가 json이 아니면 에러 발생
  .then((result) => {
    result.forEach((user) => {
      console.log(user.name);
    });
  });
```

> `Tip`  
> 기억하자! function((a) => { console.log(); })는 Arrow function에서 중괄호를 사용하면 return값을 따로 지정해줘야 한다.  
> 하지만 function((a) => console.log();)로 중괄호를 안쓰면 자동으로 return으로 지정된다는 것을 잊지 말자!

### Request 메소드

- 웹 브라우저가 서버로 보내는 리퀘스트의 종류에 크게 다음과 같은 4가지 종류가 있다.

> 기존 데이터를 조회하는 리퀘스트 - GET  
> 새 데이터를 추가하는 리퀘스트 - POST  
> 기존 데이터를 수정하는 리퀘스트 - PUT  
> 기존 데이터를 삭제하는 리퀘스트 - DELETE

#### POST Request

- 새로운 직원의 정보를 추가한다.
- fetch의 default method가 GET이므로 GET 이외의 메소드는 명시가 필요하다.
- 입력될 정보를 body로 전달한다.

```javascript
const newMember = {
  name: 'Jerry',
  email: 'abc@gmail.com',
  department: 'engineering',
};
fetch('https://learn.codeit.kr/api/members', {
  method: 'POST',
  body: JSON.stringify(newMember),
})
  .then((response) => response.text())
  .then((result) => {
    console.log(result);
  });
```

> **`Tip`**  
>  -JSON.stringify는 json 파일을 자바스크립트 객체로 변환하는 역할을 한다.  
>  -위 예시와 달리 여러 사람의 데이터을 담은 array를 stringify에 담아 전달해도 한 번에 POST 요청이 가능하다.

#### PUT Request

- 기존 직원의 정보를 덮어쓰는 방식으로 수정한다. 그래서 수정되지 않을 정보도 함께 입력되어야 한다.
- 수정될 부분에 대한 정보를 body에 전달한다.

```javascript
const member = {
  name: 'Alice',
  email: 'alice@codeitmall.kr',
  department: 'marketing', // engineering에서 marketing으로 변경
};
fetch('https://learn.codeit.kr/api/members/2', {
  method: 'PUT',
  body: JSON.stringify(member),
})
  .then((res) => res.text())
  .then((result) => {
    console.log(result);
  });
```

#### DELETE Request

- 기존 객체를 삭제한다.

```javascript
fetch('https://learn.codeit.kr/api/members/2', {
  method: 'DELETE',
})
  .then((res) => res.text())
  .then((result) => {
    console.log(result);
  });
```

> **`Tip`**  
> 자바스크립트 객체를 string 타입의 JSON 데이터로 변환하는 것은 영어로 Serialization이라고 합니다.

> 그 외 메소드 참조:  
> https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods

### jsonString과 javascript object

- request 과정에서의 serialization과 deserialization은 빈번히 발생하며 반드시 수행되어야 한다.
- jsonString은 `'{"x": 1, "y": 2}'` 형태를 지니며 이 상태에서는 key값에 접근이 불가하다. 전체가 string이기 때문이다. 그러므로 JSON.parse()를 통해 js object로 변환해서 사용하는게 편리하다.
- object는 `{ x: 1, y: 2 }` 데이터 이외에 내장 프로퍼티들이 존재하는데 이 정보들을 서버에 보낼 필요가 없으며 보낼 수도 없으므로 JSON.stringify() 처리를 해야한다.

### status_code 상태코드에 대한 정리

#### `100번대`

서버가 클라이언트에게 정보성 응답(Informational response)을 줄 때 사용되는 상태 코드들입니다.

> `100 Continue` : 클라이언트가 서버에게 계속 리퀘스트를 보내도 괜찮은지 물어봤을 때, 계속 리퀘스트를 보내도 괜찮다고 알려주는 상태 코드입니다. 예를 들어, 클라이언트가 용량이 좀 큰 파일을 리퀘스트의 바디에 담아 업로드하려고 할 때 서버에게 미리 괜찮은지를 물어보는 경우가 있다고 할 때, 서버가 이 100번 상태 코드의 리스폰스를 주면 그제서야 본격적인 파일 업로드를 시작합니다.  
> `101 Switching Protocols` : 클라이언트가 프로토콜을 바꾸자는 리퀘스트를 보냈을 때, 서버가 '그래요, 그 프로토콜로 전환하겠습니다'라는 뜻을 나타낼 때 쓰이는 상태 코드입니다.

#### `200번대`

클라이언트의 리퀘스트가 성공 처리되었음을 의미하는 상태 코드들입니다.

> `200 OK` : 리퀘스트가 성공적으로 처리되었음을 포괄적으로 의미하는 상태 코드입니다. 이때 성공의 의미는 리퀘스트에 있던 메소드의 종류에 따라 다르겠죠? GET 리퀘스트의 경우 리소스가 잘 조회되었다는 뜻이고, POST 리퀘스트의 경우 새 리소스가 잘 생성되었다, PUT 리퀘스트의 경우 기존 리소스가 잘 수정되었다, DELETE 리퀘스트의 경우 기존 리소스가 잘 삭제되었다는 뜻입니다.  
> `201 Created` : 리퀘스트의 내용대로 리소스가 잘 생성되었다는 뜻입니다. POST 리퀘스트가 성공한 경우에 200번 대신 201번이 올 수도 있습니다.  
> `202 Accepted` : 리퀘스트의 내용이 일단은 잘 접수되었다는 뜻입니다. 즉, 당장 리퀘스트의 내용이 처리된 것은 아니지만 언젠가 처리할 것이라는 뜻인데요. 리퀘스트를 어느 정도 모아서 한번에 실행하는 서버인 경우 등에 이런 응답을 줄 수도 있습니다.

#### `300번대`

클라이언트의 리퀘스트가 아직 처리되지 않았고, 리퀘스트 처리를 원하면 클라이언트 측의 추가적인 작업이 필요함을 의미하는 상태 코드들입니다.

> `301 Moved Permanently` : 리소스의 위치가 바뀌었음을 나타냅니다. 보통 이런 상태 코드가 있는 리스폰스의 헤드에는 Location이라는 헤더도 일반적으로 함께 포함되어 있습니다. 그리고 그 헤더의 값으로 리소스에 접근할 수 있는 새 URL이 담겨있는데요. 대부분의 브라우저는 만약 GET 리퀘스트를 보냈는데 이런 상태 코드가 담긴 리스폰스를 받게 되면, 헤드에 포함된 Location 헤더의 값을 읽고, 자동으로 그 새 URL에 다시 리퀘스트를 보내는 동작(리다이렉션, redirection)을 수행합니다.  
> `302 Found` : 리소스의 위치가 일시적으로 바뀌었음을 나타냅니다. 이 말은 지금 당장은 아니지만 나중에는 현재 요청한 URL이 정상적으로 인식될 것이라는 뜻입니다. 이 상태 코드의 경우에도 보통 그 리스폰스의 헤드에 Location 헤더가 있고, 여기에 해당 리소스의 임시 URL 값이 있습니다. 이 경우에도 대부분의 브라우저들은 임시 URL로 리다이렉션합니다.  
> `304 Not Modified` : 브라우저들은 보통 한번 리스폰스로 받았던 이미지 같은 리소스들을 그대로 내부에 저장하고 있습니다. 그리고 서버는 해당 리소스가 바뀌지 않았다면, 리스폰스에 그 리소스를 보내지 않고 304번 상태 코드만 헤드에 담아서 보냄으로써 '네트워크 비용'을 절약하고 브라우저가 저장된 리소스를 재활용하도록 하는데요. 사실 이 상태 코드는 웹에서 '캐시(cache)'라는 주제에 대해서 공부해야 정확하게 이해할 수 있습니다. 당장 배울 내용은 아니니까 넘어갈게요. 혹시 관심이 있는 분들은 이 링크를 참조하세요.

#### `400번대`

리퀘스트를 보내는 클라이언트 쪽에 문제가 있음을 의미하는 상태 코드들입니다.

> `400 Bad Request` : 말그대로 리퀘스트에 문제가 있음을 나타냅니다. 리퀘스트 내부 내용의 문법에 오류가 존재하는 등의 이유로 인해 발생합니다.  
> `401 Unauthorized` : 아직 신원이 확인되지 않은(unauthenticated) 사용자로부터 온 리퀘스트를 처리할 수 없다는 뜻입니다.  
> `403 Forbidden` : 사용자의 신원은 확인되었지만 해당 리소스에 대한 접근 권한이 없는 사용자라서 리퀘스트를 처리할 수 없다는 뜻입니다.  
> `404 Not Found` : 해당 URL이 나타내는 리소스를 찾을 수 없다는 뜻입니다. 보통 이런 상태 코드가 담긴 리스폰스는 그 바디에 관련 웹 페이지를 이루는 코드를 포함하고 있는 경우가 많습니다. 예를 들어 존재하지 않는 URL에 접속하려고 하면 이런 페이지가 보이는 것을 알 수 있습니다.  
> `405 Method Not Allowed` : 해당 리소스에 대해서 요구한 처리는 허용되지 않는다는 뜻입니다. 만약 어떤 서버의 이미지 파일을 누구나 조회할 수는 있지만 아무나 삭제할 수는 없다고 해봅시다. GET 리퀘스트는 허용되지만, DELETE 메소드는 허용되지 않는 상황인 건데요. 그런데 만약 그 이미지에 대한 DELETE 리퀘스트를 보낸다면 이런 상태 코드를 보게될 수도 있습니다.  
> `413 Payload Too Large` : 현재 리퀘스트의 바디에 들어있는 데이터의 용량이 지나치게 커서 서버가 거부한다는 뜻입니다.
> 429 Too Many Requests : 일정 시간 동안 클라이언트가 지나치게 많은 리퀘스트를 보냈다는 뜻입니다. 서버는 수많은 클라이언트들의 리퀘스트를 정상적으로 처리해야 하기 때문에 특정 클라이언트에게만 특혜를 줄 수는 없습니다. 따라서 지나치게 리퀘스트를 많이 보내는 클라이언트에게는 이런 상태 코드를 담은 리스폰스를 보낼 수도 있습니다.

#### `500번대`

서버 쪽의 문제로 인해 리퀘스트를 정상적으로 처리할 수 없음을 의미하는 상태 코드들입니다.

> `500 Internal Server Error` : 현재 알 수 없는 서버 내의 에러로 인해 리퀘스트를 처리할 수 없다는 뜻입니다.  
> `503 Service Unavailable` : 현재 서버 점검 중이거나, 트래픽 폭주 등으로 인해 서비스를 제공할 수 없다는 뜻입니다.

### Content-Type Header

- Request의 body에 담을 데이터가 어떤 형태인지 지정해줄 수 있다.

> `예시  ('주 타입(main type)/서브 타입(sub type)')`  
> 'text/plain' : 일반 텍스트  
> 'image/png' : png 이미지  
> 'application/json' : JSON 데이터  
> application/octet-stream : 확인되지 않은 바이너리 파일  
> ...

### application/x-www-form-urlencoded

- 과거 form 태그를 통해 js없이 html만으로 리퀘스트를 보낼 때 사용한 데이터 타입을 말한다.
- URL의 쿼리 부분에 입력되어 리퀘스트가 보내진다.
- URL에서 특정 특수문자들 그리고 영어와 숫자를 제외한 다른 나라의 문자들을 `Percent encoding`이 적용된 `URL encoding` 방식이 적용된다.

```javascript
// x-www-form-urlencoded 생성 예시
const urlencoded = new URLSearchParams();
urlencoded.append('email', 'tommy@codeit.kr');
urlencoded.append('password', 'codeit123!');
urlencoded.append('nickname', 'Nice Guy!');

>> email=tommy%40codeit.kr&password=codeit123%21&nickname=Nice+Guy%21
```

### multipart/form-data

- 여러 데이터를 담아서 리퀘스트를 보낼 때 쓰는 데이터 타입이다.

```javascript
// querySelector로 데이터를 가져온 뒤
const formData = new FormData();
formData.append('email', email.value);
formData.append('password', password.value);
formData.append('nickname', nickname.value);
formData.append('profile', image.files[0], 'me.png');

fetch('https://learn.codeit.kr/api/members', {
  method: 'POST',
  body: formData,
})
  .then((response) => response.text())
  .then((result) => {
    console.log(result);
  });
```

- formData는 boundary 값을 갖는데 이 값이 데이터 간 구분선 역할을 한다.

### Ajax 통신

- 웹 브라우저가 현재 페이지를 그대로 유지한 채로 서버에 리퀘스트를 보내고 리스폰스를 받아서, 새로운 페이지를 로드하지 않고도 변화를 줄 수 있게 해주는 기술

> `Tip`  
> 이때까지 배운 fetch 함수가 사실은 Ajax 통신을 하는 함수였다는 사실!

> `참조1`  
> '백엔드 개발자'의 경우에는 서비스의 사용자 수가 늘어나서 리퀘스트의 수가 늘어날수록 HTTP 아래에 있는 저수준 프로토콜(TCP, IP, Ethernet)에 대해서도 어느 정도 알고 있어야 각종 성능 문제 등을 해결할 수 있습니다. 그래서 혹시 '백엔드 개발자'를 꿈꾸는 분들이라면 당장은 공부하지 않더라도 다른 프로토콜들에 대해서도 일단은 미리 관심을 가져두는 게 좋습니다.

> `참조2`  
> 실제 개발을 할 때는 사용하려는 문법이 어느 웹 브라우저에서 지원을 하고 안 하는지를 아래 사이트와 같은 곳에서 체크해둘 필요가 있다.  
> https://kangax.github.io/compat-table/es6/

## Web API와 REST API 노트

### Web API

우리가 어떤 리퀘스트를 보냈을 때, 무슨 리스폰스를 받는지는 모두 그 서비스를 만드는 개발자들이 정하는 부분입니다. 잠깐 실제 개발 현장에서 일어나는 이야기를 해볼게요. 개발자에는 크게 두 가지 종류가 있습니다. 첫 번째는 사용자가 직접 서비스 화면을 보는 웹 페이지나 앱 등을 만드는 프론트엔드(Front-end) 개발자, 두 번째는 웹 브라우저나 앱이 보내는 리퀘스트를 받아서 적절한 처리를 한 후 리스폰스를 주는 서버의 프로그램을 만드는 백엔드(Back-end) 개발자, 이 두 가지인데요.

하나의 서비스를 만들 때는 프론트엔드 개발자들과 백엔드 개발자들이 모여 '프론트엔드에서 이 URL로 이렇게 생긴 리퀘스트를 보내면, 백엔드에서 이런 처리를 하고 이런 리스폰스를 보내주는 것으로 정합시다'와 같은 논의를 하고, 이런 내용들을 정리한 후에 개발을 시작합니다.

이것을 'Web API 설계'라고 하는데요. API란 Application Programming Interface의 약자로, 원래는 '개발할 때 사용할 수 있도록 특정 라이브러리나 플랫폼 등이 제공하는 데이터나 함수 등'을 의미합니다. 웹 개발에서는 어느 URL로 어떤 리퀘스트를 보냈을 때, 무슨 처리가 수행되고 어떤 리스폰스가 오는지에 관해 미리 정해진 규격을 Web API라고도 하는데요.

Web API를 설계한다는 것은 서비스에서 사용될 모든 URL들을 나열하고, 각각의 URL에 관한 예상 리퀘스트와 리스폰스의 내용을 정리한다는 뜻입니다. 예를 들어, 이전 영상에서 사용한 학습용 URL(https://learn.codeit.kr/api/members)에서 직원 정보 추가 기능을 설계한다면 다음과 같이 할 수 있는 겁니다.

```
...

3. 직원 정보 추가

https://learn.codeit.kr/api/members

(1) Request
- Head
Method : POST
...

- Body
{
  "name": "Jerry",
  "email: "jerry@codeitshopping.kr",
  "department": "engineering",
}
...

(2) Response
Success인 경우 :
- Head
...
- Body
{
  "id": "[부여된 고유 식별자 값]",
  "name": "Jerry",
  "email": "jerry@codeshopping.kr"
  "department": "engineering",
}
Fail인 경우 :
...
```

이렇게 해당 서비스에서 제공되는 각 URL에, 어떤 리퀘스트를 보내면, 서버는 어떤 리스폰스를 보내야 하는지를 일일이 설계하는 것이 Web API 설계인 겁니다. 물론 실무에서는 지금 보이는 예시보다 훨씬 체계적이고 단정한 방식으로, 상용 툴 등을 사용해서 정리하지만 일단은 이해 차원에서 보여드렸습니다. 이런 식으로 Web API가 설계되고 나면, 그때 프론트엔드/백엔드 개발자들이 해당 설계에 맞게 각자 코드를 작성하기 시작하는 겁니다. 물론 설계와 개발이 동시에 진행되기도 하고, 설계 내용이 중간에 수정되기도 합니다.

오늘날 많은 회사 내의 개발팀은 이런 식으로 Web API를 설계하고 웹 서비스를 만듭니다. 그런데 문제가 하나 있습니다. 그건 바로 Web API는 어떻게 설계해도 동작하는 데는 아무런 지장이 없다는 문제입니다.

이전 영상들에서 저는 직원 정보를 추가하기 위해

(1) 'https://learn.codeit.kr/api/members' URL로  
(2) 리퀘스트의 헤드에 POST 메소드를 설정하고,
(3) 리퀘스트의 바디에 새 직원 정보를 넣어서 보내면 된다

는 내용의 설계를 했습니다.

그런데 어떤 회사는 같은 기능을 이런 식으로 설계할 수도 있습니다.

(1) 'https://learn.codeit.kr/api/members' URL로
(2) 리퀘스트의 헤더에 GET 메소드를 설정하고,
(3) 리퀘스트의 바디에 새 직원 정보를 넣어서 보내면 된다

어느 방식으로 설계해도 서비스가 동작하는 데는 아무런 문제가 없습니다. 하지만 기능적으로 아무런 문제가 없다고 해도 Web API를 아무렇게나 설계해도 되는 것은 아닙니다. 사실 Web API가 잘 설계되었는지에 관한 기준으로는 보통 REST API라는 기준이 사용되고 있는데요. 많은 개발자들이 Web API를 개발할 때 이 REST API를 준수하기 위해 노력하고 있습니다. 이게 뭔지 한번 살펴봅시다.

### REST API

REST API는 오늘날 많은 웹 개발자들이 Web API 설계를 할 때, 준수하기 위해 노력하는 일종의 가이드라인입니다. REST API를 이해하기 위해서는 일단 REST architecture가 무엇인지부터 알아야 하는데요. 일단 REST architecture에 대해 설명하겠습니다.

REST architecture란 미국의 컴퓨터 과학자인 Roy Fielding이 본인의 박사 논문 'Architectural Styles and the Design of Network-based Software Architectures'에서 제시한 개념인데요. 그는 웹이 갖추어야 할 이상적인 아키텍처(구조)로 REST architecture라는 개념을 제시했습니다. 여기서 REST는 Representational State Transfer(표현적인 상태 이전)의 줄임말로, 해석하면 '표현적인, 상태 이전'이라는 뜻입니다. 이게 무슨 말일까요? 이 용어는 Roy Fielding이 고안한 용어인데요. 지금 여러분이 웹 서핑을 할 때를 생각해보세요. 만약 웹을 하나의 거대한 컴퓨터 프로그램이라고 생각한다면, 각각의 웹 페이지는 그 프로그램의 내부 상태를 나타낸다고 할 수 있습니다. 그렇다면 우리가 웹 페이지들을 계속 옮겨 다니면서 보게 되는 내용은, 웹이라는 프로그램의 매번 새로운 상태를 나타내는 표현이라고 할 수 있는데요. 그래서 이것을 '표현적인, 상태 이전'이라고 하는 겁니다. 조금 추상적인 느낌이지만 이해는 되시죠?

그럼 REST architecture가 되기 위한 조건에는 어떤 것들이 있을까요? 다음과 같은 6가지 기준을 충족하면 REST architecture로 인정됩니다.

Client-Server  
Stateless  
Cache  
Uniform Interface  
Layered System  
Code on Demand

`Client-Server`  
Client-Server 구조를 통해 양측의 관심사를 분리해야 합니다. 현재 토픽에서는 웹 브라우저가 실행되고 있는 컴퓨터가 Client, 서비스를 제공하는 컴퓨터가 Server에 해당하는데요. 이렇게 분리를 해놓으면 Client 측은 사용자에게 어떻게 하면 더 좋은 화면을 보여줄지, 다양한 기기에 어떻게 적절하게 대처해야할지 등의 문제에 집중할 수 있고, Server 측은 서비스에 적합한 구조, 확장 가능한 구조를 어떻게 구축할 것인지 등의 문제에 집중할 수 있습니다. 이렇게 각자가 서로를 신경쓰지 않고 독립적으로 운영될 수 있는 겁니다.

`Stateless`  
Client가 보낸 각 리퀘스트에 관해서 Server는 그 어떤 맥락(context)도 저장하지 않습니다. 즉, 매 리퀘스트는 각각 독립적인 것으로 취급된다는 뜻입니다. 이 때문에 리퀘스트에는 항상 필요한 모든 정보가 담겨야합니다.

`Cache`  
Cache를 활용해서 네트워크 비용을 절감해야 합니다. Server는 리스폰스에, Client가 리스폰스를 재활용해도 되는지 여부(Cacheable)를 담아서 보내야합니다.

`Uniform Interface`  
Client가 Server와 통신하는 인터페이스는 다음과 같은 하위 조건 4가지를 준수해야 합니다. 이 조건이 REST API와 연관이 깊은 조건입니다. 어떤 4가지 하위 조건들이 있는지 살펴봅시다.

> `identification of resources`  
> 리소스(resource)는 웹상에 존재하는 데이터를 나타내는 용어인데요. 저도 이번 노트에서는 리소스라는 용어를 사용하겠습니다. 이것은 리소스(resource)를 URI(Uniform Resource Identifier)로 식별할 수 있어야 한다는 조건입니다. URI는 URL의 상위 개념으로 일단 지금은 URL이라고 생각하셔도 큰 무리는 없습니다.

> `manipulation of resources through representations`  
> Client와 Server는 둘 다 리소스를 직접적으로 다루는 게 아니라 리소스의 '표현(representations)'을 다뤄야 합니다. 예를 들어, Server에 '오늘 날씨'(/today/weather)라는 리소스를 요청했을 때, 어떤 Client는 HTML 파일을 받을 수도 있고, 어떤 Client는 이미지 파일인 PNG 파일을 받도록 구현할 수도 있는데요. 이때 HTML 파일과 PNG 파일 같은 것들이 바로 리소스의 '표현'입니다. 즉, 동일한 리소스라도 여러 개의 표현이 있을 수 있다는 뜻입니다. 사실, 리소스는 웹에 존재하는 특정 데이터를 나타내는 추상적인 개념입니다. 실제로 우리가 다루게 되는 것은 리소스의 표현들뿐인데요. 이렇게 '리소스'와 '리소스의 표현'이라는 개념 2개를 서로 엄격하게 구분하는 것이 REST architecture의 특징입니다.

> `self-descriptive messages`  
> self-descriptive는 '자기설명적인'이라는 뜻인데요. 위에서 살펴본 2. Stateless 조건 때문에 Client는 매 리퀘스트마다 필요한 모든 정보를 담아서 전송해야 합니다. 그리고 이때 Client의 리퀘스트와 Server의 리스폰스 모두 그 자체에 있는 정보만으로 모든 것을 해석할 수 있어야 한다는 뜻입니다.

> `hypermedia as the engine of application state`  
> REST architecture는 웹이 갖추어야 할 이상적인 아키텍처라고 했죠? 이때 '웹'을 좀더 어려운 말로 풀어써 보자면 '분산 하이퍼미디어 시스템'(Distributed Hypermedia System)이라고도 할 수 있는데요. 여기서 하이퍼미디어(Hypermedia)는 하이퍼텍스트(Hypertext)처럼 서로 연결된 '문서'에 국한된 것이 아니라 이미지, 소리, 영상 등까지도 모두 포괄하는 더 넓은 개념의 단어입니다. 즉, 웹은 수많은 컴퓨터에 하이퍼미디어들이 분산되어 있는 형태이기 때문에, '분산 하이퍼미디어 시스템'에 해당합니다. 이 조건은 웹을 하나의 프로그램으로 간주했을 때, Server의 리스폰스에는 현재 상태에서 다른 상태로 이전할 수 있는 링크를 포함하고 있어야 한다는 조건입니다. 즉, 리스폰스에는 리소스의 표현, 각종 메타 정보들뿐만 아니라 계속 새로운 상태로 넘어갈 수 있도록 해주는 링크들도 포함되어 있어야 한다는 거죠.

`Layered System`  
Client와 Server 사이에는 프록시(proxy), 게이트웨이(gateway)와 같은 중간 매개 요소를 두고, 보안, 로드 밸런싱 등을 수행할 수 있어야 합니다. 이를 통해 Client와 Server 사이에는 계층형 층(hierarchical layers)들이 형성됩니다.

`Code-On-Demand`  
Client는 받아서 바로 실행할 수 있는 applet이나 script 파일을 Server로부터 받을 수 있어야 합니다. 이 조건은 Optional한 조건으로 REST architecture가 되기 위해 이 조건이 반드시 만족될 필요는 없습니다.

---

REST API는 바로 이런 REST architecture에 부합하는 API를 의미한다는 사실입니다. 참고로 이런 REST API를 사용하는 웹 서비스를 RESTful 서비스라고 합니다. 그렇다면 구체적으로 어떤 식으로 Web API를 설계해야 REST API가 될 수 있는 걸까요? 사실 Roy Fielding의 논문에는 이것에 관한 구체적이고 실천적인 내용들은 제시되어 있지 않습니다. 하지만 많은 개발자들의 경험과 논의를 통해 형성된 사실상의(de facto) 규칙들이 존재하는데요.

우리는 그중에서도 조건 4. Uniform Interface의 하위 조건인 (4-1) identificaton of resources 에 관해서 특히 개발자들이 강조하는 규칙, 2가지만 배워보겠습니다.

[1] URL은 리소스를 나타내기 위해서만 사용하고, 리소스에 대한 처리는 메소드로 표현해야 합니다.
이 규칙은 조금 다르게 설명하자면, URL에서 리소스에 대한 처리를 드러내면 안 된다는 규칙인데요. 이게 무슨 말인지 1. Web API 부분에서 마지막에 언급했던 예시를 통해 이해해보겠습니다.

예를 들어, 새 직원 정보를 추가하기 위해서

(1) 'https://learn.codeit.kr/api/members' URL로  
(2) 리퀘스트의 헤드에 POST 메소드를 설정하고,  
(3) 리퀘스트의 바디에 새 직원 정보를 넣어서 보내면 된다

고 하는 경우는, URL은 리소스만 나타내고, 리소스에 대한 처리(리소스 추가)는 메소드 값인 POST로 나타냈기 때문에 이 규칙을 준수한 것입니다.

하지만

(1) 'https://learn.codeit.kr/api/members/add' URL로  
(2) 리퀘스트의 헤더에 GET 메소드를 설정하고,  
(3) 리퀘스트의 바디에 새 직원 정보를 넣어서 보내면 된다

고 하는 이 경우는 URL에서 리소스뿐만 아니라, 해당 리소스에 대한 처리(add, 추가하다)까지도 나타내고 있습니다. 그리고 정작 메소드 값으로는 리소스 추가가 아닌 리소스 조회를 의미하는 GET을 설정했기 때문에 이 규칙을 어긴 것입니다.

URL은 리소스를 나타내는 용도로만 사용하고, 리소스에 대한 처리는 메소드로 표현해야 한다는 사실, 꼭 기억하세요!

[2] 도큐먼트는 단수 명사로, 컬렉션은 복수 명사로 표시합니다.
또 다른 규칙 하나를 살펴볼까요? 이 규칙은 URL로 리소스를 나타내는 방식에 관한 규칙인데요. URL에서는

https://www.soccer.com/europe/teams/manchester-united/players/pogba  
이런 식으로 path 부분에서 특정 리소스(pogba, 축구 선수 포그바의 정보)를 나타낼 때 슬래시(/)를 사용해서 계층적인 형태로 나타냅니다. 지금 위 URL의 path 부분을 보면 '유럽의', '축구팀들 중에서', '맨체스터 유나이티드 팀의', '선수들 중에서', '포그바'라는 선수의 정보를 의미하는 리소스라는 걸 한눈에 알 수 있는데요. 이렇게 계층적 관계를 잘 나타내면, URL만으로 무슨 리소스를 의미하는지를 누구나 쉽게 이해할 수 있습니다. Web API를 설계할 때는 이렇게 가독성 좋고, 이해하기 쉬운 URL을 설계해야 하는데요. 그런데 이때 지켜야 할 규칙이 있습니다.

사실 리소스는 그 특징에 따라 여러 종류로 나눠볼 수 있습니다. 이 중에서 우리는 '컬렉션(collection)'과 '도큐먼트(document)'를 배울 건데요. 보통 우리가 하나의 객체로 표현할 수 있는 리소스를 '도큐먼트'라고 합니다. 그리고 여러 개의 '도큐먼트'를 담을 수 있는 리소스를 '컬렉션'이라고 하는데요. 쉽게 비유하자면, 도큐먼트는 하나의 '파일', 컬렉션은 여러 '파일'들을 담을 수 있는 하나의 '디렉토리'에 해당하는 개념입니다.

그리고 이에 관한 규칙은 바로, URL에서 '도큐먼트'를 나타낼 때는 단수형 명사를, '컬렉션'을 나타낼 때는 복수형 명사를 사용해야 한다는 규칙입니다.

지금 위 URL에서 europe, manchester-united, pogba가 '도큐먼트'에 해당하고, teams, players가 '컬렉션'에 해당합니다. 도큐먼트는 단수 명사로, 컬렉션은 복수 명사로 표현한 것이 잘 보이죠?

이 규칙을 잠깐 이전 영상의 내용과 연관 지어 생각해볼까요? 예를 들어, 제가

전체 직원 정보 조회 - GET  
새 직원 정보 추가 - POST  
이 작업들을 수행하기 위해 사용했던 'https://learn.codeit.kr/api/members' URL에서도
직원 전체를 의미하는 members는 이렇게 복수 명사를 사용했다는 것을 알 수 있습니다. members는 member들을 담을 수 있는 컬렉션에 해당하는 개념이기 때문입니다.

그리고 제가

특정 직원 정보 조회 - GET  
기존 직원 정보 수정 - PUT  
기존 직원 정보 삭제 - DELETE  
이 작업들을 수행하기 위해 사용했던 https://learn.codeit.kr/api/members/3 URL에서는
도큐먼트를 나타내기 위해 단수 명사 대신 직원 고유 식별자인 id 값을 썼는데요. 이렇게 숫자를 쓰는 경우에는 단복수 문제가 없겠죠?

'도큐먼트', '컬렉션' 개념을 우리가 배운 메소드 종류와 연결해서 모든 경우의 수를 생각해보면 다음과 같습니다.
![Alt text](/img/crud.png)
지금 표에서 보이는 것처럼, 전체 직원 정보를 대상으로 PUT 리퀘스트 또는 DELETE 리퀘스트를 보내는 것은 전체 직원 정보를 모두 수정 또는 모두 삭제한다는 뜻이기 때문에 사실상 잘 쓰이지 않습니다. 위험한 동작이기 때문에 애초에 Web API 설계에 반영하지도 않고, 서버에서 허용하지 않을 때가 일반적이죠.

그리고 또 여기서 주목할 점은 POST 리퀘스트를 보낼 때, 컬렉션(members) 타입의 리소스를 대상으로 작업을 수행한다는 점입니다. 이 부분이 조금 헷갈릴 수도 있는데요. POST 리퀘스트를 보낼 때는 우리가 전체 직원 정보를 의미하는 컬렉션에 하나의 직원 정보(하나의 도큐먼트)를 추가하는 것이기 때문에 URL로는 컬렉션까지만 /members 이렇게 표현해줘야 합니다. 따라서 /members/3 이렇게 특정 도큐먼트를 나타내는 URL에 POST 리퀘스트를 보내는 것은 문맥상 맞지 않는 표현입니다. 그리고 지금 같은 경우는 추가될 직원 정보가 어떤 id 값을 할당받을지 알 수도 없기 때문에 애초에 /members/[id]에 id 값을 지정한다는 것도 불가능하죠.

이 도큐먼트와 컬렉션 개념을 잘 기억하고 있으면 나중에 URL에서 단수 명사를 써야 할지, 복수 명사를 써야 할지 고민이 될 때 답을 얻을 수 있을 겁니다.

자, 이때까지 REST API의 조건 중 하나인 4. Uniform Interface을 좀 더 잘 지키기 위해 개발자들이 강조하는 규칙 2가지를 배웠습니다. 하지만 이것만으로 Web API를 REST API로 설계할 수 있는 것은 아닙니다. 여전히 만족시켜야 하는 다른 조건들도 있기 때문이죠. 나머지 조건들을 어떻게 지킬 수 있는지에 관한 내용은 난이도 및 분량 관계상 생략하겠습니다. 나머지 조건들을 어떻게 준수하는지는 여러분이 웹에 좀더 익숙해지고 나서 나중에 더 찾아보시는 걸 추천합니다.

REST API는 개발자들이 Web API를 설계할 때 굉장히 중요하게 고려하는 가이드라인이기는 하지만, 앞서 제시한 6가지 조건을 모두 만족시켜가면서까지 100% 준수해야 할 필요성이 있는지에 관해서는 의견이 많습니다.
