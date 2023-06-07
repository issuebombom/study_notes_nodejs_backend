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
