# 유튜버 관리 및 사용자 인증 API 구현

이번 블로그 포스트에서는 유튜버 관리 API와 사용자 인증 기능을 Express.js로 구현하는 방법을 설명합니다. 이 API는 다양한 HTTP 요청 방식으로 유튜버 및 사용자 정보를 관리할 수 있게 합니다.

## 1. 동등 비교 연산자와 일치 비교 연산자

### `==` (동등 비교 연산자)
- 비교하려는 피연산자의 자료형이 서로 다를 경우 암묵적으로 형변환이 일어납니다.
  
### `===` (일치 비교 연산자)
- 비교하려는 피연산자의 자료형이 서로 달라도 형변환이 일어나지 않습니다.

## 2. 예외 처리: POST 요청

유튜버 정보를 등록할 때, 필수 필드가 포함되어 있는지를 확인하는 기능을 구성했습니다.

```javascript
const youtubers = new Map();
const fields = JSON.stringify(["channel", "channelTitle", "subscribers", "videoCount"]);

app.post("/youtubers", (req, res) => {
    const reqFields = JSON.stringify(Object.keys(req.body));
    if (fields === reqFields) {
        youtubers.set(++id, req.body);
        res.json({
            message: `${req.body.channelTitle}님, 채널 개설을 환영합니다!`,
        });
    } else {
        res.status(400).send("Request body에 필드가 잘못되었거나, 필요한 필드가 없습니다.");
    }
});
```

## 3. HTTP 상태 코드
* 2xx (성공):
    - 200: 조회/수정/삭제 성공
    - 201: 등록 성공
* 4xx (클라이언트 오류):
    - 400: 클라이언트가 잘못된 데이터를 전달함
    - 404: 찾는 리소스가 없음
* 5xx (서버 오류):
    - 500: 서버가 크리티컬한 오류를 맞았을 때

## 4. 유튜버 미니 프로젝트 설계
### 기능 요구사항
* 회원 기능:
    - 가입
    - 탈퇴
    - 개인 정보 수정
* 채널 기능:
    - 생성
    - 삭제
    - 채널 정보 수정
    - 조회

### API 엔드포인트
* 로그인: POST /login
    - 요청: body (id, pw)
    - 응답: ${name}님 환영합니다 → 메인페이지 (리다이렉트)
* 회원가입: POST /join
    - 요청: body (id, pw, name)
    - 응답: ${name}님 환영합니다. → 로그인 페이지 (리다이렉트)
* 마이페이지: GET /users/:id
    - 요청: URL (id)
    - 응답: id, name
* 회원 탈퇴: DELETE /users/:id
    - 요청: URL (id)
    - 응답: ${name}님 아쉽지만 다음에 또 만나요 or 메인 페이지 (리다이렉트)

## 5. 코드 구현 (TypeScript)
```javascript
import express from "express";
const app = express();
app.listen(3000);
app.use(express.json());

interface User {
    id: string;
    pw: string;
    name: string;
}

const db = new Map<string, User>();
const requiredFields = JSON.stringify(["id", "pw", "name"]);

// 로그인
app.post("/login", (req, res) => {
    const { id }: User = req.body;
    const user = db.get(id);

    if (user) {
        res.json({
            ...user,
            message: `${user.name}님, 어서오세요`,
        });
    } else {
        res.status(404).send("해당 계정을 찾을 수 없습니다.");
    }
});

// 회원가입
app.post("/join", (req, res) => {
    const { id, name }: User = req.body;
    const fields = JSON.stringify(Object.keys(req.body));

    if (requiredFields === fields) {
        db.set(id, req.body);
        res.status(201).json({
            ...req.body,
            message: `${name}님 환영합니다.`,
        });
    } else {
        res.status(400).send("잘못된 값을 입력하였습니다.");
    }
});

// 회원 개별 조회
app.get("/users/:id", (req, res) => {
    const id: string = req.params.id;
    const user = db.get(id);
    if (user) {
        res.json({
            id: user.id,
            name: user.name,
        });
    } else {
        res.status(404).send("존재하지 않는 id입니다.");
    }
});

// 회원 탈퇴
app.delete("/users/:id", (req, res) => {
    const id: string = req.params.id;
    const user = db.get(id);
    if (user) {
        db.delete(id);
        res.send(`${user.name}님 아쉽지만 다음에 또 만나요.`);
    } else {
        res.status(404).send("존재하지 않는 id입니다.");
    }
});
```

![image](https://github.com/user-attachments/assets/6bf0db96-7a8f-464f-96ac-75d6c05f90b8)
