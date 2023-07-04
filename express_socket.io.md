# Socket.io

Socket.io가 무엇인지, 어떤 기능에 활용되는지 공부합니다.

## 시작에 앞서 알아야할 부분

### TCP와 UDP

> `TCP란?`  
> 서버와 클라이언트 간 데이터 송수신을 위해 만들어진 프로토콜  
> `연결 지향성 프로토콜`이라고도 부르는데 그 이유는 서버와 클라이언트가 양방향으로 소통함으로서 데이터 요청 및 전송에 있어 차질이 없도록 하여 데이터의 누락 또는 순서가 뒤바뀌는 일을 방지하는 것, 즉 통신의 안정성을 바탕으로 한 신뢰에 중점을 두기 때문입니다.  
> 가령 클라이언트가 데이터를 요청했고, 서버가 보내주는 과정에서 데이터가 누락되었다면 클라이언트에서 "누락됐다."라고 알려주면 서버가 다시 보내주는 방식으로 서로 소통하므로 아무래도 데이터 전송이 잘못된 가능성이 현저히 낮아 신뢰가 높아집니다.  
> 하지만 UDP와 비교했을 때 전송 속도가 느립니다. 왜 그럴까요?

> `UDP란?`  
> TCP와 비교했을 때 "전혀 소통하지 않는다." 라고 생각하시면 이해하기 쉬울 것 같습니다. 즉 서버는 던지고, 클라이언트는 받는 과정에서 TCP와 같은 소통이 없습니다. 그러니까 데이터가 누락되거나, 전송 순서가 바뀌거나해서 온전한 데이터를 전달해줄거란 신뢰성이 떨어질 수 밖에 없습니다. 왜냐하면 클라이언트에게 성공유무에 대한 피드백을 받는 구조가 아니기 때문입니다.  
> 하지만 아무래도 전송속도가 빠를 수 밖에 없겠죠. 소통에 필요한 지연시간이 없기 때문입니다. 이는 경우에 따라서 TCP보다 유리할 수 있습니다. 전송 비용이 적다는 것에서 얻는 이점들이 있기 때문입니다.

## What is Socket?

네트워크에서 데이터를 송수신하기 위해 반드시 거쳐야 하는 연결부를 의미합니다. 가전제품을 연결하는 콘센트 종류(110v, 220v)를 떠올리면 될 것 같습니다. 앞서 언급한 TCP와 UDP가 소켓에 해당합니다.

## 패킷(Packet)?

콘센트에 가전제품을 연결해서 작동시키면 전기 신호가 흐르게 되는 것과 같이 네트워크 통신을 통해 흐르는 데이터의 단위를 패킷이라고 합니다.

## 웹소켓(WebSocket)?

실시간 웹 서비스 제공을 위한 소켓입니다. 이는 주로 실시간 소통이 필요한 환경에 적용되는 기술입니다. 가령 구글 Docs 또는 메신저 등 실시간 공동 편집이 가능한 기술에 적용됩니다.  
하지만 웹소켓이 모든 브라우저에 대해서 동작을 보장하지는 않기 때문에 MDN에서 웹소켓의 호환성을 고려할 필요가 있습니다. 이 문제를 보완하기 위해 나온 것이 Socket.io입니다.

## socket.io?

자바스크립트에서 웹소켓 구현 시 가장 많이 사용되는 라이브러리 입니다.  
socket.io는 실시간 기능 구현에 있어 모든 사용자를 고려할 수 있도록 도와주는데, 특히 서버에서 데이터를 일정 간격마다 받아오는 polling 기능으로 실시간 기능 구현을 가능케 도와주는 것이 하나의 예시로 볼 수 있습니다.

### socket.io 실습

#### 패키지 설치

```javascript
npm install socket.io -S
```

#### Socket.io 기본 설정

```javascript
// app.js
const express = require('express');
const { Server } = require('http');
const socketIo = require('socket.io');

const app = express();
const http = Server(app);
const io = socketIo(http, {
  // cors: 소켓의 접근 가능 유저나 method를 제한합니다.
  cors: {
    origin: '*',
    methods: ['GET', 'POST'],
  },
});
const port = 3000;

// 유저가 서버에 접속하면 소켓을 연결합니다.
io.on('connection', (socket) => {
  console.log('새로운 소켓이 연결됐어요!');
});

app.use(express.json());
app.use(express.urlencoded({ extended: false }));
// app.use(cookieParser());
app.use(express.static('assets'));

app.get('/', (req, res) => {
  res.send('Hello');
}); //

http.listen(POST, () => {
  console.log(POST, '포트에 접속했습니다.');
});
```

클라이언트와의 socket을 통한 통신을 위해서는 이전과 다른 app.js 설정이 필요합니다.  
추가된 부분은 아래와 같습니다.

- `http` 패키지에서 `Server` 객체를 가져와 express 패키지인 app을 감싸주고 이를 또 다시 `socketIo`메서드로 감싸줍니다.
- 이제 app.listen이 아닌 http.listen으로 서버 접속 코드를 변경합니다.
- `io.on`을 통해 클라이언트와 서버간 소켓 통신을 준비합니다.

> 코드 설명  
> io.on 메서드에 'connection' 텍스트를 입력하면 소켓 연결 시 콜백함수를 수행하겠다는 의미입니다.  
> 현재는 `console.log('새로운 소켓이 연결됐어요!')` 기능밖에 없으므로 클라이언트에 접속할 때마다 해당 문구가 서버 터미널에 기록됩니다.
> 이는 socket이라는 파라미터를 통해 클라이언트와 통신을 주고 받습니다.

> 🚨 `주의`  
> `assets` 폴더가 `express.static`으로 등록되어 있으므로 `app.get('/')` 즉 루트 경로의 GET 요청 시 assets 폴더 내 index.html을 자동으로 클라이언트에 출력합니다.  
> 이는 node express의 특성 상 index 파일을 가장 먼저 찾기 때문입니다.  
> 만약 assets 폴더가 static으로 등록되지 않을 경우 루트 경로의 GET은 'hello' 텍스트를 프론트에 전달하게 됩니다.

#### Socket 통신

위에서 작성한 io.on을 통해 본격적으로 서버-클라이언트 간 통신을 수행합니다.  
통신 예시를 살펴보겠습니다.

```javascript
// 유저가 서버에 접속하면 소켓을 연결합니다.
io.on('connection', (socket) => {
  // console.log('새로운 소켓이 연결됐어요!');

  // 소켓 사용자 접속 시 바로 실행
  socket.emit('BUY_GOODS', {
    nickname: '서버가 보내준 구매자 닉네임',
    goodsId: 10,
    goodsName: '서버가 보내준 상품명',
    date: '구매 일시',
  });

  socket.on('BUY', (data) => {
    console.log('구매한 정보입니다.');
    console.log(data);
  });
});
```

socket을 파라미터로 받는 콜백 함수에서 `emit`메서드는 보낼 때, `on`은 받을 때라고 생각하면 됩니다.  
emit 메서드에서 'BUY_GOODS' 는 일종의 이벤트리스너의 네임과 같은 존재입니다. 클라이언트가 서버에 접속 시 socket이 연결되면  
그 즉시 'BUY_GOODS'라는 이름의 이벤트를 클라이언트에 보냄을 의미합니다.  
또한 on 메서드의 경우 클라이언트에서 'BUY'라는 이벤트를 받게 된다면 콜백함수를 실행함을 의미합니다.

#### 클라이언트 사이드

클라이언트에서는 socket를 어떻게 받아들이는지 확인합니다.

```html
<!-- index.html -->
<script src="https://cdn.socket.io/socket.io-3.0.1.min.js"></script>
<script>
  const socket = io.connect('/'); // 위 script가 연결되어야 io가 동작함

  socket.on('BUY_GOODS', function (data) {
    const { nickname, goodsId, goodsName, date } = data;
    makeBuyNotification(nickname, goodsName, goodsId, date);
  });
</script>
```

서버에서 보낸 'BUY_GOODS' 이벤트를 index.html에서 on 메서드를 통해 받아 콜백함수를 실행합니다.  
예시를 보시면 서버로부터 전달 받은 정보를 `data`라는 파라미터로 받고 있으며 이를 토대로 `구매 알림창`을 띄워주는 함수에 적용되어 실행된다는 것을 예상할 수 있습니다.  
현재 서버에서는 소켓이 연결만 되면 무조건 'BUY_GOODS' 이벤트를 emit하므로 유저는 index.html에 접속할 때 마다 'BUY_GOODS' 이벤트가 실행될 것입니다.

그렇다면 서버에서 보낼 때는 어떻게 활용될까요?

```javascript
function postOrder(user, order) {
  if (!order.length) {
    return;
  }

  socket.emit('BUY', {
    nickname: user.nickname,
    goodsId: order[0].goods.goodsId,
    goodsName:
      order.length > 1
        ? `${order[0].goods.name} 외 ${order.length - 1}개의 상품`
        : order[0].goods.name,
  });
}
```

위 예시는 구매 버튼 클릭 시 postOrder 함수가 수행됨을 가정하고 있습니다.  
구매 버튼 클릭 시 유저 정보를 오브젝트에 담아 서버의 'BUY' 이벤트로 전달(emit)합니다.  
그러면 서버에서 구매자의 정보를 받게 됩니다.

#### socket을 이용한 주문 페이지 코드 일부 살펴보기

```html
<!-- order.html -->
<script>
  $(document).ready(function () { // order페이지 로드 후 실행할 함수 등록 (jquery)

    getSelf(function (user) {
      const order = JSON.parse(sessionStorage.getItem("ordered"));
      $("#nickname").text(user.nickname);
      $("#btnOrder")
        .text(
          `$${number2decimals( // 소수 변환기
            order.reduce(
              (s, o) => s + Number(o.goods.price) * o.quantity,
              0
            )
          )} 결제`
        )
        .click(function () {
          postOrder(user, order);
        });
      $(".btn-confirm").click(function () {
        location.href = "/goods.html";
      });
    });
  });

  function number2decimals(num) {
    return (Math.round(num * 100) / 100).toFixed(2);
  }
```

getSelf함수를 통해 받은 유저 정보를 토대로 특정 태그에 value를 적용하고 있어 프론트 영역에 변화를 주는 것 같다.  
그 중 결제 버튼 클릭 시 적용되어야 할 postOrder 함수의 파라미터로 해당 유저 정보를 담아 click 이벤트를 등록한다.  
그렇다면 getSelf 함수는 어떤 함수일까?? 유저 정보를 프론트에 적용하는 함수가 파라미터로 들어가므로 getSelf의 콜백 함수로 사용될 것으로 예상된다.

```javascript
function getSelf(callback) {
  $.ajax({
    type: 'GET',
    url: '/api/users/me',
    success: function (response) {
      callback(response.user);
    },
    error: function (xhr, status, error) {
      if (status == 401) {
        alert('로그인이 필요합니다.');
      } else {
        localStorage.clear();
        alert('알 수 없는 문제가 발생했습니다. 관리자에게 문의하세요.');
      }
      window.location.href = '/';
    },
  });
}
```

getSelf함수는 ajax를 통해 서버와 비동기 통신을 한다. '/api/users/me' 경로로 GET 요청을 보내면 유저 관련 정보를 받는 것으로 보여진다.  
GET 요청이 성공하면 callback 파라미터(사실상 함수)에 유저 정보를 전달해 실행되도록 하고 있으며, 요청이 실패했을 경우에 대한 대처도 존재한다.

> `xhr?`  
> 요청에 대한 응답으로 에러를 받을 시 해당 요청에 대한 정보를 담고 있는 XMLHttpRequest (XHR) 객체를 받게된다.

정리하자면 ajax를 통해 유저 정보를 GET하여 성공하면 그 정보를 파라미터로 받았던 (콜백)함수에 담아 실행되도록 한다.

- 유저 정보 -> 프론트 적용 (프론트 버튼에 맞춤 함수 적용)
