# 유튜버 관리 API 구현

이 블로그 포스트에서는 유튜버를 관리하는 간단한 API를 Express.js를 사용하여 구현하는 방법에 대해 설명하겠습니다. 이 API는 다음과 같은 기능을 포함하고 있습니다.

## 1. Map 객체를 JSON으로 변환

유튜버 정보를 `Map` 객체에 저장하고, 이를 JSON 형식으로 변환하여 클라이언트에 응답하는 기능을 구현했습니다.

```javascript
const db = new Map();

app.get("/youtubers", (req, res) => {
    res.json(Object.fromEntries(db));
});
```

## 2. Array.prototype.forEach()와 Array.prototype.map() 이해하기
### forEach
forEach 메서드는 배열의 각 요소에 대해 주어진 작업을 수행하지만, 반환값은 undefined입니다.

```javascript
arr.forEach(callbackFn[, thisArg]);
```

```javascript
arr.forEach((element) => {
    console.log(element);
});
```

### map
map 메서드는 배열의 각 요소를 변형하여 새로운 배열을 반환합니다.

```javascript
arr.map(callbackFn[, thisArg]);
```

```javascript
const doubled = arr.map((element) => element * 2);
```

## 3. 개별 유튜버 삭제
유튜버 ID를 통해 개별 유튜버를 삭제하는 엔드포인트를 만들었습니다.

```javascript
app.delete("/youtubers/:id", (req, res) => {
    const id = +req.params.id;
    const youtuber = youtubers.get(id);
    if (youtuber) {
        youtubers.delete(id);
        res.send(`${youtuber.channelTitle}님, 아쉽지만 다음에 또 만나요`);
        console.log(youtubers);
    } else {
        res.status(404).send(`요청하신 ${id}번 유튜버를 찾을 수 없습니다.`);
    }
});
```

## 4. 전체 유튜버 삭제
모든 유튜버 정보를 삭제하는 엔드포인트도 구현하였습니다.

```javascript
app.delete("/youtubers", (req, res) => {
    if (youtubers.size) {
        youtubers.clear();
        res.send(`계⭐정⭐폭⭐파`);
    } else {
        res.status(404).send("삭제할 계정이 없습니다.");
    }
});
```

## 5. 유튜버 정보 수정
유튜버의 채널명을 수정하는 기능도 추가했습니다.

```javascript
app.put("/youtubers/:id", (req, res) => {
    const id = +req.params.id;
    const youtuber = youtubers.get(id);
    if (youtuber) {
        const preChannelTitle = youtuber.channelTitle;
        const newChannelTitle = req.body.channelTitle;
        youtuber.channelTitle = newChannelTitle;
        youtubers.set(id, youtuber);
        res.send(`채널명 '${preChannelTitle}' 이(가) '${newChannelTitle}' (으)로 변경되었습니다.`);
    } else {
        res.status(404).send(`요청하신 ${id}번 유튜버를 찾을 수 없습니다.`);
    }
});
```

## 6. HTTP 상태 코드 개요
API에서 사용할 HTTP 상태 코드의 범주를 정리해 보았습니다.

* 1xx (정보): 요청을 받았으며 프로세스를 계속합니다.
* 2xx (성공): 요청을 성공적으로 받았으며 인식했고 수용하였습니다.
* 3xx (리다이렉션): 요청 완료를 위해 추가 작업 조치가 필요합니다.
* 4xx (클라이언트 오류): 요청의 문법이 잘못되었거나 요청을 처리할 수 없습니다.
* 5xx (서버 오류): 서버가 명백히 유효한 요청에 대해 충족을 실패했습니다.

## 7. 리팩토링
리팩토링은 소프트웨어의 코드 내부 구조를 변경하는 것으로, 다음과 같은 이유로 필요합니다.

* 가독성 증진
    - 성능 향상
    - 안정성 증가

* 언제 해야 할까?
    - 에러(문제점)가 발견되었을 때
    - 기능을 추가하기 전
    - 코드를 읽기 힘들 때
