# Transaction

트랜잭션(Transaction)은 하나의 쿼리(작업) 단위를 하나의 묶음으로서 처리하는 기능을 의미한다.  
가령 통장 입출금 시스템의 경우 A가 B에게 1000원을 송금하는 작업을  

1. A가 B에게 송금한다.
2. B의 잔액이 1000원 증가한다.

와 같은 작업이 이뤄져야 하는데 해당 작업이 알 수 없는 오류로 인해 중단된다면 A가 보낸 1000원이 증발할 수도 있다. 바로 이러한 현상을 방지하기 위해서 트랜잭션이 존재한다.

## Transaction의 특징 ACID

**Atomicity(원자성)** - 여러 쿼리 명령들을 하나의 작업 단위로 취급하여, 내부 쿼리 명령들이 전부 성공하거나, 아니면 모두 실패해야한다는 특징이다.  
**Consistency(일관성)** - 트랜잭션 처리 과정에서 데이터의 일관성을 유지해야하는 특징입니다. 즉 트랜잭션의 성공, 실패에 따라 데이터 또한 처리완료 또는 처리 전 이렇게 중간 단계 없이 이분법적으로 데이터가 유지 되어야 한다는 것을 의미한다. 원자성이 쿼리 명령 그룹의 관점에서 본다면 일관성은 데이터 처리 그룹의 관점에서 보는 것을 의미한다.  
**Isolation(격리성)** - 트랜잭션을 수행하는 중에는 외부에서 데이터에 접근하여 중간 상태를 보거나 변경할 수 없도록 구성하는 특징입니다. 만약 계좌 이체를 총 세번하는 트랜잭션이 처리 중인데 두 번째 계좌 이체 처리 후 누군가가 잔액을 조회할 수 있다면 안되는 일이기 때문에 작업 도중에는 해당 DB 오브젝트에 `Lock`을 걸어 또 다른 클라이언트의 접근을 막는다. 이는 여러 클라이언트가 하나의 DB에 동시에 접근할 수 있는 **동시성(Concurrency)**이 보장됨으로 인해 발생하는 문제인데 트랜잭션 처리 과정에서는 동시성 이슈를 방지하기 위해 트랜잭션을 수행한 유저에게만 해당 데이터를 점유할 수 있도록 하는 것이다.  
**Durability(지속성)** - 트랜잭션을 성공적으로 수행했다면 수정된 데이터가 영구적으로 데이터베이스에 반영 되어야 한다는 특징이다. 이는 데이터베이스에 이상이 생겨 재부팅을 한다 하더라도 이전까지의 트랜잭션 변경 사항이 그대로 반영되어 지속시킬 수 있어야 함을 의미한다.

## LOCK의 종류

하나의 트랜잭션이 수행되는 동안 외부 클라이언트의 점유를 방지하기 위한 수단이다.  
동시성을 다소 해치게 되더라도 데이터의 일관성을 유지하기 위한 Trade-off를 조절하기 위해 여러 Lock의 종류가 존재한다.

### Shared and Exclusive Locks

**Shared Lock (공유락)**의 경우 다른 트랜잭션이 읽기(READ)는 가능하지만 쓰기(WRITE)는 불가하도록 제한하는 Lock이다. 누군가가 읽고 있다면 수정이 불가하며, READ 전용 LOCK이라고도 한다.  
**Exclusive Lock (배타락)**의 경우 다른 트랜잭션이 읽기, 쓰기 모두 불가하도록 제한하는 Lock이다. 누군가가 데이터를 점유하고 있다면 타 사용자는 점유 자체가 불가하며, WRITE 전용 LOCK이라고도 부른다.

## LOCKING LEVEL

**Global Locks | Database Locks** - `데이터베이스의 모든 테이블에 락`을 걸어, 현재 트랜잭션을 제외한 나머지 트랜잭션들이 모든 테이블을 사용할 수 없도록 만든다. 즉 한 사람씩 데이터 베이스에 접근할 수 있음을 의미한다.  
**Table Locks** - `다른 사용자가 작업중인 테이블에 락`을 걸어 테이블 단위로 동시에 수정하지 못하도록 한다.  
**Named Locks** - `특정한 문자열을 점유하는 락`을 의미한다.  
**Metadata Locks** - `다른 사용자가 작업중인 테이블의 동일한 행과 동일한 데이터베이스의 객체에 락`을 걸어 동시에 수정하지 못하도록 한다.

> 🚨 `교착상태`  
> 락을 잘못 걸게되면 아무도 특정 리소스에 접근할 수 없게 되는 `교착 상태(Dead Lock)`에 빠지게 된다.  
> 가령 A 트랜잭션이 a테이블에서 처리 후 b 테이블을 처리하는 작업이고, B 트랜잭션이 b테이블 처리 후 a 테이블을 처리하는 작업이라고 하자  
> A와 B가 비슷한 시기에 트랜잭션을 수행한다면 A는 a처리 후 b로 가야하는데 B가 b를 점유하고 있고, 반대로 B는 b처리 후 a로 가야하는데 A가 a를 점유하는 상황이 발생한다.  
> 그러면 A와 B는 각자의 남은 트랜잭션을 수행하기 위해 무한정 대기하게 되는 교착 상태에 빠지게 된다.

## Isolation Level

트랜잭션은 격리 수준을 구분하여 접근 권한의 정도를 구분한다.

`READ UNCOMMITTED` - **커밋 되지 않은 데이터의 읽기(Uncommitted Read)를 허용**하는 격리 수준입니다. 이는 다른 트랜잭션에 의해 작업 중인 데이터를 읽는 것을 의미하므로 잘못된 참조를 부르게 된다. 이는 곧 일관성이 꺠짐을 의미한다.  
`READ COMMITTED` - **커밋 된 읽기(Committed Read)만을 허용**하고, `SELECT` 문을 실행할 때 공유락을 걸어 쓰기는 불가한 수준을 의미한다.  
`REPEATABLE READ` - **트랜잭션이 완전히 종료될 때 까지 락을 유지**합니다. 하지만 다른 트랜잭션에서 데이터를 삽입하는 것은 가능하므로 **팬텀 읽기**가 발생할 수 있는 문제점이 있다. 여기서 팬텀 읽기는 존재하지 않는 데이터를 참조하고 있는 상태를 의미한다. 즉 누군가 데이터를 중도 삽입했지만 점유 중인 트랜잭션 처리에 따라 즉시 삭제될 수도 있는데 이럴 경우 해당 데이터는 있다고 인식되지만 실제하지 않게 된다.  
`SERIALIZABLE` - **가장 높은 수준의 격리 수준**으로 점유 시 다른 클라이언트는 아무 것도 못한다. 동시성이 떨어지는 문제점이 존재한다.

## MySQL 트랜잭션 사용의 기본 예시

```sql
-- MYTABLE 테이블을 생성합니다.
CREATE TABLE IF NOT EXISTS MYTABLE
(
    id       INT(11)      NOT NULL PRIMARY KEY AUTO_INCREMENT,
    name     VARCHAR(255) NOT NULL,
    location VARCHAR(255) NOT NULL
);

-- 1번째 트랜잭션을 실행합니다.
START TRANSACTION;

-- 마이 테이블에 더미 데이터 3개를 삽입합니다.
INSERT INTO MYTABLE (name, location)
VALUES ('apple', 'SEOUL'),
       ('ball', 'BUSAN'),
       ('cat', 'DAEGU');

-- 1번째 트랜잭션을 DB에 적용합니다.
COMMIT;


-- 2번째 트랜잭션을 실행합니다.
START TRANSACTION;

-- 마이 테이블에 더미 데이터 3개를 삽입합니다.
INSERT INTO MYTABLE (name, location)
VALUES ('doll', 'SEOUL'),
       ('egg', 'BUSAN'),
       ('fish', 'DAEGU');

-- 2번째 트랜잭션을 롤백합니다.
ROLLBACK;
```

## Sequelize로 Transaction하기

### managed와 Unmanaged 트랜잭션

sequelize에서 Transaction는 어떻게 사용할 수 있는지 알아보자

sequelize에서 트랜잭션을 적용하는 방법은 크기 `Managed` 방식과 `Unmanaged` 방식으로 나뉜다.

```javascript
// Managed
const { sequelize, User } = require('../models');

// 트랜잭션의 콜백으로 유저 생성 로직을 처리합니다
const result = await sequelize.transaction(async (t) => {
  const user = await User.create(
    {
      nickname: 'apple',
      age: 18,
    },
    { transaction: t } // 해당 로직이 트랜잭션으로 묶여있음을 명시함
  );

  return user;
});
```

위 코드에서 보는 것과 같이 단순히 models/index.js에서 sequelize를 import한 뒤 기존에 적용하던 유저 생성 로직을 콜백 함수로 감싸주는 것을 확인할 수 있다.  
unmanaged와 비교하면 금방 알게 되겠지만 위 코드에서 요청 성공 또는 에러에 따른 `COMMIT`과 `ROLLBACK` 명령이 따로 명시되어 있지 않지만 그럼에도 자체적으로 동작한다. 그래서 코드를 간결하게 작성할 수 있다는 장점이 있다.  
하지만 `에러 상황 외 롤백이 필요한 로직의 경우 어떻게 짜야 하는가?` 에 대한 문제점이 있다. 가령 User 생성 후 생성이 잘 되었는가에 대한 검증을 거친다고 가정했을 때 문제가 있다면 다시 롤백처리 되도록 로직을 구성할 수도 있을 것인데 managed에서는 이러한 추가 로직 구성에 있어 제한이 있다.

```javascript
// Unmanaged
const { sequelize, User } = require('../models');
const { Transaction } = require('sequelize');

// 트랜잭션의 콜백으로 유저 생성 로직을 처리합니다
const t = await sequelize.transaction({
  isolationLevel: Transaction.ISOLATION_LEVELS.READ_COMMITTED,
});

try {
  const user = await User.create(
    {
      nickname: 'apple',
      age: 18,
    },
    { transaction: t } // 해당 로직이 트랜잭션으로 묶여있음을 명시함
  );

  await t.commit();
} catch (transactionError) {
  await t.rollback();
}
```

Unmanaged 방식은 트랜잭션의 t를 변수로 따로 할당하면서 `IsolationLevel(격리 수준)`을 설정할 수 있다.  
또한 commit과 rollback을 명확하게 명시하고 있다는 점이 managed와의 차이점이라 할 수 있다. 이는 managed 방식과 동일한 작동방식을 보이나 요청에 대한 성공과 에러 발생에 대한 명확한 구분을 보이고 있으며 특히 메인 task 이외 추가적인 task가 필요할 경우에도 롤백을 적용할 수 있다는 점에서 managed와의 차별점을 보인다.  
개인적으로는 항상 협업의 상황을 가정한다고 했을 떄 코드 공유를 위해 명확하게 명시되는 것이 좋다고 생각하며 또한 확장성을 고려했을 때 Unmanaged 방식이 더 선호되어야 한다고 생각한다.

이제 unmanaged 방식을 기준으로 실제 예시를 살펴보자  
회원가입 task를 수행하는 과정에서 유저 로그인 시 필요한 이메일과 패스워드정보는 User 테이블에 저장하고, 그 외 유저에 대한 정보는 UserInfo 테이블에 따로 저장한다고 했을 때 아래와 같은 트랜잭션을 적용할 수 있을 것이다.

```javascript
// Unmanaged
const { sequelize, User, UserInfo } = require('../models');
const { Transaction } = require('sequelize');
const express = require('express');
const router = express.Router();

router.post('/users', async (req, res) => {
  const { email, password, nickname, name, age, gender } = req.body;

  // 트랜잭션을 정의합니다.
  const t = await sequelize.transaction({
    isolationLevel: Transaction.ISOLATION_LEVELS.READ_COMMITTED,
  });

  try {
    // 유저를 생성합니다.
    const user = await User.create({ email, password }, { transaction: t }); // 트랜잭션 등록

    // 유저 정보를 생성합니다.
    const userInfo = await UserInfo.create(
      {
        nickname,
        name,
        age,
        gender,
      },
      { transaction: t } // 트랜잭션 등록
    );

    await t.commit(); // 성공 시 커밋
  } catch (transactionError) {
    await t.rollback(); // 실패 시 롤백
    return res.status(400).send({ message: '유저 생성 실패' });
  }

  return res.status(201).send({ message: '유저 생성 성공' });
});
```

위 코드와 같이 두 가지 다른 모델에 데이터를 생성하는 경우 하나의 트랜잭션으로 묶어서 처리하도록 함을 알 수 있다.  
기억해야할 점은 하나의 트랜잭션으로 묶일 수 있도록 반드시 각 모델 접근 task에 {transaction: t}를 적용해야 한다는 점이다.

## UUID 사용하기

UUID(범용 고유 식별자)는 테이블의 ID로 사용하는 식별자로 총 4개의 하이푼(-)으로 구분된 문자열로 구성되어 있다. 해당 식별자를 사용하는 가장 큰 장점은 전 세계적으로 중복값이 존재할 가능성이 극히 낮다는 점, 즉 고유성이 보장된다는 점 때문이다.

자세한 내용은 생략하고 UUID를 MySQL sequelize에 적용하는 방법에 대해서 살펴보자

```javascript
// migration
'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('RefreshTokens', {
      uuid: {
        allowNull: false,
        primaryKey: true,
        type: Sequelize.UUID,
        defaultValue: Sequelize.UUIDV1,
      },
      refreshToken: {
        type: Sequelize.STRING,
      },
      UserId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Users',
          key: 'userId',
        },
        onDelete: 'CASCADE',
      },
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('RefreshTokens');
  },
};
```

리프레시 토큰과 유저 id를 보관하는 테이블을 생성한다고 했을 때 먼저 sequelize-cil로 모델을 generate한 뒤 생성된 마이그레이션 파일에서 id 필드의 type와 defaultValue를 위와 같이 변경해준다.  
defaultValue에서 UUID의 버전을 V1 또는 V4를 설정할 수 있는데 V1을 설정할 경우 UUID 디코딩을 통해 해당 데이터가 생성된 시점을 알 수 있다. 이는 타임스탬프 기반으로 UUID가 생성하기 떄문이다.  
하지만 V4로 설정할 경우 완전 랜덤값을 기반으로 생성되므로 거의 완벽에 가까운 고유성을 지닌다는 장점이 있지만 어떠한 의미있는 정보를 담지는 않는다.

아래 사이트를 통해 디코더 결과를 확인할 수 있으며 V1과 V4로 설정한 UUID값을 각각 적용해보면 결과가 다른 것을 확인할 수 있다.  
[UUID 디코더 사이트](https://www.uuidtools.com/decode)

정리하자면 UUIDV1을 사용할 경우 디코딩을 활용한다면 상황에 따라 createdAt과 updatedAt을 사용하지 않아도 되므로 필드의 수를 줄일 수 있는 효과를 볼 수 있다.

```javascript
// model
RefreshToken.init(
  {
    uuid: {
      allowNull: false,
      primaryKey: true,
      type: Sequelize.UUID,
      defaultValue: Sequelize.UUIDV4,
    },
    refreshToken: {
      type: DataTypes.STRING,
    },
    UserId: {
      type: DataTypes.INTEGER,
    },
  },
  {
    sequelize,
    modelName: 'RefreshToken',
    timestamps: false,
  }
);
return RefreshToken;
```

migration 파일과 함께 model 파일도 생성되었을 것인데, 이 부분도 수정이 필요하다. migration 파일과 같이 id 부분의 type과 defaultValue를 설정해주면 되며, createdAt과 updatedAt을 사용하지 않을 경우 `timestamps; false` 설정해 주면 된다.

### UUID는 사용해야 하나?
고유성도 중요하지만 UUID값은 사실 일반 id와 비교했을 때 차지하는 크기가 다르다. 또한 인덱싱 관련해서도 문제가 될 수 있다. 문자열이며, 길이가 긴 만큼 find 하는데 드는 비용이 숫자에 비해 클 수 밖에 없다.  
하지만 클러스터 규모로 데이터를 관리하는 같은 분산 처리 시스템에서는 확실한 고유성을 보장한다는 측면에서 충돌이 발생할 가능성이 지극히 낮다는 장점이 있다. 만약 일반 숫자를 id로 사용하고 autoIncrease 되는 형태의 테이블이라면 클러스터 내 다양한 노드들에 분산 저장되어 있는 데이터들이 각 노드를 벗어났을 떄에도 고유성을 지닌다는 보장을 UUID만큼 할 수 없기 때문이다.  
그러므로 데이터베이스의 규모에 따라 판단할 문제인 것 같다.
