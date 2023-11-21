# 이벤트기반 아키텍처

> 아래 설명은 우아한테크 MSA의 이벤트기반 아키텍처 영상을 참고하여 정리한 내용입니다.  
> [영상 참조](https://youtu.be/b65zIH7sDug?si=2ijo3LU5KFbYeQFJ)

## MSA와 EDA

`EDA(Event-Driven Architecture)`는 `MSA(Microservice Architecture)`도입의 핵심 이유 중 하나인 `서비스의 독립`을 충족시키기에 적합하여 적용된다. 사업 규모가 커지게 되면 다수의 회사가 Monolithic 구조에서 MSA로의 전환을 고려하는데, 그 이유는 내부적으로 관리해야할 서비스의 확장, 규모가 증가하는데 있다.

가령 로그인 기능을 생각해보자. 과거 서비스 규모가 작을 떄는 로그인 요청 시 계정 검증, 로그인 허용 정도의 수준에서 끝났을지도 모른다.  
하지만 이후 사업이 확장되면서 유저가 로그인을 요청했을 뿐인데 '로그인 기록 남기기(로깅)', '회원 대상 이벤트 띄우기', '회원에게 온 알림 메시지 불러오기', '동일 디바이스에 로그인 된 타 계정 로그아웃 처리하기', '일일 출석체크 처리하기', '이전에 봤던 게시글 기록 나열하기', '작성 중이던 게시글 보여주기' 등등 다양한 로직을 한 번에 수행해야 하게 될 것이다.

위와 같이 하나의 로직을 수행하는데 관여하는 부가적인 로직의 수가 증가하면 이후 단 하나의 서비스만 추가해도 어디에서 어떤 에러가 터질지 감을 잡기 어려워지므로 (얽히고 설켜있기 때문에) 확장성이 떨어지게 되고, 그만큼 관리가 어려워질 수 밖에 없다.  
특히 위 예시와 같이 로그인과 관련된 회원 도메인이 이벤트 알림과 관련된 이벤트 도메인에 관여하는 로직은 이벤트 도메인 담당 팀에서 예측하지 못한 오류를 자아낼 가능성도 존재하게 된다.

그러므로 각각의 도메인은 독립적으로 관리되어야 한다는 니즈가 발생하고, 서로 최대한 관여하지 않는 것을 목표로 두고서 MSA로 확장한다. 이 때 각 도메인 간 최대한 관여하지 않도록, 즉 의존성을 낮추는 방향을 추구하는데, EDA가 이러한 조건을 달성하기에 용이하다.

## 느슨한 결합, 강한 결합

EDA를 구축함에 있어 고려해야할 점이 느슨한 결합을 이루고 있는가이다.

가령 회원의 본인 인증이 초기화되면 가족계정 서비스에서 탈퇴되어야 한다는 로직을 수행하는 과정을 통해 강한 결합과 느슨한 결합에 대해서 이해해보자.

<img width="794" alt="스크린샷 2023-11-15 오후 2 11 42" src="https://github.com/issuebombom/nodejs_study_alone/assets/79882498/d43d4037-00a0-41b1-91f6-41555029dfe8">

첫번째로 모놀리스에서는 본인 인증 로직에 가족계정 서비스 탈퇴 로직이 포함되어 있다. 그러므로 주요 동작인 본인 인증 초기화 동작이 가족계정 탈퇴 로직에 관여하게 되어 본인인증 초기화 로직에 높은 의존성을 갖게 되어 `강한 결합`을 보인다.

<img width="794" alt="스크린샷 2023-11-15 오후 2 12 06" src="https://github.com/issuebombom/nodejs_study_alone/assets/79882498/b5a0cbe7-51e3-4cbb-b7b2-1f24792ec966">

두번째로 MSA 구조로 변경하여 두 도메인이 물리적인 두 시스템으로 분리되었다. 하지만 회원 인증 초기화 로직이 HTTP 통신으로 가족계정 탈퇴 요청을 보내고 있다. 그러므로 물리적으로 거리가 멀어졌지만 여전히 가족 계정 서비스에 관여하고 있어 `강한 결합`을 보인다.

<img width="788" alt="스크린샷 2023-11-15 오후 4 12 32" src="https://github.com/issuebombom/nodejs_study_alone/assets/79882498/2d7038c5-8659-492c-b20b-251a7404e215">

세번째로 메시징 시스템을 도입하여 가족 계정 탈퇴 메시지를 메시지 큐로 보내고, 가족 계정 서비스는 해당 큐를 구독하여 메시지를 받는 구조이다. 이러한 경우 두 도메인 사이에 메시지 큐를 두고 있어 물리적 의존성은 제거되었다고 볼 수 있지만 논리적으로는 여전히 가족계정 탈퇴에 관여하고 있으므로 `강한 결합`으로 볼 수 있다.

<img width="788" alt="스크린샷 2023-11-15 오후 4 12 38" src="https://github.com/issuebombom/nodejs_study_alone/assets/79882498/c48c435f-cafb-4511-bbc4-67f51e048e31">

마지막으로 메시징 시스템을 통해 개인 인증 초기화를 실행한 후 "초기화를 실행했습니다." 라는 이벤트가 발생했다는 사실을 알려주면 (이벤트 발행) 가족 계정은 해당 알림을 받아 탈퇴 로직을 수행한다. 이와 같은 경우 단지 회원 인증 초기화를 실행했다는 사실을 알리는 것일 뿐 가족 계정 서비스에 관여하지 않지만, 가족 계정 서비스를 이 사실을 인지하고 해당 계정을 탈퇴 처리한다. 그러므로 직접적인 개입이 없으므로 `느슨한 결합`으로 볼 수 있다.

> EDA의 핵심은 회원 도메인이 가족계정 도메인에게 직접적인 로직 실행을 요청하지 않는 것에 있다.

여기서 EDA의 장점을 예상할 수 있는데, 바로 `독립적인 배포`에 있다. 더이상 도메인 간의 직접적인 간섭이 없고, 두 도메인 사이에 위치한 알림메시지에 따라 반응 여부가 결정되기 때문이다.

## 이벤트 발행과 구독

아래와 같은 로직이 있다고 가정하자

> \*로그인 이벤트 발생 시
>
> 1. 해당 디바이스의 로그인 처리
> 2. 타 디바이스 로그아웃 처리
> 3. 동일 디바이스 내 타 게정 로그아웃 처리
> 4. 디바이스 로그인 기록

로그인 요청 시 로그인 자체를 실행하는 것을 주요 로직이라고 본다면 나머지는 부가 로직에 해당하며 이를 "비관심사" 라고 부른다. MSA의 핵심인 느슨한 결합을 고려한다면 이러한 비관심사는 이벤트 기반으로 처리할 수 있다.

위 과정을 어플리케이션 이벤트, 내부 이벤트, 외부 이벤트 이렇게 총 3단계 이벤트로 처리할 수 있다.
<img width="1326" alt="스크린샷 2023-11-15 오후 4 15 26" src="https://github.com/issuebombom/nodejs_study_alone/assets/79882498/b2c4cee7-fed9-4fa5-bc24-7806e985cd59">

### 어플리케이션 이벤트

로그인이 실행되면 로그인 발생 이벤트를 발행하고, 이를 구독한 구독계층이 비관심사를 처리하거나 내부 이벤트를 발행한다.

<img width="603" alt="스크린샷 2023-11-15 오후 3 38 31" src="https://github.com/issuebombom/nodejs_study_alone/assets/79882498/8e299d62-22ea-44f4-8174-3af3f7eadb7f">

### 내부 이벤트

첫번째 구독계층에서 메시징 시스템 전달을 위한 로직을 수행한다.

<img width="721" alt="스크린샷 2023-11-15 오후 3 53 57" src="https://github.com/issuebombom/nodejs_study_alone/assets/79882498/fbf9108c-7cda-4747-8440-2cb4a320a9da">

AWS SNS에 어플리케이션 이벤트에서 발행한 이벤트가 적재된다.  
하나의 이벤트에 대한 다양한 로직 수행을 위해 여러 구독으로 나누어 준다.(SQS)  
이벤트 처리기(Event Worker)에서 내부 이벤트 비관심사를 실행한다.  
`어플리케이션 이벤트와 내부 이벤트는 하나의 시스템 안에 존재한다.`

> `"비관심사"를 어플리케이션 이벤트에서 한꺼번에 처리하면 되지 않을까?`  
> 주요 행위와 트랜잭션, 성능이 공유되어야 하는 비관심사는 어플리케이션에서,  
> 주요 행위와 트랜잭션, 성능 공유가 불필요한 비관심사는 내부 이벤트에서,  
> TRADE OFF

### 외부 이벤트

<img width="650" alt="스크린샷 2023-11-15 오후 3 54 11" src="https://github.com/issuebombom/nodejs_study_alone/assets/79882498/c2525327-9a63-44f5-851e-23fcf588d27a">

MSA에서 "로그인" 도메인이 아닌 타 도메인의 로직이 수행되어야할 수 있다. 이 경우 다른 시스템으로 이벤트를 전달해야 한다.  
외부 시스템에 대한 변경 권한은 외부 도메인 관리자에게 있으므로 변경 비용이 비싸다. 그렇기 때문에 페이로드 설정에 신중해야 한다.  
\*참조 영상에서는 "언제(시간), 누가(식별자), 무엇을 했고(행위), 어떤 변화가 발생했나(속성)"에 대한 정보를 페이로드로 채택한다.

### 문제점

내부 이벤트 발행 시 HTTP로 AWS SNS에 이벤트를 발행한다. 이 경우 어플리케이션 이벤트 발행과 로직 수행을 트랜잭션으로 공유하여 정합성을 보장할 수는 있으나 이로 인한 에러가 내부 이벤트로 발행될 위험성이 존재한다. HTTP 통신이 SNS, SQS만큼 구독, 발행의 신뢰도가 낮기 때문이다.

이를 해결하기 위해 `이벤트 저장소`를 두어 이벤트 발행 여부를 기록한다.

- 내부 이벤트 발행을 위한 구독 계층과 더불어 "이벤트 발행 여부를 기록(DB)"하는 구독 계층을 생성한다.
- 트랜잭션은 이벤트 기록 로직하고만 공유한다. 그러면 내부 이벤트 발행 구독계층이 이벤트 발행을 누락시켜도 DB를 통해 발행 유무를 확인해서 재발행하는 배치 처리가 가능하다.
- DB에 기록할 때는 초기 이벤트 발행 시 published 컬럼을 false로 발행하고, 내부 이벤트 처리기 단에서 false를 true로 변경해주는 방식을 취한다.

## 간단한 EDA 구현하기

> 회원가입이 완료되면 이에 대한 이벤트를 발행하고, 이메일 서비스에서는 이를 확인하여 가입 인증 메일을 발송하는 구조를 구현해보자

MSA 환경에서는 RabbitMQ나 Kafka와 같은 도구로 시스템 간 메시지를 발행, 구독할 수 있지만 여기서는 EDA에 대한 감을 익히기 위해 모놀리식 아키텍처에서 eventEmitter를 이용하여 EDA 느낌을 아래와 같이 구현해 보았다.

### 1. 이벤트를 어떤 형식으로 발행할 것인가?

이벤트 발행을 위한 작성 양식은 아래와 같이 고정한다. 또한 이벤트를 DB에 저장하여 이벤트 발행 유무 체크 및 이벤트를 기록한다.  
이벤트 발행 양식은 누가(who), 어떤 이벤트를 발행했고(what), 전달할 페이로드는 무엇인지?, 전달 완료 유무에 대해 작성하도록 한다.

```typescript
import { Column, CreateDateColumn, Entity, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Event {
  @PrimaryGeneratedColumn()
  id: string;

  @Column()
  who: string;

  @Column()
  what: string;

  @Column({ type: 'simple-json', nullable: true, default: null })
  payload: {};

  @Column({ default: false })
  published: boolean;

  @Column({ nullable: true, default: null })
  publishedAt: Date | null;

  @CreateDateColumn()
  createdAt: Date;
}
```

### 2. 신규 회원 생성 후 이벤트 발행하기

```typescript
// users.controller.ts
@Controller('api/users')
export class UsersController {
  constructor(
    private readonly usersService: UsersService,
    @InjectRepository(Event)
    private readonly eventRepository: Repository<Event>,
    private readonly eventEmitter: EventEmitter2
  ) {}

  @Post()
  // 인증받지 않은 유저 생성
  async createUserWithoutVerification(@Body() createUserDto: CreateUserDto) {
    const user = await this.usersService.createUserWithoutVerification({
      createUserDto,
    });

    // 회원 생성 완료 후 인증 메일 전송을 위한 이벤트 발행
    const event = new CommonEvent(user.id, 'user.created');
    // create event record
    const userCreatedEvent = await this.eventRepository.save(event);
    this.eventEmitter.emit('user.created', userCreatedEvent); // publish

    return { message: '회원 생성 완료, 이메일 인증 필요', statusCode: 200 };
  }
}
```

먼저 `CommonEvent`는 이벤트 발행을 위한 양식에 해당한다.

```typescript
// common.event.ts
import { IEvent } from '@nestjs/cqrs';

export class CommonEvent implements IEvent {
  constructor(
    readonly who: string,
    readonly what: string,
    readonly payload?: { [key: string]: any }
  ) {}
}
```

이벤트를 작성할 때는 기본적으로 누가 해당 이벤트를 발생시켰는지, 어떤 이벤트가 벌어졌는지 표기하도록 한다. 페이로드는 선택사항이다.  
사실 페이로드를 전혀 사용하지 않는 것이 유저 도메인과 이메일 도메인 간에 느슨한 결합을 제대로 이뤘다고 할 수 있겠으나, 예외 상황을 고려하여 작성 항목에 추가했다.

먼저 이벤트 저장소에 이벤트 레코드를 생성한다. 이 때 published가 false값으로 생성되며 이후 구독자가 비관심사를 처리한 뒤 true값으로 변경한다.

> 위와 같이 이벤트 발행과 구독이 일대일 관계로 연결되어 있다면 발행 완료 표기를 위와 같이 하면 되겠지만 1대다의 경우 다른 방법을 써야할 것이다.

emit을 통해 이벤트명을 'user.created'로 지정하여 DB에 생성한 이벤트 기록과 함께 발행한다. 이렇게하여 최종적으로 "이벤트 ID, 발행자, 발행이유" 이 세가지 정보를 전달한다.

### 3. 이메일 도메인에서 이벤트 구독하기

```typescript
// email-events.handler.ts
@Injectable()
export class EmailEventsHandler {
  constructor(
    @InjectRepository(Event)
    private readonly eventRepository: Repository<Event>,
    private readonly emailService: EmailService
  ) {}

  @OnEvent('user.created')
  async handleUserCreated(event: Event) {
    // 이벤트 수신 완료 처리
    const userCreatedEvent = await this.eventRepository.findOne({
      where: { id: event.id },
    });
    this.eventRepository.save({ ...userCreatedEvent, ...{ published: true } });
    // 인증 메일 발송
    this.emailService.sendMemberJoinVerification(event);
    // 이벤트 저장소에서 '인증 메일 발송' 이벤트 기록
    const e = new CommonEvent(event.who, 'verifyEmail.sent');
    this.eventRepository.save({
      ...e,
      ...{ published: true },
    });
  }
}
```

@OnEvent 데코레이터를 통해 'user.created'라는 이벤트를 구독하도록 한다. MSA 환경에서는 두 도메인 사이에 Queue가 존재하겠지만 여기서는 생략했다.

이메일 도메인에서는 구독한 이벤트를 받은 직후 DB에 생성된 해당 이벤트 기록에서 published를 true로 변경한다. 이로써 이메일 도메인으로 이벤트가 잘 발행되었음을 기록으로 확인할 수 있다.

그리고나서 가입 유저를 대상으로 인증 메일을 발송한다.

마지막으로 '인증 메일 발송' 이벤트를 기록한다. 해당 이벤트는 현재 후속 처리 로직이 없기 때문에 곧바로 published를 true로 설정한다. 나중에라도 이어지는 비관심사 처리가 추가된다면 여기서 또 다시 event를 emit하면 된다.

> 위와 같은 방식을 통해 유저 생성은 인증 메일 발송에 직접적인 개입을 하지 않고서도 이를 수행하게 되었다. 단지 유저 생성 이벤트가 발생했음을 이메일 도메인에게 알려준게 전부이기 때문이다.
