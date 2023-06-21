# 자바스크립트 Express 기본기

## Express로 API 서버 구축하기

우선 Express 구현에 집중하기 위해 사전에 준비된 members.js 파일에 기록된 멤버 정보를 `module.exports`로 전달받는 것으로 데이터베이스 `리소스` 통신을 대신한다.

> `💡 Tip`  
> 데이터베이스에 담긴 각종 데이터를 `Resource(자원, 정보)`라고 부른다.

### 멤버 리소스 가져오기

멤버 리소스를 불러오는 간단한 API를 구현하는 과제에서 아래와 같이 구현했다.

```javascript
// Before
const express = require('express');
const members = require('./members.js');
const app = express();

app.get('/api/members/:id', (req, res) => {
  const id = Number(req.params.id);
  const member = members.filter((member) => member.id === id);
  if (member) {
    res.send(member);
  } else {
    res.status(404).send({ msg: 'There is no such member' });
  }
});

app.listen(3000, () => {
  console.log('Server is listening...');
});
```

하지만 위 방식에서 params를 불러오는 방식에서 두 가지 수정이 필요했다.

```javascript
app.get('/api/members/:id', (req, res) => {
  const { id } = req.params;
  const member = members.find((member) => member.id === Number(id));
});
```

먼저 현재 상황에서는 path에서 id값 하나만 받지만 다른 경우에는 여러 개의 `route parmeter`를 받는 상황이 있을 수 있다. 이를 대비한다면 모든 api 구현에서 위 코드처럼 `구조분해 할당(destructuring)`으로 일괄 처리하는 것이 코드의 통일성과 가독성을 지킬 수 있을 것 같다.

또한 filter를 find로 변경했는데 이는 filter 메서드가 기본적으로 array형태로 출력된다는 점 때문이다. 그래서 filter 조건에 해당하는 값을 찾지 못할 경우 빈 array가 출력되고 이는 undefined 결과가 아니므로 응답 실패에 대한 처리를 못하게 된다.  
물론 array의 length가 0인 케이스를 실패로 처리할 수 있겠지만 id가 유일성을 지녔다는 점을 생각한다면 find 메서드로 대체하는게 깔끔하겠다는 생각이 들었다.  
find 메서드의 경우 조건에 부합하는 가장 먼저 발견된 결과를 object 형태로 출력하기 때문에 해당하는 값을 찾지 못할 경우 undefined를 출력하므로 응답 실패에 대한 처리를 간결하게 구현할 수 있게 된다.

### 멤버 리소스를 부서 기준으로 가져오기

```javascript
app.get('/api/members', (req, res) => {
  const { team } = req.query;
  if (team) {
    const teamMembers = members.filter((member) => member.team === team);
    res.send(teamMembers);
  } else {
    res.send(members);
  }
});
```

쿼리로 팀명을 입력하면 해당 팀의 멤버를 가져오는 API이다. 특정 팀명을 필터링하는 과정에서 찾는 결과가 없다면 "일치하는 결과가 없다."는 결과가 전달되어야 하므로 filter 메서드를 적용했다. 이 경우 조건에 부합하는 결과가 없을 경우 빈 array를 출력해주기 때문이다.

또한 API에 설정되지 않은 쿼리를 입력받을 경우 전체 멤버 정보를 출력하도록 처리했다. 이 부분은 선택사항인 것 같다. 상황에 따라 404 Not Found를 출력해야할 수도 있겠다는 생각이 들기 때문이다.

### 멤버 추가하기

```javascript
app.use(express.json()); // middleware: 리퀘스트의 라우트 핸들러에 도달하기 전 json에 대한 전처리

app.post('/api/members', (req, res) => {
  const newMember = req.body;
  members.push(newMember);
  res.send(newMember);
});
```

POST 리퀘스트에서 위 예시는 post fetch에서 body의 Content-Type이 `Application/json`임을 가정했다.  
Express에서는 json 타입의 body가 라우트에 들어오기 전 Express의 `미들웨어`라고 불리는 곳에 들러 jsonParser를 통해 전처리하는 과정을 거치도록 되어있다.

### 멤버 정보 수정하기

```javascript
app.put('/api/members/:id', (req, res) => {
  const { id } = req.params;
  const newInfo = req.body;
  const member = members.find((member) => member.id === Number(id));
  if (member) {
    Object.keys(newInfo).forEach((key) => {
      member[key] = newInfo[key];
    });
    res.send(member);
  } else {
    res.status(404).send({ message: 'There is no member with the id' });
  }
});
```

요청받은 id에 대한 멤버 정보를 가져와 json형태로 받은 body의 프로퍼티값을 덮어씌우고 있다.

> 💡 `Tip`  
> Object.entries를 쓰면 [ [key1, value1], [key2, value2] ...] 형태의 리스트를 반환해주므로 이를 이용한 프로퍼티 수정도 가능하다.

```javascript
Object.entries(newInfo).forEach((pair) => {
  member[pair[0]] = pair[1];
});
```

### 멤버 삭제하기

```javascript
app.delete('/api/members/:id', (req, res) => {
  const { id } = req.params;
  const membersCount = members.length;
  members = members.filter((member) => member.id !== id);
  if (members.length < membersCount) {
    res.send({ message: 'Deleted' });
  } else {
    res.status(404).send({ message: 'There is no member with the id' });
  }
});
```

현재 members 데이터가 object 형태이므로 id에 해당하지 않는 멤버를 필터링하여 object를 재할당하는 방식으로 멤버 정보를 제거했다.

## 실시간 코드 반영을 위한 nodemon 설정

nodemon을 사용하면 매 코드 수정 시 서버를 재실행하므로 코드 수정에 따른 서버 on/off를 수동으로 해주지 않아도 되서 편리하다.

### nodemon 설치하기

개발용도로만 쓰기 위해 아래와 같이 설치한다.

```zsh
npm install nodemon --save-dev
```

`--save-dev` 옵션은 해당 패키지를 node_modules에 설치하는 것은 동일하나 package.json 파일에서 디펜던시가 아닌 dev디펜던시에 기록되도록 하기 위함이다.  
프로젝트 공유 시 상대방에게 package.json을 넘겨주면 해당 파일을 프로젝트 폴더에 복사해서 `npm install`를 하는 방식으로 패키지를 설치하는데 `npm install --production` 옵션을 추가하여 설치하면 dev디펜던시를 제외한 설치가 가능하기 때문이다.  
배포 단계에 있다면 굳이 dev디펜던시 설치로 인해 불필요한 용량 차지를 방지할 수 있다.

> 💡 `Tip`  
> 우리가 흔히 쓰는 `npm install`은 사실 default 옵션으로 설정된 `npm install --save-prod` 명령과 동일하다. 즉 `--save-prod`는 디펜던시 등록을 의미한다.

설치 후 실행은 아래와 같이 한다.

```zsh
npx nodemon app.js
```

### package scripts 수정을 통해 단축 명령 설정하기

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  }
```

프로젝트 내부에 package.json파일에서 "scripts" 부분은 명령어 실행을 간결하게 하는 용도로 사용된다.  
그래서 아래와 같이 서버를 실행하는 명령어를 아래와 같이 설정하는것이 관행이다.

```json
"scripts": {
    "start": "node app.js"
  }
```

위와 같이 설정하면 `npm start`라는 명령만으로도 서버를 실행할 수 있게 된다.

```zsh
npm start
```

지금은 `node app.js`라는 명령어도 짧아서 굳이 필요할까 싶지만 앞으로 길어질 명령어에 대해서는 이 방법이 편리하기 때문에 설정법을 알아둬야 한다.

```json
"scripts": {
  "start": "node app.js",
    "dev": "nodemon app.js"
  }
```

이번에는 추가적으로 nodemon 실행을 간결하기 위한 `dev` 단축어를 추가했다. 하지만 `start`와 달리 dev에 대해서는 아래와 같이 명령해야 한다.

```zsh
npm run dev
```

왜냐하면 `start`와 같은 키워드는 run이 없어도 되도록 후보 명령어로서 미리 설정이 되어있지만 그 외 단어에 대해서는 그렇지 않기 때문에 `run`을 추가로 입력해야 한다.

## MYSQL 데이터베이스 활용과 ORM

### 기본 환경 설정하기

node에서 MySQL 데이터베이스를 사용하기 위해 Community Server 설치가 필요하다.  
https://dev.mysql.com/downloads/

또한 node와 데이터베이스를 ORM 방식으로 사용하기 위해 추가적인 npm 패키지 설치가 필요하다.

```zsh
npm install mysql2 sequelize sequelize-cli
```

위 패키지 중 `sequelize-cli`는 필수 패키지는 아니나 데이터베이스 생성이나 마이그레이션 등을 cli 명령으로 가능하게 하여 작업의 편의성을 더해준다.

> 💡 `Tip`  
> `ORM`은 데이터베이스에 있는 객체를 하나의 객체에 매핑시키는 기술  
> 데이터베이스 문법을 잘 모르더라도 자바스크립트 언어로 비교적 쉽게 상호작용을 하기 위해 사용된다.

이제 ORM 환경 설정을 위해 데이터베이스 환경 초기화를 한다.

```zsh
npx sequelize init
```

위 명령을 수행하면 ORM 동작 환경을 위한 폴더 및 파일이 자동으로 생성된다.

이제 데이터베이스 환경을 설정해준다.

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_production",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```

config.json 파일을 보면 `development`, `test`, `production` 이렇게 세 가지로 데이터베이스를 구분해서 관리할 수 있게금 설정이 준비되어 있다. 개발 단계에서 사용하는 데이터베이스와 서비스 단계에서 사용하는 데이터베이스의 구분을 위함이다.

우선 개발 단계에 있으므로 development의 값을 수정해 준다.

먼저 MySQL Community Server 설치 시 지정해줬던 username과 password를 입력해준다. username의 경우 특별한 설정을 하지 않았다면 'root'로 지정된다.  
그리고 database의 이름을 설정해 준다. 현재 'database_development'라고 입력된 내용 대신 사용할 데이터베이스 이름을 정해주면 된다.

이후 아래 명령어를 실행하여 데이터베이스를 생성한다.

```zsh
npx sequelize db:create --env development
```

> 💡 `Tip`  
> `npx`는 패키지에 내장된 명령을 실행하기 위해 사용하는 키워드

참고로 `--env development` 옵션은 development용으로 사용할 것을 지정하는 옵션이다. 해당 옵션은 디폴트값에 해당하므로 사실 개발용 DB 생성 시에는 옵션을 주지 않아도 된다.

### 마이그레이션 생성하기

데이터베이스 생성 후에는 테이블 생성을 진행한다. 사실 순수 MySQL 문법을 사용한다면 테이블 생성 또한 MySQL 문법을 입력하여 생성해야 한다.  
하지만 ORM 방식을 채택한다면 `마이그레이션` 생성을 거쳐 테이블의 변동사항을 적용하고 logging할 수 있다.

ORM에서의 마이그레이션 생성은 sequelize-cli 명령으로 수행한다.

```zsh
npx sequelize model:generate --name Member --attributes name:string,team:string,position:string,emailAddress:string,phoneNumber:string,admissionDate:date,birthday:date,profileImage:string
```

위 명령을 해석하면 아래와 같다.

- model을 생성하는데 이름을 'Member'로 설정한다.
- 모델에 포함시킬 속성은 속성명:타입으로 설정한다.

서버 API에서는 `모델`이 곧 DB의 `테이블`과 연동되는 클래스를 의미하고, 모델의 `프로퍼티`가 곧 `테이블의 컬럼(속성)`을 뜻한다.  
간단하게 정리하자면 테이블을 생성했고, 그 안의 컬럼명을 지어줬는데 각 컬럼에 담을 데이터타입도 함께 지정해줬다.

### 마이그레이션 생성 결과 및 추가 작업

위 명령을 수행하면 `models` 폴더에 index.js와 members.js 파일이 생성된 것과 `migrations` 폴더에 마이그레이션이 생성된 것을 확인할 수 있다. 특히 members.js파일을 보면 내가 설정한 테이블 양식이 클래스와 프로퍼티 형태로 정리된 것을 알 수 있다.

또한 마이그레이션 파일을 확인해 보면 `우리가 정의하지 않았던 "id" ,"createdAt", "updatedAt" 속성이 자동으로 추가된 것을 확인할 수 있다.` 이 속성들은 테이블에서 반드시 필요한 속성이므로 항상 자동으로 생성해준다.  
이 중 createdAt과 updatedAt 속성에는 아래 프로퍼티를 추가해주면 좋다.

```javascript
defaultValue: Sequelize.fn('now');
```

이는 업로드 및 수정 시각을 자동적으로 데이터가 반영되는 현재 시각으로 등록되도록 하기 위함이다.

아래 명령을 통해 해당 마이그레이션을 `migrate`하면 비로소 MySQL 데이터베이스에 테이블이 생성된다.

```zsh
npx sequelize db:migrate
```

다시 한 번 정리하자면  
ORM 방식에서는 테이블 생성을 위해 테이블을 정의해주면 이에 맞는 마이그레이션과 js파일을 생성해주고, 마이그레이션 파일을 대상으로 migrate 명령을 수행하면 그제서야 데이터베이스에 테이블이 생성된다.

ORM 방식은 항상 이러한 형태로 테이블을 관리한다. Django 프레임워크의 ORM도 이와 동일하며 이번 예시는 sequelize라는 ORM을 적용했다고 이해하면 된다. 각 ORM 간 명령어는 다를지라도 작동 방식에 큰 차이는 없다.

> 💡 `Tip`  
> sequelize의 재밌는 점은 모델명을 'Member'라고 설정했음에도 복수 형태인 'Members'로 생성해 준다는 점이다.

### 테이블 삭제 기능

최종적으로 진행한 migrate를 `UNDO` 명령을 통해 실행을 되돌리는 방식으로 삭제가 가능하다.

```zsh
npx sequelize db:migrate:undo
```

### 테이블의 기본 데이터 타입 정의

기본적으로 사용되는 데이터타입 중에서 ORM방식의 타입 지정이 실제 데이터베이스에서 어떠한 타입을 지정하는 것과 같은지 확인한다.

| sequelize-ORM     | DATABASE     |
| ----------------- | ------------ |
| Sequelize.STRING  | VARCHAR(255) |
| Sequelize.INTEGER | INTEGER      |
| Sequelize.FLOAT   | FLOAT        |
| Sequelize.DATE    | DATETIME     |

### 데이터베이스에 첫 데이터(seed) 넣기

데이터베이스와 테이블 정의가 끝났다면 첫 데이터를 입력하는 과정을 seed데이터를 넣는다고 표현한다.  
이 과정에서는 아래와 같은 순서를 따른다.

1. sequelize-cli를 통해 시드데이터 추가를 위한 파일을 생성한다. (마이그레이션 틀을 만드는 것과 비슷합니다.)
   ```zsh
   npx sequelize seed:generate --name initialMembers
   ```
   위 명령어 실행을 완료하면 `seeders` 폴더가 생성되고, 그 안에 `initialMembers`이름이 딸린 파일이 생성된 것을 확인할 수 있다.
2. seed 파일을 살펴보면 아래와 같이 작성 방법이 제공되는데 가이드를 따라서 입력할 데이터를 작성합니다.

   ```javascript
   'use strict';

   /** @type {import('sequelize-cli').Migration} */
   module.exports = {
     async up(queryInterface, Sequelize) {
       /**
        * Add seed commands here.
        *
        * Example:
        * await queryInterface.bulkInsert('People', [{
        *   name: 'John Doe',
        *   isBetaMember: false
        * }], {});
        */
     },

     async down(queryInterface, Sequelize) {
       /**
        * Add commands to revert seed here.
        *
        * Example:
        * await queryInterface.bulkDelete('People', null, {});
        */
     },
   };
   ```

   이전에 봤던 migration처럼 up, down 메소드로 구분되어 생성 및 삭제가 가능함을 알 수 있다.

   ```javascript
   await queryInterface.bulkInsert(
     'People',
     [
       {
         name: 'John Doe',
         isBetaMember: false,
       },
     ],
     {}
   );
   ```

   up 부분을 보면 이와 같은 형식으로 데이터를 입력할 것을 가이드한다. 'People'부분에 실제 테이블명을 넣어주고 해당 프로퍼티를 작성해주면 된다.

3. seed파일을 기준으로 데이터베이스에 데이터를 입력한다.
   ```zsh
   npx sequelize db:seed:all
   ```
   이 또한 migrate와 동일한 행위에 속하므로 결과에서 migrated되었다는 문구를 확인할 수 있고, 이제 seed데이터가 데이터베이스에 저장되었다.

### 모델과 테이블 연동하기

생성한 members.js를 보면 테이블에 대한 정의를 볼 수 있다.

```javascript
const { Model } = require('sequelize');
module.exports = (sequelize, DataTypes) => {
  class Member extends Model {}
  Member.init(
    {
      name: DataTypes.STRING,
      team: DataTypes.STRING,
      position: DataTypes.STRING,
      emailAddress: DataTypes.STRING,
      phoneNumber: DataTypes.STRING,
      admissionDate: DataTypes.DATE,
      birthday: DataTypes.DATE,
      profileImage: DataTypes.STRING,
    },
    {
      sequelize,
      modelName: 'Member',
    }
  );
  return Member;
};
```

가이드를 위한 주석과 `static associate(models){}` 부분을 우선 제외하면(나중에 테이블간 관계 설정 시 사용) 위 코드가 남게 된다.  
테이블의 정의를 보면 primary key에 대한 정보가 담겨있지 않은데, 보통 id는 자동 생성되는 영역이므로 이 곳에 정의되지 않는게 일반적이다. 하지만 현재 실습 중인 예시의 경우 members 데이터에 이미 id속성이 존재하고, 이를 primary key로 사용하면서 신규 맴버 생성 시 id값을 input할 예정이므로 아래와 같이 추가해준다.

```javascript
Member.init(
    {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: DataTypes.INTEGER,
      },
      name: DataTypes.STRING,
      team: DataTypes.STRING,
      position: DataTypes.STRING,
      emailAddress: DataTypes.STRING,
      phoneNumber: DataTypes.STRING,
      admissionDate: DataTypes.DATE,
      birthday: DataTypes.DATE,
      profileImage: DataTypes.STRING,
    },
    ...
)
```

위 id 정보는 마이그레이션에서 가져왔으며 type의 `Sequelize.INTEGER` 부분을 `DataTypes.INTEGER`로 바꿔주면 된다. 이렇게 하면 이제 맴버 ID를 primary key로 쓰게되며, 이제는 데이터 입력 시 id값을 입력해줘야 한다.

이 밖에도 만약 name, team 등 migrate 파일에서 특정 필드를 대상으로 validation 등 기능을 추가 설정한 뒤 migrate했다면 모델에도 그대로 반영해주자. 아래 예시를 통해 확인해보자

```javascript
// migrate js파일
username: {
  type: Sequelize.STRING(20),
  allowNull: false,
  unique: true,
  validate: {
    len: {
      args: [3, 20],
      msg: '최소 3자 이상 입력해야 합니다.',
    },
    is: {
      args: /^[a-zA-Z0-9]+$/,
      msg: '영문 대소문자, 숫자만 입력이 가능합니다.',
    },
  },
},
```

위 예시처럼 유저 네임 생성 시 입력 제한을 설정했다면 해당 코드를 그대로 복사해서 models 폴더 내 user.js 모델에도 똑같이 적용해주자

```javascript
User.init({
  username: {
    type: DataTypes.STRING(20),
    allowNull: false,
    unique: true,
    validate: {
      len: {
        args: [3, 20],
        msg: '닉네임을 최소 3자 이상 입력해 주세요.',
      },
      is: {
        args: /^[a-zA-Z0-9]+$/,
        msg: '닉네임은 영문 대소문자, 숫자만 입력해 주세요.',
      },
    },
  },
});
```

User.init에 기존에 있던 필드 내용을 덮어쓰기 한다. 그리고 `Sequelize.STRING(20),`을 `DataTypes.STRING(20),`으로 변경해주자  
그 외에도 만약 `Sequelize.fn('now'),` 코드가 있다면 `DataTypes.NOW`로 변경해줘야 한다. 이처럼 부분 수정이 필요하다는 사실을 기억하자

이제 index.js파일을 통해 app.js와 모델을 연동하기 위한 작업을 진행한다.  
빠른 이해를 돕기 위해 index.js에 이미 작성된 기존 코드값을 지우고 아래 코드를 작성한다.

```javascript
const Sequelize = require('sequelize');
const config = require('../config/config.json');

const { username, password, database, host, dialect } = config.development;
const sequelize = new Sequelize(database, username, password, {
  host,
  dialect,
});

const Member = require('./member')(sequelize, Sequelize.DataTypes);

const db = {};
db.Member = Member;

module.exports = db;
```

sequelize와 config.json을 불러온 뒤  
먼저 config.development에 사전에 적어둔 데이터베이스 관련 정보를 불러옵니다.  
그리고 나서 Sequelize 객체를 생성하는데 활용하고,  
Member 객체를 불러옵니다. 이 때 member.js에서 공개된 함수의 실행을 통해 초기 설정을 마무리합니다.  
이제 db라는 오브젝트로 담아 외부에 공개합니다.

최종적으로 app.py에 모델을 불러옵니다.  
이때 신기한 점은 `require에서 models 디렉토리만 선택했지만 자동으로 하위 파일인 index.js을 찾아서 db 오브젝트를 불러 올 수 있다는 점`입니다.

> 💡 `Tip`  
> index.js파일을 자동으로 찾는 것은 node의 특징입니다.

```javascript
// app.py

const db = require('./models');
const { Member } = db;'
```

이제 모델과 데이터베이스를 연동하였고, 모델을 통해 데이터베이스를 수정할 수 있게 되었습니다.

### ORM 방식으로 MySQL 데이터베이스에 GET 요청하기

아래 코드는 전체 데이터와 필터링에 대한 GET 방식을 구현한다.

```javascript
const express = require('express');
const app = express();
const db = require('./models');
const { Member } = db;

app.use(express.json());

app.get('/api/members', async (req, res) => {
  const { team } = req.query;
  if (team) {
    // const teamMembers = await Member.findAll({ where: { team: team } });
    const teamMembers = await Member.findAll({ where: { team } }); // 위와 동일
    res.send(teamMembers);
  } else {
    const members = await Member.findAll();
    res.send(members);
  }
});

app.listen(3000, () => {
  console.log('Server is listening...');
});
```

위 코드에서 데이터베이스를 가져오는 방법과 데이터를 찾는 방법에 주목할 필요가 있다.  
먼저 위에서도 언급했지만 require를 통해 models 디렉토리를 가져오면 하위파일 중 index.js를 자동으로 가져오게 되고(node.js의 특징), index.js는 db라는 변수를 export하고 있어 결론적으로 db 변수에 담는다.  
이후 db 안에 담긴 Member라는 객체(테이블)를 가져온 뒤 findAll 메서드로 데이터를 가져온다.

> 💡 `Tip`  
> findAll 메소드의 where에서 {team: team}을 `Shorthand Property Names` 기법을 적용한 것을 볼 수 있다.  
> 이를 통해 찾고자 하는 key의 값과 확인할 데이터의 변수명이 동일할 경우 하나로 쓸 수 있다.

### sequelize ORM find의 또다른 기능들

```javascript
// 하나만 찾기
const member = await Member.findOne({ where: { id } });
res.send(member);

// 기본 정렬
const members = await Member.findAll({
  order: [
    ['admissionDate', 'DESC'],
    ['team', 'ASC'],
  ],
});
res.send(members);
```

### sequelize ORM find 결과의 실체

위 예시와 같이 find메소드와 추가 옵션을 덧붙여 받은 결과를 `res.send(members)`로 리스폰하고 있다.  
여기서 만약 members를 console.log()로 출력해서 확인할 경우 단순 오브젝트 형태의 정보가 아니라는 점을 확인할 수 있다.

```javascript
// members의 실제 결과

Member {
  dataValues: {
    id: 1,
    name: 'Alex',
    team: 'engineering',
    position: 'Server Developer',
    emailAddress: 'alex@google.com',
    phoneNumber: '010-xxxx-xxxx',
    admissionDate: 2018-12-10T00:00:00.000Z,
    birthday: 1994-11-08T00:00:00.000Z,
    profileImage: 'profile1.png',
    createdAt: 2023-06-02T02:42:07.000Z,
    updatedAt: 2023-06-02T02:42:07.000Z
  },
  _previousDataValues: {
    id: 1,
    name: 'Alex',
    team: 'engineering',
    position: 'Server Developer',
    emailAddress: 'alex@google.com',
    phoneNumber: '010-xxxx-xxxx',
    admissionDate: 2018-12-10T00:00:00.000Z,
    birthday: 1994-11-08T00:00:00.000Z,
    profileImage: 'profile1.png',
    createdAt: 2023-06-02T02:42:07.000Z,
    updatedAt: 2023-06-02T02:42:07.000Z
  },
  uniqno: 1,
  _changed: Set(0) {},
  _options: {
    isNewRecord: false,
    _schema: null,
    _schemaDelimiter: '',
    raw: true,
    attributes: [
      'id',            'name',
      'team',          'position',
      'emailAddress',  'phoneNumber',
      'admissionDate', 'birthday',
      'profileImage',  'createdAt',
      'updatedAt'
    ]
  },
  isNewRecord: false
}
```

원래 members 변수는 위 결과처럼 하나의 객체 형태를 response한다. 여기서 실제 우리가 필요한 데이터는 `dataValues` 프로퍼티이다.  
그럼에도 우리가 `dataValues` 프로퍼티에 직접적으로 접근하지 않고서도 원하는 결과를 얻을 수 있었던 이유는 바로 `send` 메서드가 이를 자동으로 처리해줬기 때문이다.  
자동 처리된 과정은 바로 `.toJSON()` 메서드를 적용하는 것으로, 이는 Member 클래스에서 dataValues 객체를 가져오는 역할을 하는데 만약 `send` 메서드로 모델 클래스를 보내준다면 이 과정의 생략이 가능하다.

### 데이터 추가를 위한 POST 요청

이전 예시들을 통해 ORM 메서드에 대한 설명이 충분했으므로 코드만 보고 파악해보자

```javascript
app.post('/api/members', async (req, res) => {
  const newMember = req.body;
  const member = Member.build(newMember);
  await member.save();
  res.send(member);
});
```

빌드 후 저장 방식으로 새 데이터를 추가하는 것을 확인할 수 있다.

```javascript
app.post('/api/members', async (req, res) => {
  const newMember = req.body;
  const member = await Member.create(newMember);
  res.send(member);
});
```

build와 save 메서드를 합친 `create` 메서드를 써도 동일한 결과를 얻을 수 있다.  
하지만 만약에 빌드 후 수정이 필요한 상황이라면 build, save로 구분하서 쓰면 된다.

### 데이터 수정을 위한 PUT 요청

sequealize의 update 결과로 [수정된 row의 개수, 수정된 row의 data]가 반환된다. 공식 문서에 따르면 결과 배열의 두번째 요소는 postgreSQL에서만 지원한다고 한다.  
어쨌든 `result[0]`은 수정 성공 개수를 의미하므로 아래와 같이 활용한다.

```javascript
app.put('/api/members/:id', async (req, res) => {
  const id = req.params.id;
  const newInfo = req.body;
  const result = await Member.update(newInfo, { where: { id } });
  if (result[0]) {
    res.send({ message: `${result[0]} row(s) is affected` });
  } else {
    res.status(404).send({ message: 'There is no member with id' });
  }
});
```

참고로 변경할 사항 즉 req.body에 보낼 json 데이터에는 모든 컬럼이 아닌 변경할 컬럼과 값만 지정해줘도 된다.

```javascript
// 또 다른 수정 방법
app.put('/api/members/:id', async (req, res) => {
  const id = req.params.id;
  const newInfo = req.body;
  const member = await Member.findOne({ where: { id } });
  if (member) {
    Object.keys(newInfo).forEach((prop) => {
      member[prop] = newInfo[prop];
    });
    await member.save();
    res.send(member);
  } else {
    res.status(404).send({ message: 'There is no member with id' });
  }
});
```

또 다른 수정 방법으로는 위 코드와 같이 수정하고자 하는 데이터를 `findOne`메소드를 불러와 직접적으로 수정한 뒤 다시 저장하는 방법을 취할 수도 있다.

### 데이터 삭제를 위한 DELETE 요청

DELETE의 기능으로 `destroy`메서드를 사용하고 있으며, 데이터 삭제의 결과로 삭제된 row의 수를 출력하여 `update`메서드와 같이 성공 유무의 기준으로 사용하고 있다.

```javascript
app.delete('/api/member/:id', async (req, res) => {
  const id = req.params.id;
  const deletedCount = Member.destroy({ where: { id } });
  if (deletedCount) {
    res.send({ message: `${deletedCount} row(s) deleted` });
  } else {
    res.status(404).send({ message: 'There is no member with id' });
  }
});
```

### MySQL의 JOIN 적용을 위한 준비

게시글 작성 사이트를 예시로 들어보자.  
데이터베이스에는 유저 테이블과 게시글 테이블이 구분되어 있을 것이고, 게시글 테이블을 통해 해당 게시글의 작성자가 누구인지, 좀 더 깊게 들어가서 작성자의 이메일은 무엇인지 알기 위해서는 두 테이블 간 JOIN 작업이 필요할 것이다. sequelize를 통해 두 테이블을 JOIN 해서 정보를 획득하는 방법에 대해 알아보자

#### 두 테이블의 관계 설정

```javascript
// 유저 모델과 포스트 모델 간 1대 다 관계 설정

// models/user.js
static associate(models) {
  this.hasMany(models.Post, {
    sourceKey: 'id',
    foreignKey: 'userId',
  });
}

// models/post.js
static associate(models) {
  this.belongsTo(models.User, {
    targetKey: 'id',
    foreignKey: 'userId',
});
}
```

우선 sequelize의 migrate를 통해 생성된 models 파일을 살펴본다.  
만약 migrate를 통해 User 모델과 Post 모델이 생성되었다고 가정했을 때 각각의 코드에는 `associate`라는 staticmethod가 있을 것이다. 해당 란에 위와 같이 각 모델 파일을 수정해 준다.

위 예시의 경우 유저와 게시글 테이블 간의 관계를 1대 Many로 설정하고 있다. 유저의 입장에서 게시글은 여러 개 작성할 수 있지만 게시글의 입장에서는 본인의 작성자는 단 한명이기 때문이다.

그래서 models/user.js 모델에 `hasMany` 메서드를 적용하여 User의 id와 Post의 userId 필드를 엮어준다.  
그리고 models/post.js 모델의 경우 1대 다 중 '다'에 해당하므로 해당 케이스는 `belongsTo` 메서드를 통해 위와 같이 코드를 작성해준다.

저렇게하면 이제 sequelize의 CRUD 메서드 사용 시 `include` 메서드를 활용하여 JOIN 쿼리를 간접적으로 날릴 수 있게 된다.

예시를 살펴보자

```javascript
// sequelize
const getPosts = async (req, res) => {
  const getPosts = await Post.findAll({
    attributes: {
      exclude: ['userId'],
    },
    include: [
      {
        model: User,
        attributes: ['id', 'username', 'role'],
      },
    ],
    order: [['createdAt', 'DESC']],
  });
};
```

포스트 정보를 GET 하는 함수에서 findAll을 통해 Post테이블 데이터를 모두 불러오며, 이 과정에서 userId 필드를 제외한다. 그 이유는 이후 include 메서드를 통해 User 모델을 불러와 해당 모델에 담긴 필드를 불러오면서 유저 id를 가져올 것이기 때문이다. (굳이 userId에 대한 정보가 중복되어 reponse될 필요가 없기 때문이다.) 마지막으로 게시글 조회 시 게시일 기준으로 내림차순 정렬이 되도록 설정하였다.
