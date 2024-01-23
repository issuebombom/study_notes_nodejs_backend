# Nest.js passport로 카카오, 구글 로그인 인증 구현하기

passport를 통해 카카오, 구글 계정으로 로그인할 수 있도록 구현하면 유저 입장에서는 회원가입 절차 없이 바로 로그인 가능하기 때문에 편의성을 제공해 줄 수 있게 되고, 무엇보다 소셜 로그인 정보를 기반으로 유저에게 특별한 회원가입 절차 요청 없이 회원 가입 처리가 가능해진다.

아래 링크에 접속하면 카카오에서 로그인 인증을 어떤 과정으로 처리하는지에 대한 설명이 담겨 있으니 관심있으면 참고해보자.

[카카오 로그인 처리 Flow](https://developers.kakao.com/docs/latest/ko/kakaologin/common)

간략하게 소셜 로그인 인증과정을 정리하자면 아래와 같다.

1. 사용자가 사이트(이하 프론트)에서 카카오 로그인 버튼을 클릭합니다.
2. 백엔드는 카카오 인증 서버에게 인가 코드 발급을 요청합니다.
3. 카카오 인증 서버에서 사용자에게 로그인을 통한 인증을 요청합니다.
4. 사용자 인증 성공 시, 개발자 앱에서 사전에 설정한 개인정보 동의 항목을 바탕으로 사용자에게 동의 화면을 보여줍니다. ('이런저런 데이터를 제공하게 될거다.' 라는 내용)
5. 사용자가 필수 동의 항목에 동의하고 로그인에 성공하면, 카카오 인증 서버는 인가 코드를 발급해 개발자 앱에 등록된 Redirect URI로 전달합니다.
6. 백엔드는 전달받은 인가 코드로 토큰과 유저 프로파일 정보를 받습니다.
7. 발급받은 토큰을 활용하면 이후 소셜 로그인 유저가 동의한 항목에 대한 추가 개인정보를 획득할 수 있게 됩니다.

우리는 위 과정에서 6번까지만 진행해볼 계획이다. 6번에서 받는 유저 프로파일 정보에는 이메일, 소셜ID, 이름 등이 담겨있는데 본 프로젝트에서는 닉네임, 이메일, 패스워드 정보로 회원 가입을 진행할 수 있다.

먼저 카카오 로그인을 구현해보자. 이를 마무리하면 구글 로그인은 매우 간단하다.

참고로 카카오 개발자 페이지, 구글 클라우드 페이지 등에서 로그인 API 구현을 위한 애플리케이션 등록 과정은 생략한다.

먼저 카카오 로그인 기능 구현을 위해 아래 패키지 설치가 필요하다.

```zsh
yarn add @nestjs/passport passport passport-kakao @types/passport-kakao
```

## 카카오 로그인 버튼을 클릭했을 때

Nest.js에서는 passport가 상당히 추상적이면서도 편리하게 기능을 제공하고 있는데, 백엔드와 카카오와의 소통을 PassportStrategy라는 기능과 AuthGuard 데코레이터를 통해 구현한다.

```typescript
// social-kakao-strategy.ts

import { PassportStrategy } from '@nestjs/passport';
import { Profile, Strategy } from 'passport-kakao';

export class KakaoStrategy extends PassportStrategy(Strategy, 'kakao') {
  constructor() {
    super({
      clientID: process.env.KAKAO_CLIENT_ID,
      clientSecret: process.env.KAKAO_CLIENT_SECRET,
      callbackURL: process.env.KAKAO_CALLBACK_URL,
      scope: ['account_email', 'profile_nickname'],
    });
  }

  async validate(accessToken: string, refreshToken: string, profile: Profile) {
    // 토큰과 프로파일을 한번 확인해보자
    console.log('accessToken: ', accessToken);
    console.log('refreshToken: ', refreshToken);
    console.log(profile);

    return {
      email: profile._json.kakao_account.email,
      password: String(profile.id),
      nickname: profile.displayName,
    };
  }
}
```

일명 카카오 전략 클래스 하나만으로 복잡한 과정이 상당 부분 처리된다.

먼저 constructor의 super를 통해 카카오 개발자 페이지에서 ID, Secret, 그리고 callback 경로를 지정해준다. 카카오 콜백 URL에 대해서는 이후 설명하도록 한다.

validate 함수는 이름을 바꿔서는 안된다. 이 함수는 passport에서 내부적으로 실행하게 되는 콜백 함수로 유저가 소셜 로그인을 완료하면 그에 따른 정보를 받아볼 수 있다.

`validate` 함수의 리턴값을 보면 대략적으로 profile에 어떤 정보가 들어있을 지 예상이 될 것이다. 우리는 획득한 정보를 회원가입에 활용할 것이다. 특히 profile.id를 password로 활용하는데 이는 사실 안전한 방법은 아니지만 저렇게 활용될 수 있다는 점을 참고하자.

위에서 나열한 로그인 Flow 중 2번에서 6번까지의 과정이 이 코드 안에서 모두 작동한다. Strategy 클래스와 그 내부인 passport-kakao 패키지에서 내부적으로 처리한다.

```typescript
// auth.guard.ts

import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class KakaoAuthGuard extends AuthGuard('kakao') {}
```

위 Strategy를 미들웨어처럼 활용하기 위해 Nest.js에서는 AuthGuard를 활용하도록 프레임이 짜여져 있다. strategy 파일에서 적용한 'kakao' 네이밍을 그대로 가져와야 한다.

```typescript
// auth.module.ts

import { Module } from '@nestjs/common';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';
import { JwtModule } from '@nestjs/jwt';
import { UsersModule } from '../users/users.module';
import { KakaoStrategy } from './strategies/social-kakao.strategy';

@Module({
  imports: [
    JwtModule.register({}),
    UsersModule, //
  ],
  controllers: [AuthController],
  providers: [AuthService, KakaoStrategy],
})
export class AuthModule {}
```

이제 KakaoStrategy를 AuthGuard로 사용하기 위해서는 모듈 providers에 반드시 등록해 줘야하는 것을 잊지 말자

## 유저가 소셜 로그인에 성공했다면

```typescript
// auth.controller.ts

@Controller('api/auth')
export class AuthController {
  constructor(
    private readonly authService: AuthService //
  ) {}

  @UseGuards(KakaoAuthGuard)
  @Get('login/kakao')
  async kakaoCallback(
    @SocialUser() socialUser: SocialUserAfterAuth,
    @Res({ passthrough: true }) res: Response
  ): Promise<void> {
    const { accessToken, refreshToken } = await this.authService.OAuthLogin({
      socialLoginDto: socialUser,
    });

    res.cookie('refreshToken', refreshToken);
    res.cookie('accessToken', accessToken);

    res.redirect('/');
  }
}
```

컨트롤러에서 kakao 로그인에 관한 엔드포인트를 설정한다.  
프론트에서 a 태그의 href 또는 이벤트리스너로 '카카오 로그인' 버튼을 클릭하면 `/api/auth/login/kakao` 경로로 이동하도록 처리할 것이다. 여기서 만약 fetch 함수를 사용하게 된다면 CORS 이슈가 발생하게 되므로 '요청'이 아닌 '이동' 이 되도록 프론트에서 처리해야 할 것이다.

어쨌든 위 코드에서 UseGuard를 통해 Strategy가 미들웨어 성격으로 적용되는 것을 확인할 수 있다.

여기서 한번 짚고 넘어가야할 점은 상단의 카카오 로그인 Flow의 5번 과정이다. 5번 과정에서는 `개발자 앱에 등록된 Redirect URI로 전달` 한다는 말이 있는데 이게 개발자 페이지에서 어떻게 설정했는지 확인할 필요가 있다. Nest.js에서는 kakao 로그인을 위한 엔드포인트 경로인 `/api/auth/login/kakao`를 Redirect URI에 그대로 적용하면 된다.

원래는 사용자에게 소셜 로그인을 요청하는 엔드포인트와, 사용자가 로그인에 성공하면 백엔드에게 인가 코드를 발급해 주게 되는데 이 인가 코드를 받을 엔드포인트 이렇게 두 개의 엔드포인트가 구성되어야 한다.

하지만 우리의 kakao strategy는 이 모두를 처리할 수 있도록 구성되어 있기 때문에 요청도 인가 코드에 대한 처리도 모두 한 곳에서 가능하다. 아마도 strategy에 인가 코드가 전달되지 않는다면 로그인 요청 페이지를 띄워주고, 인가 코드를 받으면 이를 처리하도록 구성되어 있을 것으로 예상한다.

그래서 UseGuard 데코레이터의 결과로 무엇을 가져올 수 있는지 확인해보자. 위 코드에서는 @SocialUser 라는 커스텀 데코레이터를 직접 만들어서 사용하고 있지만 사실 `@Req() req: Request` 데코레이터로 받아서 req.user로 접근하여 이메일, 닉네임, 패스워드를 가져올 수도 있다. 하지만 이는 typescript에서 타입 지정하기에 다소 번거로운 면이 있어 @SocialUser라는 커스텀 데코레이터를 사용하여 req.user로 추가 접근하는 과정을 생략하고자 했다.

커스텀 데코레이터에 관심이 간다면 아래 코드를 참조하자

```typescript
// user.decorator.ts

import { ExecutionContext, createParamDecorator } from '@nestjs/common';

export const SocialUser = createParamDecorator((data: unknown, ctx: ExecutionContext) => {
  const request = ctx.switchToHttp().getRequest();
  return request.user;
});

export interface SocialUserAfterAuth {
  email: string;
  password: string;
  nickname: string;
}
```

이제 서비스의 `OAuthLogin` 함수를 보기 전에 그 리턴값으로 accessToken과 refreshToken을 쿠키로 발급받는 것을 확인할 수 있을 것이다. 이를 통해 이후 로그인 유저의 각종 인가 처리에 쿠키가 사용될 것임을 예상할 수 있다.

```typescript
// auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private readonly usersService: UsersService //
  ) {}

  async OAuthLogin({ socialLoginDto }: IAuthServiceSocialLogin) {
    const { email } = socialLoginDto;
    let user = await this.usersService.findByEmail({ email });

    if (!user)
      user = await this.usersService.createUser({
        createUserDto: socialLoginDto,
      });

    const accessToken = this.getAccessToken({ userId: user.id });
    const refreshToken = this.getRefreshToken({ userId: user.id });

    return { accessToken, refreshToken };
  }
}
```

`OAuthLogin` 함수 로직은 간단하다. 이메일 존재 유무 확인, 없으면 회원가입 후 토큰 발행, 있으면 즉시 토큰 발행한다.

이렇게 카카오 로그인을 통한 인증 로직을 구현해 보았다.

## 구글 로그인 살펴보기

구글 로그인도 카카오와 매우매우 흡사하다. 카카오 로그인 Flow와 별 차이가 없으며 회사가 다르기 때문에 발생하는 차이(?) 정도만 수정해주면 된다.

그렇기 때문에 카카오와 다른 부분 위주로 코드를 살펴보도록 하자

```zsh
# passport 관련 패키지가 이미 설치된 것을 전제로 함
yarn add passport-google-oauth20 @types/passport-google-oauth20
```

```typescript
// social-google.strategy.ts

export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
  constructor() {
    super({
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: process.env.GOOGLE_CALLBACK_URL,
      scope: ['email', 'profile'], // 카카오와 다르다.
    });
  }

  async validate(accessToken: string, refreshToken: string, profile: Profile) {
    console.log('accessToken: ', accessToken);
    console.log('refreshToken: ', refreshToken);
    console.log(profile);

    // 프로파일 접근 방법도 다소 차이가 있다.
    return {
      email: profile.emails[0].value,
      password: profile.id, // 구글은 id가 string이다.
      nickname: profile.displayName,
    };
  }
}
```

그 외추가적으로 다른점은 AuthGuard 등록 시 'google' 과 같은 키워드를 써서 등록 한다는 점, 컨트롤러에서 엔드포인트 경로를 'login/google' 과 같은 형태로 바꿔주는 점 외에는 그대로 코드를 재활용하면 된다. 위 글을 자세히 읽어왔다면 구글 로그인 구현도 매우 금방 끝낼 수 있을 것이다.
