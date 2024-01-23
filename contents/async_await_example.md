# 비동기 함수 실행 이해하기

## STEP 1

```javascript
async function fetchAndPrint() {
  console.log(2);
  const response = await fetch('https://jsonplaceholder.typicode.com/users');
  console.log(7);
  const result = await response.json();
  console.log(8);
  console.log(result[0].name);
}

// 시작
console.log(1);
fetchAndPrint();
console.log(3);
console.log(4);
console.log(5);
console.log(6);

// 결과: 1, 2, 3, 4, 5, 6, 7, 8, Leanne Graham
```

fetchAndPrint()는 비동기 함수인 fetch가 await와 함께 있다.  
그래서 fetch는 백엔드에 보낸 요청에 대한 결과를 받을 때까지(pending상태) 기다린다(await).  
fetch때문에 frozen 상태가 된 fetchAndPrint() 함수는 일단 제쳐두고, 코드 실행 흐름은 함수 바깥으로 빠져 나와 다음 코드를 순차적으로 실행한다.  
만약 순차 실행 중 fetch가 요청 결과를 받게되면(fulfilled상태) fetchAndPrint() 함수의 frozen 상태가 해제되고, 코드 실행 흐름은 다시 fetchAndPrint() 함수로 돌아와 fetch 이후의 코드를 실행한다.  
그래서 fetch의 결과를 response 변수로 선언하고, 숫자 7을 출력한다.  
그리고 await response.text() 가 만약 fulfilled 상태가 되기까지 오래 걸린다면 fetchAndPrint() 함수는 또 다시 frozen 상태가 될 것이다.

정리하자면 코드 실행 중 await가 붙은 fetch를 만나면, fetch가 fullfilled상태가 되기 전까지 async 함수를 빠져나와 다음 코드가 실행된다.

위 코드예시의 경우 fulfilled가 되기 전에 남은 코드가 전부 실행을 완료 한다. 그 이유는 남은 console.log() 실행에 걸리는 시간을 모두 합해도 fetch가 더 오래 걸리기 때문이다.

## STEP 2

```javascript
async function fetchAndPrint() {
  console.log(2);
  const response = fetch('https://jsonplaceholder.typicode.com/users');
  console.log(response); // pending 상태
  console.log(7);
  console.log(8);
}

// 시작
console.log(1);
fetchAndPrint();
console.log(3);
console.log(4);
console.log(5);
console.log(6);

// 결과: 1, 2, Promise [ pending ], 7, 8, 3, 4, 5, 6, fetch(fulfilled) 시 실행 종료
```

위 코드에서는 fetch함수에 await를 적용하지 않았다.  
그러므로 아무도 fetch가 fulfilled상태가 될 때까지 기다려주지 않는다.  
그래서 console.log(7) 코드가 fetch 요청 직후 즉시 실행된다.  
현재 모든 console.log() 실행에 걸리는 시간을 모두 합해도 fetch함수가 fulfilled 되는데 걸리는 시간이 훨씬 길다.

await를 붙이는 이유는 뭘까?

- fetch가 fulfilled상태가 되어야 요청한 데이터를 response 변수에 할당할 수 있다. 그래서 await를 붙여 fulfilled가 될 때 까지 기다려주는 것이다. pending 상태는 아직 백엔드 단에서 처리중이라는 의미이므로 당연히 받을 데이터도 없다.
- await가 붙은 코드 아래에(다음에) 위치한 코드의 실행을 잠시 대기시키기 위함도 있다. 아래 코드를 대기시키는 이유는 await의 결과물을 활용해야 하기 때문이다.

## STEP 3

```javascript
let array = [];

const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('7');
  }, 1000);
});

const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('8');
  }, 3000);
});

const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('9');
  }, 1000);
});

const p4 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('10');
  }, 3000);
});

async function promiseFunc1() {
  console.log(2);
  array.push(await p1); // 1초
  array.push(await p2); // 3초 -> 2초 간 대기
  console.log('F1:', array);
}

async function promiseFunc2() {
  array.push(await p3); // 1초
  array.push(await p4); // 3초
  console.log('F2:', array);
}

// 시작
console.log(1);
promiseFunc1();
promiseFunc2();
console.log(3);
console.log(4);
console.log(5);
console.log(6);

// 결과: 1, 2, 3, 4, 5, 6, F1: [ '7', '9', '8' ], F2: [ '7', '9', '8', '10' ]
```

promiseFunc1()에서 p1을 await한다. setTimeout 함수를 통해 p1이 fulfilled 되기까지 걸리는 시간은 1초이다. 그때 동안 promiseFunc1()는 frozen되므로 빠져나온다.  
promiseFunc2()에서 p3을 await 한다. setTimeout 함수를 통해 p3가 fulfilled 되기까지 걸리는 시간은 1초이다. 그때 동안 promiseFunc2()는 frozen되므로 빠져나온다.  
1초가 지나기도 전에 함수 외부의 console.log() 들이 전부 실행된다. console.log() 실행 소요 시간이 매우 짧기 때문이다.  
p1이 가장 먼저 fulfilled될 것이다. 왜냐하면 p3와 소요시간은 같지만 먼저 시작했기 때문이다.  
promiseFunc1()의 p1이 fulfilled되어 다음 코드인 p2가 실행된다. setTimeout 함수를 통해 p2가 fulfilled 되기까지 걸리는 시간은 3초이다.  
이후 promiseFunc2()의 p3가 fulfilled 상태가 되어 다음 코드인 p4가 실행된다. setTimeout 함수를 통해 p2가 fulfilled 되기까지 걸리는 시간은 3초이다.  
promiseFunc1()의 p2가 fulfilled되어 그 다음 코드인 console.log('F1:', array); 가 실행된다.  
이제 promiseFunc2()의 p4가 fulfilled되어 그 다음 코드인 console.log('F2:', array) 가 실행된다.

총 4개의 promise 객체를 상단에 전역변수로 선언했는데 신기한 점은 선언한 순서가 fulfilled 상태 도달 시간에 영향을 준다는 점이다.  
fulfilled 도달 시간이 거의 동일할 경우 먼저 선언된 promise 객체가 먼저 fulfilled에 도달하는 것을 확인할 수 있었다. (p1~p4의 선언 위치를 바꿔보면 알 수 있다.)  
예상하기로는 promiseFunc에서 p 변수를 접했을 때 p변수가 어디에 선언되어 있는지 가장 윗줄부터 순차적으로 탐색하기 때문이 아닐까 예상한다.

## Appendix

setTimeout 함수가 즉시 실행함수로 구성되었다.  
아래 코드를 실행해보자, 10번 정도 돌려보면 결과가 늘 같지는 않다는 것을 확인할 수 있을 것이다.  
이는 fetch의 처리 시간이 약 600ms보다 늦다면 5, 1, 2, 6, 7...순서로 나타나지만 600ms보다 빠르다면 5, 1, 6, 7, 2... 순서로 나타날 수 있기 때문이다.  
이를 통해 비동기 함수는 fulfilled 상태가 되면 코드 실행 순서가 비동기 함수 다음 순서로 옮겨 간다는 것을 확인할 수 있다.

```javascript
async function fetchAndPrint() {
  console.log(5);
  const response = await fetch('https://jsonplaceholder.typicode.com/users');
  console.log(6);
  const result = await response.json();
  console.log(7);
  console.log(result[0].name); // Leanne Graham
}

(() => {
  setTimeout(() => {
    console.log('1');
  }, 300);
})();

fetchAndPrint();

(() => {
  setTimeout(() => {
    console.log('2');
  }, 600);
})();

(() => {
  setTimeout(() => {
    console.log('3');
  }, 1500);
})();

(() => {
  setTimeout(() => {
    console.log('4');
  }, 2000);
})();
```
