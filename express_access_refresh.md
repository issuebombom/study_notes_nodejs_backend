# Access Token | Refresh Token

Access Token과 Refresh Token은 유저가 로그인 했을 때 발급되며 이후 해당 유저가 누구인지에 대해 인증하는 용도로 활용됩니다.  
가령 마이페이지에 접속할 경우 본인이 누구인지에 따라 그에 맞는 마이페이지를 보여줘야 하므로 이럴 때 사용되곘죠??

## workflow

- 로그인 시 엑세스 토큰 발급 + 리프레시 토큰 발급 (각각)
- 두 토큰 모두 쿠키로 전달하되 리프레시토큰은 DB에 {리프레시 토큰: userId} 형태로 저장
- 로그인 이후 물품 구매, 마이페이지 접속 등 본인이 누구인지 파악하기 위해 쿠키에 접근
- 쿠키에 두 토큰이 존재하는지 먼저 확인
- try, catch문으로 엑세스 토큰과 리프레시 토큰 검증 결과를 boolean으로 받는다. (isAccessTokenValid, isRefreshTokenValid)
- 만약 리프레시 토큰이 만료된 경우 재로그인이 필요 (이에 따른 경고, 로그인 페이지로의 유도가 필요)
- 엑세스 토큰이 만료된 경우 DB에서 유저의 리프레시 토큰(key)와 매칭되는 userId(value)를 꺼내서 엑세스 토큰을 재발급
- 재발급한 엑세스 토큰을 쿠키에 다시 담는다.

## 필요한 패키지 설치

```zsh
npm init -y
npm install express jsonwebtoken cookie-parser -S
```

## jwt와 cookie-parser 등록
cookie-parser 패키지를 활용하지 않으면 쿠키값을 가져오지 못하므로 미들웨어로 사용하자

```javascript
const jwt = require('jsonwebtoken');
const cookieParser = require('cookie-parser');
const express = require('express');
const app = express();
const port = 3000;

app.use(cookieParser()); //

app.get('/', (req, res) => {
  res.status(200).send('Hello');
});

app.listen(port, () => {
  console.log(port, '포트로 서버가 열렸어요!');
});
```

## 토큰 생성

```javascript
let tokenObject = {}; // Refresh Token을 저장할 Object (실습을 위한 DB 대용으로 생각하자)

app.get('/set-token/:id', (req, res) => {
  const id = req.params.id;
  const accessToken = createAccessToken(id);
  const refreshToken = createRefreshToken();

  tokenObject[refreshToken] = id; // Refresh Token을 가지고 해당 유저의 정보를 서버에 저장한다.
  res.cookie('accessToken', accessToken); // Access Token을 Cookie에 전달한다.
  res.cookie('refreshToken', refreshToken); // Refresh Token을 Cookie에 전달한다.

  return res.status(200).send({ message: 'Token이 정상적으로 발급되었습니다.' });
});

// Access Token을 생성
function createAccessToken(id) {
  const accessToken = jwt.sign(
    { id: id }, // JWT 데이터
    SECRET_KEY, // 비밀키 (환경변수(.env)로 전달해주자)
    { expiresIn: '10s' }
  ); // Access Token이 10초 뒤에 만료되도록 설정

  return accessToken;
}

// Refresh Token을 생성
function createRefreshToken() {
  const refreshToken = jwt.sign(
    {}, // JWT 데이터
    SECRET_KEY, // 비밀키
    { expiresIn: '7d' }
  ); // Refresh Token이 7일 뒤에 만료되도록 설정

  return refreshToken;
}
```

## 토큰 검증

```javascript
app.get('/get-token', (req, res) => {
  const accessToken = req.cookies.accessToken; // 쿠키에서 엑세스토큰 획득
  const refreshToken = req.cookies.refreshToken; // 쿠키에서 리프레시토큰 획득

  // 쿠키에 토큰이 없을 경우 (재 로그인)
  if (!refreshToken) return res.status(400).json({ message: 'Refresh Token이 존재하지 않습니다.' });
  if (!accessToken) return res.status(400).json({ message: 'Access Token이 존재하지 않습니다.' });

  // 쿠키 유효성 검증
  const isAccessTokenValidate = validateAccessToken(accessToken);
  const isRefreshTokenValidate = validateRefreshToken(refreshToken);

  // 리프레시 토큰 만료 시 (재 로그인 유도)
  if (!isRefreshTokenValidate)
    return res.status(419).json({ message: 'Refresh Token이 만료되었습니다.' });

  // 엑세스 토큰 만료 시 리프레시 토큰을 활용하여 재발급 한다.
  if (!isAccessTokenValidate) {
    const accessTokenId = tokenObject[refreshToken];
    if (!accessTokenId)
      return res.status(419).json({ message: 'Refresh Token의 정보가 서버에 존재하지 않습니다.' });

    const newAccessToken = createAccessToken(accessTokenId);
    res.cookie('accessToken', newAccessToken);
    return res.json({ message: 'Access Token을 새롭게 발급하였습니다.' });
  }

  // 쿠키 인증을 성공하면 토큰에 저장된 유저 id를 가져와 앞으로 활용한다.
  const { id } = getAccessTokenPayload(accessToken);
  return res.json({ message: `${id}의 Payload를 가진 Token이 성공적으로 인증되었습니다.` });
});

// Access Token을 검증합니다.
function validateAccessToken(accessToken) {
  try {
    jwt.verify(accessToken, SECRET_KEY); // JWT를 검증합니다.
    return true;
  } catch (error) {
    return false;
  }
}

// Refresh Token을 검증합니다.
function validateRefreshToken(refreshToken) {
  try {
    jwt.verify(refreshToken, SECRET_KEY); // JWT를 검증합니다.
    return true;
  } catch (error) {
    return false;
  }
}

// Access Token의 Payload를 가져옵니다.
function getAccessTokenPayload(accessToken) {
  try {
    const payload = jwt.verify(accessToken, SECRET_KEY); // JWT에서 Payload를 가져옵니다.
    return payload;
  } catch (error) {
    return null;
  }
}
```

> `NOTE`  
> POSTMAN이나 Thunder Client로 API 테스트를 완료하면 프론트와의 연결을 고려해야 한다.
> 현재 토큰 재발급 이후 message를 리턴하지만 그러지 말고 인증 이후의 단계로 연결시켜줘야 한다는 점을 기억하자
> 지금 코드 흐름대로라면 엑세스 쿠키 재발급 이후의 액션이 없으므로 유저는 10초 마다 재발급 메시지를 만나야 한다.
> 그러므로 토큰 인증성공 또는 재발행 이후 추가적인 활동을 고려한 코드 수정이 필요하다.
