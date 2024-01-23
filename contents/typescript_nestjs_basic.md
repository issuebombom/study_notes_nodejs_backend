# Nest.js 입문하기

## What is Nest.js?

이전에 사용한 express는 사실 프레임워크라고 부르기 민망할 정도로 너무 제약 사항이 없는 편이었다. 프레임워크의 대표적인 특징은 프로젝트 수행에 있어서 필요한 초기설정, 아키텍처 틀이 정해져 있다는 점인데, express는 자잘한 기능까지도 하나부터 열까지 구현해야 한다는 점이 불편한 점이다. 자유도가 높다고도 볼 수 있겠지만 그만큼 오류를 범할 가능성도 높아지는 것이다.  
Nest나 Django와 같은 프레임워크는 API 요청, 응답 방법이라든지, 아키텍처 구성 방식이라든지 이미 강제로 정해진 부분이 있어 이를 지키지 않을 수 없도록 구현되어있다. 그렇게 약속되어 있기 때문에 프로젝트를 생성하면 반드시 필요한 기능은 자동으로 파일 생성 및 코드 작성까지 미리 해준다. express가 제빵을 비교했을 때 밀가루와 물 정도만 던져주는 형태라면 Nest.js는 반죽할 그릇, 반죽 모양 틀, 반죽도구 등을 제공해주는 느낌이다. 그렇기 때문에 반죽을 어떤 그릇에 할지, 어떤 도구로 반죽을 평평하게 펼칠 지 등을 고민하는 시간을 절약할 수 있다.

결론적으로 아래와 같은 이점을 얻을 수 있다.

- 생각할 거리가 줄어들어 비지니스 로직의 퀄리티가 향상될 가능성이 커집니다!(진짜 집중해야할 기능에 집중할 수 있다.)
- 자잘한 기능(유효성 검증을 위한 코드 작성 등) 직접 손대지 않아도 되므로 코드 생산성이 향상된다는 얘기에요!(빨리빨리 할 수 있다는 의미)

## Nest.js 실습하기

```zsh
# 설치
npm i -g @nestjs/cli
```

```
# 프로젝트 생성
nest new 프로젝트명
# -> npm으로 설치
```

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

앱 컨트롤러를 하나의 클래스로 정의해서 export한다.  
constructor에 활용할 앱 서비스를 장착한다.  
데코레이터 함수로 API 메서드를 정의한다.

### 테스트 코드 생성

jest라는 테스트 프레임워크를 활용하여 테스트를 한다.
app.controller.spec.ts 파일에 테스트용 빌드가 짜여져 있다.

### IoC와 DI

#### Inversion of Control

기존 express방식에서는 컨트롤러 구조를 짜는 과정을 살펴보면

1. 컨트롤러 파일 내부에 필요한 서비스 파일을 import했고,
2. 컨트롤러 클래스 내부에서 서비스의 타입 선언을 한 뒤,
3. constructor에서 서비스 인스턴스를 직접 생성해야 했다.
   위 과정은 필요한 서비스를 추가, 제거할 때 마다 고려해줘야 하는 방식이며, 적여야 할 부분이 많은 만큼 수정 시 휴먼 에러 발생 확률도 높아진다는 것을 예상할 수 있다.

하지만 Nestjs에서는 사용하고자 하는 객체(위 예시에서는 서비스)를 일일이 직접 생성하고 관리하는 것이 아니라 해당 권한을 Nest.js IoC 컨테이너에 위임한다. 즉 객체 제어 부분에 대해서 신경쓰지 않아도 된다. 이를 IoC에 위임했다고 해서 제어 역전(Inversion of Control)이라고 부른다.

```typescript
constructor(private readonly appService: AppService) {}
```

위 코드를 보는 것과 같이 필요한 서비스 import는 저 한 줄 만으로 충분하다.  
이게 가능한 이유는 우선 AppService라는 파일 자체를 Nest 프레임워크에서 자체 관리하기 때문이다. (제어권을 Nest에게 위임했기 때문)

#### Dependency Injection

의존성 주입의 줄임말인 DI는 말 그대로 Controller에 Service 객체를 주입하는 행위를 말한다. Controller 구조에 Service를 import하는 순간 Controller는 Service에 `의존`하게 된다. 이렇게 의존성 구조를 형성하게 되므로 `주입`이라는 단어를 채택한 것 같다.

### Nest.js 입문하기

#### 게시판 기능 구현 예시 살펴보기(without DB)

먼저 board(게시판)와 관련된 아키텍처를 구성하기 위해 기본적으로 module과 controller 파일을 생성한다. 생성 방식은 프레임워크의 원칙에 따라 자동 생성되도록 하기 위해 cli명령어로 수행한다.

```zsh
# src 폴더 내에서 커멘드 실행합니다. (typescript니까)

# 글로벌, 모듈, 생성객체명
nest g mo board
# -> board 폴더 및 board.module.ts 파일 생성됨

# 글로벌, 컨트롤러, 생성객체명
nest g co board
# -> board.controller.ts 파일이 생성됨
# -> board.module.ts 내부 코드에 boardController 클래스가 자동 추가된다.
```

위 진행 순서를 봤을 때 앞으로 생성될 여러 컨트롤러가 모듈에 import 되어 최종적으로 board의 기능을 모아둔 하나의 '모듈'로서 활용이 가능하도록 하는 구조로 짜여지는 것 같다.

```typescript
// board.controller.ts
import { Controller } from '@nestjs/common';

@Controller('board') // routing path
export class BoardController {}
```

컨트롤러를 생성하면 위 코드처럼 컨트롤러 데코레이터에 'board'라는 routing path까지 자동으로 잡아주는 것을 확인할 수 있다.

```zsh
# 글로벌, 서비스, 생성객체명
nest g s board
# -> board.service.ts 파일이 생성됨
```

서비스를 생성함과 동시에 모듈에도 탑재되는 것을 확인할 수 있다.  
하지만 아쉬운 점은 Controller에 service가 자동 탑재되지 않는다는 부분이다.(수동 입력 필요)

```typescript
import { module } from '@nestjs/common';
import { BoardController } from './board.controller';
import { BoardService } from './board.service';

@module({
  controllers: [BoardController],
  providers: [BoardService],
})
export class BoardModule {}
```

module에서 컨트롤러는 컨트롤러, 서비스는 프로바이더에 자동 탑재된다.

이제 최종적으로 board.module.ts는 초기 Nest프로젝트 생성 시 만들어졌던 app.module.ts로 전달되어야 할 것이다. 이 부분도 아래와 같이 이미 자동 연결 되어 있음을 확인할 수 있다.

```typescript
@module({
  imports: [BoardModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### 본격 들어가기 전 lodash 패키지 살펴보기

해당 패키지는 자바스크립트 자체로 구현하기에는 다소 번거로운 여러 연산을 간단하게 수행할 수 있도록 도와주는 유틸리티이다.

가령 ['app', 'ball', 'cat']이라는 배열에 든 각각의 string length를 확인하고 싶다고 했을 때 일반적으로 iteration 코드와 length를 찍어 일일이 세어보는 코드를 작성할 수 있겠지만 아래와 같이 작성하고 답을 얻을 수 있다.

```javascript
_.groupBy(['app', 'ball', 'cat'], 'length');
// -> { '3': ['app', 'cat'], '4': ['ball'] }
```

```zsh
# lodash 설치
npm i lodash
```

> 🚨 commonjs 라이브러리 사용을 위해서는 tsconfig.json에서 "esMouleInterop": true 설정을 잊지 말자

### 기본 CRUD 틀 작성하기

```typescript
// board.controller.ts
import { Controller, Delete, Get, Post, Put } from '@nestjs/common';
import { BoardService } from './board.service';

@Controller('board')
export class BoardController {
  constructor(private readonly boardService: BoardService) {}

  @Get('/articles')
  getArticles() {
    return this.boardService.getArticles();
  }

  @Get('/articles/:id')
  getArticleById() {
    return this.boardService.getArticleById(id);
  }

  @Post('/articles')
  createArticle() {
    return this.boardService.createArticle();
  }

  @Put('/articles/:id')
  updateArticle() {
    return this.boardService.updateArticle(id);
  }

  @Delete('/articles/:id')
  deleteArticle() {
    return this.boardService.deleteArticle(id);
  }
}
```

Controller 데코레이터에 이어 GET, POST와 같은 API 메서드의 데코레이터에도 routing path가 입력되어 있는 것을 확인할 수 있다. 기존 express에서 router가 하던 일을 컨트롤러에서 한꺼번에 하고 있는 것을 확인할 수 있다.

또한 위 코드는 아직 데이터를 service에 전달하는 부분이 구현되어 있지 않다. express 환경에서는 API 요청 시 함께 전달해줘야 할 데이터(가령 id, 회원가입 정보 등)를 body parser 등을 통해 가져왔어야 하는데 해당 과정이 생략되어 있다. 그 이유는 Nest에서는 `DTO`라는 개념이 도입되어 데이터 전송과 관련된 코드를 따로 관리하기 때문이다. 그러므로 DTO 파일을 먼저 작성한 뒤 컨트롤러에서 불러오는 형태로 구현될 예정이다.  
먼저 필요한 추가 패키지를 설치한 뒤 DTO를 어떻게 구성하는지 살펴보자

### Nest의 편의성을 강화시켜줄 패키지 설치

#### 손쉬운 타입 선언을 위한 패키지

```zsh
npm i @nestjs/mapped-types
```

#### Nest에서 유효성 검사, 유연한 타입 변환을 위해 필요한 패키지

```zsh
npm i class-validator class-transformer
# -> class-validator는 @nestjs/mapped-types의 의존성을 지니고 있어 반드시 설치 순서를 지켜줘야 한다.
```

### DTO 작성

dto는 API 요청 시 전달받을 데이터를 따로 정의하고 타입을 선언, 유효성을 검증하는 역할을 한다. 해당 파일은 CLI 명령으로 자동 생성되는 것이 아니므로 직접 파일 생성 및 수정이 필요하다.

```typescript
// create-article.dto.ts
import { IsNumber, IsString } from 'class-validator';

export class CreateArticleDto {
  @IsString()
  readonly author: string;

  @IsString()
  readonly title: string;

  @IsString()
  readonly content: string;

  @IsNumber()
  readonly password: number;
}
```

IsString과 IsNumber와 같은 데코레이터 적용을 통해 유효성 검증을 매우 간단하게 구현할 수 있다.  
또한 추가적으로 유효성 검사를 패키지를 적용하기 위해서는 main.ts에서도 아래와 같은 설정이 필요하다.

```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({ transform: true })); // 이 코드
  await app.listen(3000);
}
bootstrap();
```

ValidationPipe의 옵션으로 transform: true를 설정하면 API 요청 시 number 형태의 string은 알아서 number로 변환해주는 역할을 한다. 가령 param으로 userId를 받게 되면 기본적으로 string으로 받아지기 때문에 유효성 검증에서 탈락하게 되는데 이를 유연하게 처리해주라는 의미가 된다.

위와 같이 DTO 파일을 기본적으로 구성할 수 있다. 해당 예시는 게시글 create에 필요한 요소를 간단하게 구현한 예시인데, 향후 수정이나 삭제에 대한 DTO를 생성할 때도 위와 비슷한 형태의 코드가 될 것으로 보인다. 즉 `복붙의 기운이 느껴진다.`

이러한 복붙이 아닌 좀 더 편리하게 DTO를 구성할 수 있게 돕는 패키지가 바로 `@nestjs/mapped-types`에 해당한다.

이를 설치했다면 아래와 같이 게시글 수정 DTO를 구성할 수 있다.

```typescript
// update-article.dto.ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateArticleDto } from './create-article.dto';

export class UpdateArticleDto extends PartialType(CreateArticleDto) {}
```

위 코드에서는 create-article.dto에 있는 클래스를 가지고 와서 `PartialType` 기능을 통해 코드를 간소화 시킨 것을 확인할 수 있다.  
이는 `게시글 "생성"에 필요한 요소와 게시글 "업데이트" 시 필요한 요소가 같기 때문에 적용할 수 있는 명령이다.` PartialType은 전체 타입 중 일부를 참조하겠다는 의미이다.

```typescript
// delete-article.dto.ts
import { PickType } from '@nestjs/mapped-types';
import { CreateArticleDto } from './create-article.dto';

export class DeleteArticleDto extends PickType(CreateArticleDto, ['password'] as const) {}
```

게시글 삭제 기능의 경우 필요한 요소는 password이다. 이는 게시글 생성에 필요한 요소 중 하나인데 PickType 기능을 사용하면 그 중 하나를 집어서 불러오는 기능을 뜻한다.

### 컨트롤러 작성하기(요소 받기)

이어서 컨트롤러에서 요청 시 body와 params와 같은 요소를 받아오는 코드를 작성합니다.

```typescript
// board.controller.ts
import { Body, Controller, Delete, Get, Param, Post, Put } from '@nestjs/common';
import { BoardService } from './board.service';
import { CreateArticleDto } from './dto/create-article.dto';
import { DeleteArticleDto } from './dto/delete-article.dto';
import { UpdateArticleDto } from './dto/update-article.dto';

@Controller('board')
export class BoardController {
  constructor(private readonly boardService: BoardService) {}

  @Get('/articles')
  getArticles() {
    return this.boardService.getArticles();
  }

  @Get('/articles/:id')
  getArticleById(@Param('id') articleId: number) {
    return this.boardService.getArticleById(articleId);
  }

  @Post('/articles')
  createArticle(@Body() data: CreateArticleDto) {
    return this.boardService.createArticle(data.title, data.content, data.password);
  }

  @Put('/articles/:id')
  updateArticle(@Param('id') articleId: number, @Body() data: UpdateArticleDto) {
    return this.boardService.updateArticle(articleId, data.title, data.content, data.password);
  }

  @Delete('/articles/:id')
  deleteArticle(@Param('id') articleId: number, @Body() data: DeleteArticleDto) {
    return this.boardService.deleteArticle(articleId, data.password);
  }
}
```

컨트롤러의 세부 함수의 파라미터로 @Body, @Param 데코레이터와 해당하는 DTO파일을 적용하는 방식으로 필요한 요소를 받아오는 것을 확인할 수 있다.

### 서비스 작성하기

현재 TypeORM을 적용하지 않고 단순히 배열에 데이터를 저장하고 불러온다는 개념으로 간단하게 구현해보자

```typescript
// board.service.ts
import {
  Injectable,
  NotFoundException,
  UnauthorizedException,
} from '@nestjs/common';
import _ from 'lodash';

@Injectable()
export class BoardService {
  private articles = [];

  // 게시글 비밀번호를 저장하기 위한 Map 객체입니다.
  private articlePasswords = new Map();

  getArticles() {
    return this.articles;
  }

  getArticleById(id: number) {
    return this.articles.find((article) => return article.id === id);
  }

  createArticle(title: string, content: string, password: number) {
    const articleId = this.articles.length + 1;
    this.articles.push({ id: articleId, title, content });
    this.articlePasswords.set(articleId, password);
    return articleId;
  }

  updateArticle(id: number, title: string, content: string, password: number) {
    if (this.articlePasswords.get(id) !== password) {
      throw new UnauthorizedException(
        `Article password is not correct. id: ${id}`,
      );
    }

    const article = this.getArticleById(id);
    if (_.isNil(article)) {
      throw new NotFoundException(`Article not found. id: ${id}`);
    }

    article.title = title;
    article.content = content;
  }

  deleteArticle(id: number, password: number) {
    if (this.articlePasswords.get(id) !== password) {
      throw new UnauthorizedException(
        `Article password is not correct. id: ${id}`,
      );
    }

    this.articles = this.articles.filter((article) => article.id !== id);
  }
}
```

위 코드에서 주목해서 봐야할 점은 아래와 같다.

1. 당연하겠지만 typescript 기반이니 서비스 함수마다 파라미터 타입 정의가 필요하다.

```typescript
createArticle(title: string, content: string, password: number) { // ...
```

2. 예외처리를 status 코드 챙겨서 적용하기 보다는 아래와 같이 해당 클래스를 불러오자

```typescript
throw new UnauthorizedException(`Article password is not correct. id: ${id}`);
throw new NotFoundException(`Article not found. id: ${id}`);
```

3. lodash를 활용하여 null값에 대한 예외처리 시 사용하고 있다.

```typescript
if (_.isNil(article)) {
  // ...
}
```

여기까지 DB없이 Nest.js로 간단한 게시글 CRUD 예시입니다.

---

## TypeORM

### What is TypeORM
MySQL, MariaDB, PostgreSQL, MongoDB(experimental: 아직 실험단계임이 명시되어 있음) 등 여러 종류의 데이터베이스를 동일한 방식으로 다룰 수 있도록 돕는 도구를 ORM이라 한다. 거의 비슷하지만 조금씩 다른 SQL들은 쿼리문에도 조금씩에 차이가 있을 수 있는데 그런 부분에서의 걱정을 덜어준다. 

[Documentation](https://typeorm.io/)  

### TypeORM 기본 설치
```zsh
# 설치
npm i @nestjs/typeorm typeorm mysql
npm i @nestjs/config

```

```zsh
# 설치 이슈 발생한다면 class-validator 설치 순서 체크
npm uninstall class-validator
npm i @nestjs/typeorm typeorm mysql
npm i @nestjs/config
npm i class-validator
```

### DB 연결을 위한 초기 설정

먼저 데이터베이스 접속에 필요한 정보를 .env에 보관한다.  

```zsh
# .env
DATABASE_HOST="localhost"
DATABASE_PORT=3306
DATABASE_USERNAME="DB 접속 아이디"
DATABASE_PASSWORD="DB 접속 비밀번호"
DATABASE_NAME="board" # DB 이름
DATABASE_SYNCHRONIZE=true # 해당 부분은 배포 시 false로 변경
```

이후 root module에 DB사용을 위한 설정을 한다.  

```typescript
// app.module.ts
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { BoardModule } from "./board/board.module";
import { TypeOrmConfigService } from "./config/typeorm.config.service";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { Article } from "./board/article.entity";

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }), // config값 읽어오기
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useClass: TypeOrmConfigService, // DB 접속 관련 정보가 모인 곳
      inject: [ConfigService],
    }),
    BoardModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

먼저 `config 폴더를 생성하여 그 안에 typeorm.config.service.ts 파일을 생성`한다. 해당 파일 안에 실제 DB가 참조해야할 정보가 담기도록 할 예정이다.  
TypeOrmModule은 TypeORM 연결을 위해 필요한 셋팅으로 볼 수 있다. imports와 inject 설정을 통해 config 즉 .env를 활용하기 위한 설정을 하고, useClass에 실제 DB 연결을 위한 정보를 담는다.

```typescript
// config/typeorm.config.service.ts
import { Injectable } from "@nestjs/common";
import { TypeOrmModuleOptions, TypeOrmOptionsFactory } from "@nestjs/typeorm";
import { ConfigService } from "@nestjs/config";
import { Article } from "src/board/article.entity";

@Injectable()
export class TypeOrmConfigService implements TypeOrmOptionsFactory {
  constructor(private readonly configService: ConfigService) {}

  createTypeOrmOptions(): TypeOrmModuleOptions {
    return {
      type: "mysql",
      host: this.configService.get<string>("DATABASE_HOST"),
      port: this.configService.get<number>("DATABASE_PORT"),
      username: this.configService.get<string>("DATABASE_USERNAME"),
      password: this.configService.get<string>("DATABASE_PASSWORD"),
      database: this.configService.get<string>("DATABASE_NAME"),
      entities: [__dirname + "/**/*.entity{.ts,.js}"],
      synchronize: this.configService.get<boolean>("DATABASE_SYNCHRONIZE"),
    };
  }
}
```

DB 연결에 필요한 정보를 createTypeOrmOptions 함수에 담아 리턴하는 것을 확인할 수 있다. 특히 .env에 기록한 DB 정보를 가져오도록 configService가 활용되고 있으며 초기에 .env 정보에 대한 타입도 지정하고 있다.  

마지막으로 레포지토리 사용을 위해 board 모듈에 아래와 같이 코드를 추가해준다.

```typescript
// board.module.ts
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { Article } from "./article.entity";
import { BoardController } from "./board.controller";
import { BoardService } from "./board.service";

@Module({
  // 매우 중요: 서비스에서 사용할 리포지토리 사용을 imports에 명시!
  imports: [TypeOrmModule.forFeature([Article])],
  controllers: [BoardController],
  providers: [BoardService],
})
export class BoardModule {}
```


### Entity 생성하기
DB의 테이블과 맵핑되는 개체(Entity)를 뜻한다. sequelize에서 model과 비슷한 역할을 한다. 이는 실제 DB와 레포지토리(DB에 쿼리를 날리기 위한 파일) 간 소통을 위한 설정 정도로 생각하자.  

```typescript
import {
  Column,
  CreateDateColumn,
  DeleteDateColumn,
  Entity,
  PrimaryGeneratedColumn,
  UpdateDateColumn,
} from "typeorm";

@Entity({ schema: "board", name: "articles" })
export class Article {
  @PrimaryGeneratedColumn({ type: "int", name: "id" })
  id: number;

  @Column("varchar", { length: 10 })
  author: string;

  @Column("varchar", { length: 50 })
  title: string;

  @Column("varchar", { length: 1000 })
  content: string;

  @Column("varchar", { select: false })
  password: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn()
  deletedAt: Date | null;
}
```

schema는 DB 이름을 적고, name은 테이블명으로 지정한다.  
데코레이터로 쉽게 테이블의 각 컬럼을 정의하고 있음을 알 수 있고, 간단하게 validation 요소도 추가한 것을 확인할 수 있다. 특히 신기한 점은 password 컬럼이 select 쿼리로의 접근을 막고 있음을 볼 수 있다. 아무래도 select 쿼리의 결과로 비밀번호를 노출시킬 필요가 없기 때문이다.  

### Repository 생성하기
Nest는 일반 레포지토리와 커스텀 레포지토리 이렇게 두 부류가 존재한다. 우선 기본적인 CRUD API는 일반 레포지토리 적용으로 충분하므로 일반 레포지토리를 기준으로 설명한다.  

일반 레포지토리(이하 레포지토리)는 따로 ts파일을 생성하여 코드를 관리하는 것이 아니라 연결할 service에 바로 코드를 작성할 수 있는데, 이는 TypeORM에서 기본적인 GET, POST, INSERT, DELETE와 같은 기본 메서드 구현을 위한 레포지토리를 이미 모듈로서 가지고 있기 때문이다. 그래서 이를 서비스에 injection만 하면 되는 방식으로 레포지토리를 생성 가능하다.  

참고로 커스텀 레포지토리는 좀 더 세부적인, 구체적인 쿼리문을 날려야할 때(일반 레포지토리에는 없는 쿼리문) 사용되며 이는 말그대로 커스텀이므로 레포지토리 파일을 생성해서 직접 작성해야 한다.

아래 코드를 살펴보자

```typescript
// board.service.ts
import {
  Injectable,
  NotFoundException,
  UnauthorizedException,
} from '@nestjs/common';
import _ from 'lodash';
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm";
import { Article } from "./article.entity";

@Injectable()
export class BoardService {
  // constructor에 주목
  constructor(
    @InjectRepository(Article) private articleRepository: Repository<Article>
  ) {}
```

BoardService 클래스의 constructor로 `InjectRepository` 데코레이터를 적용해서 Article Entity를 선택한 레포지토리를 `articleRepository`로 바로 생성하고 있다. 우리가 따로 레포지토리를 만든 적은 없지만 TypeORM에서 이미 모듈을 생성해 뒀기 때문이다.  

이제 articleRepository를 기본적인 CRUD에서 어떻게 활용하는지 확인해보자.  

```typescript
async getArticles() {
    return await this.articleRepository.find({
      where: { deletedAt: null },
      select: ["author", "title", "createdAt"],
    });
  }
```

게시글을 조회하는 레포지토리이다. constructor에서 생성한 articleRepository의 find 메서드를 적용하고 옵션으로 where 조건절과 attr(컬럼)을 선택적으로 가져오는 것을 확인할 수 있다. TypeORM에서 find는 기본적으로 배열 형태를 반환하며 즉 sequelize의 findAll과 같은 함수임을 기억하자.  
또한 Nest의 Repository에서도 호출 결과가 Promise 객체가 반환된다. 그러므로 `조회`와 같이 결과물을 받아야 되므로 async과 await가 필수적으로 붙어야 한다.  

```typescript
async getArticleById(id: number) {
    return await this.articleRepository.findOne({
      where: { id, deletedAt: null },
      select: ["author", "title", "content", "createdAt", "updatedAt"],
    });
  }
```

id를 기준으로 조회하는 레포지토리이다. find 메서드와 findOne 메서드의 차이로 사용법에 큰 차이가 없다.  

```typescript
createArticle(title: string, content: string, password: number) {
    this.articleRepository.insert({
      author,
      title,
      content,
      // 나중에 암호화된 비밀번호를 저장하도록 하겠습니다.
      password: password.toString(),
    });
  }

```
create는 `insert` 메서드로 적용되는 것을 확인할 수 있다. 입력받을 값을 오브젝트 형태로 전달하는 점은 동일하다. password의 경우 예시를 위해 적어뒀는데 `타입 변환이 필요한 상황이라면 .toString() 메서드를 붙이면 된다.`  
여기서 주목할 점은 async와 await를 쓰지 않았다는 점이다. 사실 데이터 생성의 경우 리턴값이 없어도 되므로 await가 필수적이지는 않다. 게다가 async를 쓰지 않는다면 다른 async 요청들이 그 요청들이 서는 줄을 서지 않아도 된다. 이 점은 트래픽을 줄이는 것에 도움이 될 수 있다. 물론 게시글 생성이 조회만큼 활발하게 요청이 발생하는 쿼리는 아니지만 말이다.

```typescript
async deleteArticle(id: number, password: number) {
    await this.checkPassword(id, password);
    this.articleRepository.softDelete(id); // soft delete를 시켜주는 것이 핵심!
  }

  private async checkPassword(id: number, password: number) {
    const article = await this.articleRepository.findOne({
      where: { id, deletedAt: null },
      select: ["password"],
    });
    if (_.isNil(article)) {
      throw new NotFoundException(`Article not found. id: ${id}`);
    }

    if (article.password !== password.toString()) {
      throw new UnauthorizedException(
        `Article password is not correct. id: ${id}`
      );
    }
  }
```

delete의 경우 게시글에 설정된 패스워드 조회가 우선적으로 필요하다. 위 코드에서는 패스워드 검증을 위한 로직을 checkPassword라는 헬퍼 함수로 따로 생성하여 관리하고 있음을 볼 수 있다. 만일 비밀번호 검증과 같이 자주 필요한 로직이 있다면 저렇게 헬퍼함수로 관리하는 것이 유지 보수에 안정적이다. 또한 비밀번호를 `조회` 해서 `대조`하는 로직이 보안 상 클래스 외부에서 접근이 불가해야 하므로 함수 앞에 `private`를 적용하고 있음을 볼 수 있다.

### Controller의 수정

Service에서는 Repository 사용으로 인해 async와 await를 사용하였고 해당 Service는 고스란히 Controller에 전달되므로 컨트롤러에서도 각 함수와 service 접근 함수에 async와 await를 붙여줘야 한다.

### 외부 패키지 사용 (jwt)

외부 라이브러리를 적용할 때는 항상 module.ts에 적용이 되어야 사용이 가능하다. jwt를 UserService, UserController에서 쓰인다고 가정한다면 user.module.ts의 imports를 통해 jwt가 import 되어야 할 것이다.  

먼저 Nest에서 사용할 jwt 패키지를 설치한다.

```zsh
npm i @nestjs/jwt
```

이후 jwt 사용을 위한 module 세팅을 한다.  

```typescript
import { Module } from "@nestjs/common";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { JwtModule, JwtService } from "@nestjs/jwt";
import { TypeOrmModule } from "@nestjs/typeorm";
import { Article } from "src/board/article.entity";
import { JwtConfigService } from "src/config/jwt.config.service";
import { Repository } from "typeorm";
import { User } from "./user.entity";
import { UserService } from "./user.service";
import { UserController } from "./user.controller";

@Module({
  imports: [
    TypeOrmModule.forFeature([User]), // 이건 TypeORM 강의 시간에 배웠죠?
    JwtModule.register({
      secret: 'secret',
      signOptions: { expiresIn: '3600s' }, // 토큰의 만료시간은 1시간
    }),
  ],
  providers: [UserService],
  exports: [UserService],
  controllers: [UserController],
})
export class UserModule {}
```

위 코드와 같이 imports에 직관적으로 jwt 사용을 위한 등록이 가능하다. 하지만 예상한 바와 같이 jwt의 secret키가 저렇게 쉽게 노출되어서는 안되므로 이 또한 config 패키지를 활용하여 숨겨줄 필요가 있다.  

잠시 config 사용 flow를 떠올려 보자면
- xxx.config.service파일을 생성하여 module에 injection하는 형태를 취한다.
- config.service 내부에서 .env의 변수를 읽어줄 수 있도록 조치한다.

```typescript
// config/jwt.config.service.ts
import { Injectable } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import { JwtModuleOptions, JwtOptionsFactory } from "@nestjs/jwt";

@Injectable()
export class JwtConfigService implements JwtOptionsFactory {
  constructor(private readonly configService: ConfigService) {}

  createJwtOptions(): JwtModuleOptions {
    return {
      secret: this.configService.get<string>("JWT_SECRET"),
      signOptions: { expiresIn: "3600s" },
    };
  }
}
```

config.service.ts를 상세히 살펴보면
- 옵션 설정을 위한 클래스 생성
- 클래스의 implements 설정, constructor의 configService 설정
- .get 메서드로 .env값에 접근(설정)

이후 module 파일에서는 imports 시 아래와 같은 형태를 취한다.  

```typescript
// user.module.ts
// ... 윗 줄 import 부분 생략
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useClass: JwtConfigService,
      inject: [ConfigService],
    }),
  ],
  providers: [UserService],
  exports: [UserService],
  controllers: [UserController],
})
export class UserModule {}
```

imports 내부에서 import, useClass, inject설정에 필요한 것들이 어떤 것들인지 인지해두자, 이 외 패키지를 추가해서 사용할 때도 위와 같은 방식을 취해야 할 것이다.


