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

서버 API에서는 `모델`이 곧 `테이블`을 의미하고, `속성`이 곧 `테이블의 컬럼명`을 뜻한다.  
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
