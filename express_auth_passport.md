# PASSPORT를 활용한 유저 인증 기능 구현

타입스크립트 기반 웹페이지 구현 프로젝트에서 로그인 및 인증, 인가 기능을 passport 패키지를 활용하여 구현하는 과정에서 참조한 코드를 리뷰하는 형태로 설명하고자 합니다.

> 아래 블로그를 참고하여 passport 기능 구현을 공부했습니다.  
> [블로그 참조](https://inpa.tistory.com/entry/NODE-%F0%9F%93%9A-Passport-%EB%AA%A8%EB%93%88-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%B2%98%EB%A6%AC%EA%B3%BC%EC%A0%95-%F0%9F%92%AF-%EC%9D%B4%ED%95%B4%ED%95%98%EC%9E%90)

## passport 패키지

이전까지 jwt와 쿠키를 활용한 Access Token, Refresh Token 생성 및 인증 기능을 구현하였다. 이번 시간에는 passport라는 패키지를 활용하여 인증 기능을 구현해 보고자 한다. 이는 개인적으로 구현하는 인증 기능보다 더 많은 기능이 담겨 있으며, 무엇보다 검증된 패키지로서 보완 측면에서 좀 더 신뢰할 수 있지 않을까? 라는 막연한 기대감이 있기에 사용해 보았다.

```zsh
# 패키지 설치
npm install passport passport-local
```

passport-local 패키지를 함께 설치하여 로컬 환경에서의 `인증 전략`을 구현하고자 한다. passport 패키지에서 내부적으로 제공해주는 여러 기능을 활용하여 나만의 로컬 인증 전략을 구사하는 것이 목표이다.

## import passport

```typescript
// app.js
import dotenv from 'dotenv';
dotenv.config();

import express, { Express } from 'express';
import cookieParser from 'cookie-parser';
import bodyParser from 'body-parser';
import session from 'express-session';
import passport from 'passport';

import passportConfig from './passport'; // ./passport/index.ts
import { authRouter } from './routes';

const app: Express = express();
const port: number = 3000;

passportConfig(); // 패스포트 설정
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(cookieParser(process.env.COOKIE_SECRET_KEY));
app.use(
  session({
    resave: false,
    saveUninitialized: false,
    secret: process.env.COOKIE_SECRET_KEY as string,
    cookie: {
      httpOnly: true,
      secure: false,
    },
  })
);
app.use(passport.initialize()); // 요청 객체에 passport 설정을 심는다.
app.use(passport.session()); // req.session 객체에 passport정보를 추가 저장

//* 라우터
app.use('/auth', authRouter);

app.listen(port, () => {
  console.log(port, '포트로 접속하였습니다.');
});
```

먼저 기본적으로 body, cookie 리딩에 필요한 패키지 설정과 세션 활용을 위한 express-session 패키지가 미들웨어로 초기 설정되는 것을 확인할 수 있다.  
그 외 passport 설정에 관한 코드는 `passportConfig()`와 `passport.initialize()`, `passport.session()`에 해당하며 이를 활용한 로그인, 인증, 인가 활동은 `authRouter`에서 구체적으로 진행된다.

`passportConfig()` 함수는 추후 설명할 passport/index.ts 파일의 실행에 해당한다.

이제 본격적으로 위와 같은 기본 설정이 왜 필요했는가에 대해서 살펴보도록 하자

## 로그인 전략

### STEP 1. 로그인 요청

```typescript
// authControllers.ts
login = async (req: Request, res: Response, next: NextFunction) => {
    interface Info {
      message: string;
    }

    try {
      // localstrategy.js 실행
      passport.authenticate('local', (error: Error | null, user: false | Client, info: Info) => {
        //* localStrategy의 결과로 done 콜백함수가 실행된다.
    // ...
```

authRouter를 통해 먼저 login 요청이 들어온다면 위 login 함수가 실행되도록 합니다. 이 때 `passport.authenticate` 함수를 실행하여 passport 패키지를 활용한 로그인 인증 절차를 밟는다. 이 때 해당 함수의 파라미터로 `'local'`을 넘겨주면 앞서 설치했던 `'passport-local'` 패키지를 통해 Strategy(인증 전략)가 구현된 코드가 있는 파일로 직접 찾아 이동한다.  
그리고 인증 전략 파일의 실행 결과는 `'local'` 파라미터 뒤에 위치한 콜백 함수의 실행으로 이어지게 된다. 여기서는 해당 콜백 함수를 done 함수라고 부른다.  
해당 구조에 익숙하지 않을 수도 있겠지만 결론적으로 로그인 인증 절차를 통해 얻은 결과를 done 콜백함수의 파라미터로 지정하여 이를 실행하는 구조라고 생각하면 되겠다.

### STEP 2. 로그인 인증

```typescript
// ./passport/localStrategy.ts
import passport from 'passport';
import passportLocal from 'passport-local';
import bcrypt from 'bcrypt';
import { Client } from '../db';

const LocalStrategy = passportLocal.Strategy;

export = () => {
  //? auth 라우터에서 /login 요청이 오면 local설정대로 이쪽이 실행되게 된다.
  passport.use(
    new LocalStrategy(
      {
        // req.body와 일치시킬 것
        usernameField: 'email', // req.body.email
        passwordField: 'password', // req.body.password
      },
      async (email, password, done) => {
        // done()의 첫번째 함수는 에러용으로 처리함
        try {
          const foundUser = await Client.findOne({ where: { email } });
          if (foundUser) {
            const result = bcrypt.compareSync(password, foundUser.password);
            if (result) {
              done(null, foundUser); // 성공 (done이 수행되면 다시 왔던 곳으로 돌아감)
            } else {
              done(null, false, { message: '비밀번호가 일치하지 않습니다.' });
            }
          } else {
            done(null, false, { message: '가입되지 않은 회원입니다.' });
          }
        } catch (error) {
          console.error(error);
          done(error);
        }
      }
    )
  );
};
```

앞서 말한 바와 같이 `passport.authenticate('local')`이 실행되면 LocalStrategy가 실행된 코드를 찾아 온다.

```typescript
import passportLocal from 'passport-local';
const LocalStrategy = passportLocal.Strategy;
```

그리고 나서 `new LocalStrategy`를 통해 body로 입력받은 이메일과 비밀번호 정보를 획득한 뒤 여러 인증과정을 거친다. LocalStrategy 클래스의 첫번째 파라미터로 프론트의 body 정보를 획득하고, 두번째 파라미터로 인증 전략을 담는다.

`async (email, password, done)` 함수가 곧 인증 전략 수행 함수를 의미한다. 내부적으로 회원 여부를 먼저 파악한 뒤 비밀번호 검증을 마치면 결과를 done 함수의 파라미터로 담아 전달한다.

여기서 개인적으로 주목했던 점은 로직의 실행 순서였다. 해당 로직을 살펴보면 만약 성공에 해당할 로그인 요청이 들어왔을 경우 실패 경우에 대한 로직을 전혀 거치지 않도록 짜여져 있다. 개인적으로 나는 validator와 실패 결과 처리에 대한 로직을 앞부분에 비치하였고, 마지막에 성공 결과 처리 로직을 비치했었는데 위 코드를 통해 실패 결과 처리 로직을 성공 케이스가 굳이 거쳐갈 필요가 없도록 코드를 짜야겠다는 점을 배울 수 있었다.

또한 주목해볼 점은 done 함수의 활용법입니다. done 콜백 함수의 파라미터로 총 3개를 받고 있는데 (error, data, message) 이렇게 받고 있습니다. 에러(Error)의 경우 검증 외 시스템적인 문제로 인해 발생한 에러를 전달하고 있고, data는 요청 결과를, message는 검증 과정을 통과하지 못했을 경우 해당 메시지를 전달하는 용도로 설정되어 있습니다. 이렇듯 done 함수의 파라미터를 본인이 세운 인증 전략에 따라 자유롭게 구현할 수 있음을 볼 수 있습니다.

어쨌든 결론적으로 LocalStrategy 클래스로 인증 절차용 인스턴스가 생성되어 passport.use를 통해 검증을 실시하고 그 결과를 done이라는 함수의 실행으로 이어진다는 것을 확인할 수 있다.

### STEP 3. 로그인 성공 결과 기록 남기기

```typescript
// authControllers.ts
passport.authenticate('local', (error: Error | null, user: false | Client, info: Info) => {
  //* localStrategy의 결과로 done 콜백함수가 실행된다.
  // done(err)가 발생한 경우
  if (error) {
    throw error;
  }
  // 유저 이슈가 발생한 경우
  if (!user) {
    return res.status(403).send({ message: info.message });
  }

  //* done에서 user값을 제대로 가져온 경우(성공한 경우)
  // passport.serializeUser 함수로 이동
  return req.login(user, (loginError) => {
    // deserializeUser 함수의 done이 실행되면
    // done(err) 발생 시
    if (loginError) {
      throw loginError;
    }
    // 세션에 사용자 정보를 저장하여 로그인 상태가 된다.
    return res.send({ message: '로그인 완료' });
  });
})(req, res, next);
```

LocalStrategy.ts의 결과를 done 콜백함수로 전달한다. 위 코드에서 `(error: Error | null, user: false | Client, info: Info)`에 해당한다.  
먼저 검증 외 에러가 발생했을 경우에 대한 처리 (error)와 검증 실패에 대한 메시지 처리를 해준 뒤 이를 통과한다면 user를 파라미터로 담은 `req.login` 함수 실행으로 연결되고 있다. 여기서 user는 이메일 조회를 통해 찾는 유저 데이터이다.  
`req.login`함수의 경우 실행되면 `passport.serializeUser` 함수로 연결되도록 구성되어 있다. 이는 유저 정보를 '직렬화'하여 세션에 저장하는 과정을 담고 있다.

잠시 처음에 봤던 app.ts 파일 중 일부를 확인해보면

```typescript
// ...
import passport from 'passport';

import passportConfig from './passport';
import { authRouter, userRouter, showRouter, placeRouter } from './routes';

const app: Express = express();
const port: number = 3000;
passportConfig(); // 패스포트 설정
// ...
```

passportConfig() 함수가 실행된 것을 확인할 수 있는데 이는 passport 폴더 내 index.ts 파일이 실행됨을 의미한다. 또한 윗글로 잠시 올라가 app.ts를 다시 확인해보면 `passportConfig()` 함수 실행 이후에 `passport.initialize()` 메서드가 실행되는 것을 볼 수 있다. 이는 index.ts에서 passport 사용에 필요한 여러 설정을 미리 마친 뒤 init되고 있다는 것을 알 수 있다.

그럼 index.ts가 무엇인지 살펴보자.

```typescript
// ./passport/index.ts
import passport from 'passport';
import local from './localStrategy';
import { Client } from '../db';

export = () => {
  // req.login() 함수 실행 시 serializeUser가 실행됨
  passport.serializeUser((user, done) => {
    // req.session객체에 어떤 데이터를 저장할 지 선택한다.
    const client = user as Client;
    done(null, client.userId); // deserializeUser로 이동
  });

  // serializeUser()가 done하거나 passport.session()을 실행 시
  passport.deserializeUser((userId: number, done) => {
    Client.findOne({ where: { userId } })
      .then((user) => done(null, user)) // req.login으로 돌아가 유저 정보를 다음 미들웨어에 전달
      .catch((err) => done(err)); // 에러가 있으면 다시 req.login으로 이동(콜백)
  });
  local();
};
```

passport의 serializeUser와 deserializeUser, 그리고 localStrategy.ts 파일에 해당하는 local()가 `passportConfig()`에 의해 사전에 실행되었음을 알 수 있다.

> 즉 로그인 인증을 위한 passport를 사용하기 위해서는 passport initialize를 하기 전에 필수적으로 인증 전략과 serializeUser, deserializeUser 메서드를 어떻게 활용할 것인지에 대한 정의를 사전에 해야 함을 의미한다.

계속해서 req.login() 함수가 serializeUser 메서드로 이어진다는 점에 주목해보자.

serialize는 직렬화를 의미하며 이는 세션에 데이터를 저장하기 위해 비트 또는 문자열을 일렬로 나열시키는 것을 의미한다. 굳이 데이터를 직접적으로 저장하기 않고 '직렬화' 과정을 거치는 이유는 아무래도 저장 효율성 때문이다.

login 함수를 통해 user정보를 전달받은 serializeUser 함수는 done 콜백 함수를 통해 세션에 저장할 데이터를 지정할 수 있는데 위 코드에서는 유저 정보 중 userId를 세션에 저장하는 것을 볼 수 있다. done 콜백 함수는 `authController.ts` 파일의 `return req.login(user, (loginError)` 함수의 (loginError) 부분에 해당한다.

```typescript
return req.login(user, (loginError) => {
    // deserializeUser 함수의 done이 실행되면
    // done(err) 발생 시
    if (loginError) {
      throw loginError;
    }
    // 세션에 사용자 정보를 저장하여 로그인 상태가 된다.
    return res.send({ message: '로그인 완료' });
  });
})(req, res, next);
```

login의 done 콜백 함수의 파라미터명을 loginError로 지정한 이유는 이전에 봤던 done 함수의 첫번째 파라미터, 즉 에러값에 대한 처리를 지정하는 의미에서 붙여진 이름이며, 해당 에러는 로그인 정보와 관련된 세션 저장 과정에서 에러가 발생할 경우 발동된다.

정상적으로 세션에 정보가 잘 저장되었다면 '로그인 완료'를 응답한다.

### STEP 4. 세션을 통해 로그인 유무 확인하기

```typescript
import { Request, Response, NextFunction } from 'express';

const isLoggedIn = (req: Request, res: Response, next: NextFunction) => {
  if (req.isAuthenticated()) {
    next();
  } else {
    res.status(403).send({ message: '로그인 필요' });
  }
};

const isNotLoggedIn = (req: Request, res: Response, next: NextFunction) => {
  if (!req.isAuthenticated()) {
    next();
  } else {
    res.status(403).send({ message: '이미 로그인 되어 있음' });
  }
};

export { isLoggedIn, isNotLoggedIn };
```

위 코드는 로그인 유무를 확인하기 위한 코드로 API 요청 router에서 미들웨어로 추가하여 활용이 가능하다.  
isAuthenticated() 함수 역시 passport.initialize()를 통해 생성되는 함수로서 세션에 저장된 로그인 유무에 대한 기록을 확인해준다. 보시다시피 로그인 되어 있다면 true, 아니라면 false를 리턴한다.

```typescript
// 프로필 조회 (유저 정보 조회)
router.get('/users/:userId', isLoggedIn, clientController.findUser);
```

이와 같이 활용이 가능할 것이다.

### STEP 5. 유저 정보 가져오기

로그인을 완료했다면 앞으로 세션에 저장된 유저 정보를 활용해야할 일이 많을 것이다. passport에서는 `passport.deserializeUser` 함수에 정의된 대로 작동한다.

```typescript
// ./passport/index.ts
// serializeUser()가 done하거나 passport.session()을 실행 시
passport.deserializeUser((userId: number, done) => {
  Client.findOne({ where: { userId } })
    .then((user) => done(null, user)) // req.login으로 돌아가 유저 정보를 다음 미들웨어에 전달
    .catch((err) => done(err)); // 에러가 있으면 다시 req.login으로 이동(콜백)
});
```

이전 코드에서도 확인했겠지만 deserializeUser 함수를 통해 세션에 직렬화로 저장된 유저 정보(여기서는 유저 id)를 복호화 과정을 거쳐 다시 활용가능한 데이터 형태로 복원하며, 이를 기반으로 유저 전체 정보를 DB에 조회한 결과를 리턴한다.  
해당 함수는 로그인 후 `req.user` 프로퍼티를 수행했을 경우 작동한다. 그러므로 완전한 유저 정보가 세션에 저장되는 형태가 아니며 필요할 때 마다 직렬화된 userId정보를 세션에서 꺼내 복호화하여 DB에 조회한 결과를 출력한다고 볼 수 있다.  
이러한 로직은 유저 정보가 쉽게 탈취되는 것을 방어해 준다. 하지만 유저 정보를 필요로 하는 모든 요청 마다 복호화 -> DB 조회 -> 리턴 과정을 거쳐야 하므로 처리 과정이 길다는 느낌은 든다.

## 정리

passport의 과정을 간단하게 정리하자면 아래와 같다.

> `PASSPORT LOGIN FLOW`  
> -> 로그인 요청  
> -> 로그인 유무 확인(isAutheticated)  
> -> authController의 login 함수 프로퍼티 실행  
> -> passportLocal.Strategy가 설정된 파일(localStrategy.ts)로 이동  
> -> 인증결과를 done 콜백함수에 전달  
> -> 성공 시 req.login()으로 인해 serializeUser 함수가 있는 곳(index.ts)으로 유저 데이터 전달  
> -> 세션에 저장할 데이터를 선택, 직렬화하여 저장(userId)  
> -> req.user 프로퍼티 사용 시 deserializeUser로 이동해서 세션에 저장된 userId기반 유저 정보 조회 및 결과 리턴

위 과정은 local 인증에 대한 과정이며 이 뿐 아니라 카카오 또는 네이버 인증 strategy를 세울 수 있도록 패키지가 제공되고 있으므로 추가적으로 서드파티 로그인 API를 적용해보는 것도 좋을 것 같다.
