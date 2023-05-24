# 자바스크립트 Express 기본기

## Express로 API 서버 구축하기

우선 Express 구현에 집중하기 위해 사전에 준비된 members.js 파일에 기록된 멤버 정보를 `module.exports`로 전달받는 것으로 데이터베이스 `리소스` 통신을 대신한다.

> `💡 Tip`  
> 데이터베이스에 담긴 각종 데이터를 `Resource(자원, 정보)`라고 부른다.

### 멤버 리소스 가져오기

멤버 리소스를 불러오는 간단한 API를 구현하는 과제에서 아래와 같이 구현했다.

```javascript
// Before
const express = require("express");
const members = require("./members.js");
const app = express();

app.get("/api/members/:id", (req, res) => {
  const id = Number(req.params.id);
  const member = members.filter((member) => member.id === id);
  if (member) {
    res.send(member);
  } else {
    res.status(404).send({ msg: "There is no such member" });
  }
});

app.listen(3000, () => {
  console.log("Server is listening...");
});
```

하지만 위 방식에서 params를 불러오는 방식에서 두 가지 수정이 필요했다.

```javascript
app.get("/api/members/:id", (req, res) => {
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
app.get("/api/members", (req, res) => {
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

app.post("/api/members", (req, res) => {
  const newMember = req.body;
  members.push(newMember);
  res.send(newMember);
});
```

POST 리퀘스트에서 위 예시는 post fetch에서 body의 Content-Type이 `Application/json`임을 가정했다.  
Express에서는 json 타입의 body가 라우트에 들어오기 전 Express의 `미들웨어`라고 불리는 곳에 들러 jsonParser를 통해 전처리하는 과정을 거치도록 되어있다.

### 멤버 정보 수정하기

```javascript
app.put("/api/members/:id", (req, res) => {
  const { id } = req.params;
  const newInfo = req.body;
  const member = members.find((member) => member.id === Number(id));
  if (member) {
    Object.keys(newInfo).forEach((key) => {
      member[key] = newInfo[key];
    });
    res.send(member);
  } else {
    res.status(404).send({ message: "There is no member with the id" });
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
app.delete("/api/members/:id", (req, res) => {
  const { id } = req.params;
  const membersCount = members.length;
  members = members.filter((member) => member.id !== id);
  if (members.length < membersCount) {
    res.send({ message: "Deleted" });
  } else {
    res.status(404).send({ message: "There is no member with the id" });
  }
});
```

현재 members 데이터가 object 형태이므로 id에 해당하지 않는 멤버를 필터링하여 object를 재할당하는 방식으로 멤버 정보를 제거했다.

### nodemon 설치

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

### package scripts 수정을 통해 간단하게 시작하기

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
