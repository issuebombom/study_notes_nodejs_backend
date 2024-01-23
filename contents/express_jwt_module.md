# 로그인, 인증 기능 구축을 위한 기능
## 인증기능이 왜 필요할까?
서버가 클라이언트로부터 어떠한 정보를 요청받았을 때 해당 클라이언트가 누구냐에 따라 응답 데이터가 달라지거나 또는 제공해서는 안되는 보완적인 절차가 필요한 경우가 있기 떄문이다.  

가령 A 유저에게 B 유저에 대한 개인정보, 플레이리스트, 결제내역 등을 보여줘서는 안되기 때문이다.  

이러한 클라이언트 구분을 위해 로그인 시스템이 존재한다. 최초 로그인 시 클라이언트는 로그인 정보를 body에 담아 전달하면 서버는 그것을 내장 시크릿 키와 잘 버무려 `토큰` 이라는 hashing text를 생성하여 클라이언트에게 제공해준다. 이 토큰은 곧 인증서 역할을 한다. 그래서 클라이언트는 로그인한 후 해당 사이트에서 여러가지 요청(게시글 보기, 플레이리스트 보기 등)을 할 때 인증서를 같이 제출하여 내 신원에 맞는 응답을 받게 되는 것이다.  

## 인증 및 인가 절차 기본 흐름
1. 클라이언트에서 본인이 누구인지 밝힌다.(로그인)
2. 서버는 유저 정보를 포함하는 토큰을 생성한다.
3. 클라이언트에게 토큰을 보낸다.(HTTP Response Header에 담아 보낸다.)
4. 클라이언트에서 토큰을 저장한다.(Cookie나 localStorage 등)
5. 서버에 특정 요청을 할 때 늘 저장해 둔 토큰을 Header에 담아 서버에 Request를 보낸다.
6. 서버는 클라이언트가 요청한 task에 응답하기 전, 받은 토큰과 내부에 저장된 Signature를 조합한 토큰을 생성하여 전달받은 토큰과 비교하여 일치하는지에 대한 검증 절차를 먼저 갖는다.
7. 토큰의 일치 여부에 따라 클라이언트의 요청에 응답 또는 거부한다.

## 토큰의 생성 방법 (JWT 모듈)
> 📌 What is JWT?  
> JSON Web Token의 약자로 당사자간의 정보를 JSON객체로 안전하게 전송 또는 유저의 권한 체크를 위해 사용하는 모듈

[JWT 공식 사이트](https://jwt.io/)

### JWT 인코딩과 디코딩
![JWT-example](./img/JWT_exam.png)
위 사진에서 암호화 되어있는 토큰을 서버에서 생성하여 클라이언트에 보내준다. 이 토큰은 `HEADER`, `PAYLOAD`, `VERIFY SIGNATURE`의 조합으로 생성된다.  
JWT는 색깔로 구분된 것처럼 총 세 단계의 `세그먼트`로 분류가 된다.  
> `HEADER`: 해당 토큰에 적용된 해싱(Hashing) 알고리즘과 데이터 타입과 같은 메타데이터에 해당  
> `PAYLOAD`: 클라이언트로부터 전달받은 유저정보와 더불어, 만료기간, 주제와 같은 각종 정보를 담은 부분에 해당  
> `VERIFY SIGNATURE`: 클라이언트에서 전달받은 토큰이 유효한지를 검증하는 부분, 헤더와 페이로드를 인코딩한 값과 서버에 저장된 Secret 키를 조합하여 해당 세그먼트값을 만든다.

## 간단한 인증 시스템 구현
### 환경 설정
가장 먼저 npm 초기화
```zsh
npm init -y
```
아래 패키지 설치 필요
```zsh
npm install express dotenv jsonwebtoken
```

### 토큰 생성 API 구현
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

const PORT = 4000;
const secretText = 'supersecret'; // 서버 내 보관할 시크릿 키

app.use(express.json());

app.post('/login', (req, res) => {
  const username = req.body.username;
  const user = { username };

  const accessToken = jwt.sign(user, secretText);
  res.json({ accessToken });
});

app.listen(PORT, () => {
  console.log('listening on port ' + PORT);
});
```
`jsonwebtoken` 패키지를 통해 post body로 받은 username json 데이터와 시크릿 키를 가지고 accessToken을 생성한다. 우선 시크릿 키는 secretText라는 변수로 간단하게 표현했지만 실제로는 dotenv 패키지를 활용하여 환경변수에 숨겨야 한다.  
토큰 생성 결과는 아래와 같다.  
```json
{
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImpvaG4iLCJpYXQiOjE2ODYxNTMzOTl9.rFxpq_s_a9eTbBWqv2SXXMrSVbB314hfxCi2ROkOR_0"
}
```

> 💡 `Tip`  
> `postman`이라는 앱을 설치하면 html 없이 request를 보낼 수 있어 response 결과를 바로 확인해 볼 수 있다. [사이트 확인](https://www.postman.com/)
![postman-example](./img/postman_exam.png)

### 클라이언트에서 토큰으로 요청하기
우선 테스트를 위해 토큰 유효성이 검증되었을 경우 출력할 임시 데이터와 API를 준비합니다.
```javascript
// 임시 포스트 데이터
const posts = [
  {
    username: 'Apple',
    title: 'Title 1'
  },
  {
    username: 'Ball',
    title: 'TItle 2'
  }
];

// /posts로 요청 시 posts데이터를 제공하는 API
app.get('/posts', authMiddleware, (req, res) => res.json(posts));
```
GET 메소드를 보면 미들웨어 함수가 추가되어 있는 것을 확인할 수 있다. 이처럼 express에서는 미들웨어를 활용하여 검증 과정을 추가한다.  

검증 과정에 대한 미들웨어는 아래와 같이 구현이 가능하다.
```javascript
function authMiddleware(req, res, next) {
  // 토큰 가져오기
  const authHeader = req.headers.authorization;

  // authorization 유무를 체크하여 split 한다. (Bearer 부분 뗴어내기)
  const token = authHeader && authHeader.split(' ')[1];
  if (token == null) return res.sendStatus(401);

  // 토큰 확인
  jwt.verify(token, secretText, (err, user) => {
    if (err) return res.sendStatus(403);

    // 다음 미들웨어에서 req.user로 유저 정보를 활용할 수 있게 전달해준다.
    req.user = user;
    next();
  });
}
```
미들웨어의 프로세스를 찬찬히 살펴보자
1. 요청과 함께 받은 헤더에서 auth 정보를 가져온다.
2. auth정보가 빈 값이 아니라면 `Bearer` 부분을 뗴어내기 위해 split한다. (일반적으로 authorization을 보낼 때 앞에 Bearer를 붙인다.)  
3. auth가 없다면 401 에러를 보낸다.
4. 요청으로 전달 받은 토큰과 서버 내 시크릿 키를 jwt.verify 파라미터로 전달하여 error 또는 user값을 리턴한다.  

> 📌 `정리하자면`  
> 서버로부터 토큰을 제공받을 때 user정보 json 파일을 body에 담아 서버에 전달  
> 서버에서는 user 데이터와 내부 시크릿 키를 토대로 토큰을 생성  
> 토큰을 클라이언트에 전달  
> 클라이언트에서 posts(게시글) 정보를 보여달라고 요청한다. 이 때 저장해뒀던 토큰을 헤더의 Authorization에 담아 함께 보낸다.  
> 서버에서는 auth가 있는지 먼저 확인한다. auth가 있다면 jwt 검증을 통해 토큰의 일치여부를 판단한다.  
> 일치 여부에 따라 요청에 응답하거나 거부한다.  
> 위 코드에는 없지만 만약 유저의 권한에 따라 허용 또는 불허하는 과정이 있다면 미들웨어를 추가해서 user정보의 권한(등급)에 따른 검증 절차를 추가한다.

## Refresh Token
### What is Refresh Token?
로그인 후 생성된 토큰을 사실 `Access Token`이라 부른다. 실제로는 Access Token에 대하여 유효기간을 지정해서 사용해야 한다. 그 이유는 해커와 같은 제 3자에게 악의적인 용도로 Token이 탈취된다면 유저가 재로그인을 한다 해도 기존에 생성된 토큰이 유효하기 때문에 보완상 위험에 처하게 되기 때문이다.  
하지만 만약 Access Token의 유효기간이 매우 짧게 설정되어 있다면 유저는 매번 재로그인을 시도해야 하고, 반대로 너무 길게 설정되어 있다면 보완상 의미가 없어진다.  
금융 관련 서비스 이용 시에는 유저가 번거롭더라도 유효기간을 짧게 하는 것이 타당할 수 있지만 그 외의 경우에는 적절한 타협점을 찾아야 하는 어려움이 있는데 이를 위한 해결책으로 `Refresh Token`이 나왔다.  

### Refresh Token의 용도
`Refresh Token`은 기존에 클라이언트가 가지고 있던 Access Token이 만료될 경우 재발급 받기위한 용도로 사용한다. 그래서 Access Token의 유효기간은 짧게, Refresh Token은 길게 설정한다.

### 토큰에 대한 클라이언트 서버의 Flow
[기존에 정리한 인증 절차의 흐름](#인증-및-인가-절차-기본-흐름)에서 달라진 점은 아래와 같다.  
- 최초 서버에서 엑세스 토큰과 리프레시 토큰을 함께 발급하여 클라이언트에 전달한다.
- 서버에서 리프레시 토큰은 주로 DB에 보관한다.
- 클라이언트에서 보낸 엑세스 토큰이 만료된 경우 서버에서는 'invalid token error'를 응답하고, 이 때 클라이언트는 리프레시 토큰을 보낸다.  
- 서버에서 리프레시 토큰이 검증되면 새로운 엑세스 토큰을 재발행하여 클라이언트에 함께 전달한다.

> 📌 Authorization Flow  
> ![auth_flow_example](./img/auth_flow_exam.png)

### 로그인 시 Access, Refresh 토큰 생성 API 
```javascript
// 임시 DB
let refreshTokens = [];

// 로그인 POST 요청
app.post('/login', (req, res) => {
  const username = req.body.username;
  const user = { username };

  // 토큰 생성 시 유효기간 추가
  const accessToken = jwt.sign(user, secretText, 
    { expiresIn: '15s' });
  const refreshToken = jwt.sign(user, refreshSecretText, 
    { expiresIn: '1d' });

  // 리프레시 토큰 DB에 저장
  refreshTokens.push(refreshToken);

  // 리프레시 토큰 쿠키에 넣어주기 (쿠키 이름: jwt로 설정)
  res.cookie('jwt', refreshToken, {
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000 // 만료 기간 설정 (밀리세컨 단위)
  });

  res.json({ accessToken });
});
```
[이전에 작성한 코드](#토큰-생성-api-구현)와 비교했을 때 차이점은 
1. 리프레시 토큰 생성 및 DB에 저장 (DB는 배열로 임시 구현)
2. 두 개의 토큰 모드 유효기간 설정
3. 쿠키에 리프레시 토큰을 담아서 전달 (httpOnly 옵션, 쿠키 유효기간 설정)

> 💡 `HttpOnly?`  
> 이 옵션을 설정할 경우 자바스크립트 명령을 통해 쿠키를 가져올 수 없도록 하여 해킹 툴과 같은 악성 도구로 부터 쿠키 수집에 대한 방어를 할 수 있다.

### 리프레시 토큰 생성 API
```javascript
const cookieParser = require('cookie-parser');
app.use(cookieParser())

// 리프레시 토큰
app.get('/refresh', (req, res) => {
  // cookies안에 jwt라는 이름의 쿠키가 있는지 확인
  const cookies = req.cookies; // cookie-parser 모듈이 설치되었기 때문에 리턴 가능
  if (!cookies?.jwt) return res.sendStatus(403);

  const refreshToken = cookies.jwt; // jwt 쿠키를 토큰으로 선언
  // DB에 저장된 쿠키와 일치하는지 검증
  if (!refreshTokens.includes(refreshToken)) return res.sendStatus(403);

  jwt.verify(refreshToken, refreshSecretText, (err, user) => {
    // 토큰 검증
    if (err) return res.sendStatus(403);

    // accessToken 생성
    const accessToken = jwt.sign({ username: user.username },
      secretText,
      { expiresIn: '15s' }
    );
    // 엑세스토큰 전달
    res.json({ accessToken });
  });
});
```
GET 메서드의 Response를 받으면 쿠키가 자동으로 함께 전달된다. 그러므로 특별한 요청 없이 바로 `res.cookies` 명령을 통해 쿠키값들을 오브젝트로 가져올 수 있다. 하지만 `npm install cookie-parser`를 통해 cookieParser를 설치하여 require 및 미들웨어 등록을 해야 정상적으로 값을 얻을 수 있게 된다. 위 모듈이 없을 경우 undefined를 받게 된다.

위 코드의 work flow는 아래와 같다.
1. jwt라는 키값으로 저장된 쿠키가 있는지 검증한다.
2. DB에 저장된 리프레시 토큰이 쿠키에 저장된 토큰과 같은지 검증한다.
3. 검증을 통과하면 리프레시 토큰에 담았던 유저 정보(`user`)에서 username 부분을 토대로 엑세스 토큰을 재발생한 뒤 프론트에 전달한다.

