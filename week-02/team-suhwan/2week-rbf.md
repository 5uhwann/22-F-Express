### Express.js

서버 로직은 매우 복잡한데, 우리는 비즈니스 로직과 같은 애플리케이션을 정의하는 코드에 집중하고 싶다!

Express.js는 비즈니스 처리에 집중하는데 도움을 주는 프레임워크이다.

```jsx
import express from "express";
const app = express();
const server = http.createServer(app);
```

- 상수 `app` 에 express 프레임워크의 로직이 담김.
- `app` 은 유효한 요청 핸들러이기 때문에 `app` 을 사용해 서버 생성 가능

- 들어오는 요청을 분석하기 위해 아래의 코드를 짬.
    - body를 추출하기 위해 직접 데이터 이벤트와 end 이벤트를 사용하고 난 후 마지막에는 문자열로 변환했던 버퍼를 생성해야 했음

```jsx
req.on('data', (chunk)=>{
            console.log(chunk);
            body.push(chunk);
        });
        return req.on('end', ()=>{
            const parsedBody = Buffer.concat(body).toString();
            ...
            });
        });
```

- 데이터를 취급하거나 파싱하는 방법을 내부에 포함하고 있는 것은 아니지만 파싱을 대신해 주기 위한 프로젝트 내부에 쉽게 연결하는 다른 패키지를 쉽게 설치하도록 도와줌
    - 파싱: 문서나 HTML 등 어떤 큰 자료에서 내가 원하는 정보만 가공하고 추출해서 원할 때 불러올 수 있게 하는 것
        - 특정 패턴이나 순서로 추출해서 정보로 가공할 수 있음
- 특징
    - 매우 유연하며, 특별한 기능을 과도하게 추가하지 않음
    - 애플리케이션을 구축하거나 유입되는 요청들을 처리함에 있어서 특별한 방법을 제공해 높은 확장성을 가진다 → Express를 대상으로 구축된 제 3자 패키지의 수는 수천개 존재하며 쉽게 Node Express 애플리케이션에 추가할 수 있음
    - 깔끔한 코드를 작성하는 데 도움을 줌

### Middleware

**Middleware란?**

클라이언트에게 요청이 오고 그 요청을 보내기 위해 응답하려는 중간(미들)에 목적에 맞게 처리를 하는 메소드를 의미한다.

서버에 들어오는 요청을 express.js의 다양한 함수를 통해 자동으로 이동하게 한다.

하나의 함수에서 모든 걸 처리하게 하지 않고 분리한다.

```jsx
app.use((req, res, next) => {
  console.log("In the middleware1!");
  next();
});
app.use((req, res, next) => {
  console.log("In the middleware2!");
});

```

- `(req, res, next) ⇒ {};` 의 형태를 가진 함수는 모두 middleware로 사용 가능하다.
- 파라미터로 받은 `next`는 다음 middleware에 대한 엑세스 권한을 갖는 함수이며, 다음 middleware로 이동하는 역할을 한다.

**Middleware 동작 원리**

![https://user-images.githubusercontent.com/106325839/191742334-295f3971-dfce-4b95-ae7c-b93b081f6584.png](https://user-images.githubusercontent.com/106325839/191742334-295f3971-dfce-4b95-ae7c-b93b081f6584.png)

Middleware는 요청이 들어온 순간부터 순차적으로 실행되어 응답이 마무리 될 때까지 동작 사이클이 실행된다.

**미들웨어를 활용한 제어**

미들웨어를 활용해 무엇이 표시되는지 제어할 때는 `next()` 의 호출 여부 뿐만 아니라, 미들웨어의 순서도 중요하다. → Node.js 는 스크립트를 위에서 아래로 읽기 때문

아래는 미들웨어의 실행 순서에 관한 코드와 간단한 퀴즈이다.

```jsx
app.use("/", (req, res, next) => {
  console.log("a");
  next();
});
app.use("/add-product", (req, res, next) => {
  console.log("In another Middleware1");
  res.send('<h1>The "Add-Product" Page</h1>');
});
app.use("/", (req, res, next) => {
  console.log("In another Middleware2");
  res.send("<h1>Hello from Express!</h1>");
});
// Q1. localhost:3000 -> localhost:3000/add-product 어떤 화면이 나오고, 콘솔창은 어떻게 출력이 될까?

// Answer: 콘솔창에서는 a, In anoter Middleware2, a, In another Middleware1 순으로 출력되고, 최종 화면에는 The "Add-Product" Page 가 출력된다.
```

**요청 분석**

기본적으로 request는 들어오는 request body에 대한 내용을 분석하려 하지 않는다.

→ request body 분석을 위한 middleware 추가 구현

```jsx
//urlencoded()가 미들웨어 함수 포함, next()실행
app.use(express.urlencoded({ extended: false }));
app.use("/", (req, res, next) => {
  console.log("a");
  next();
});
```

- 일반적으로 경로 처리 미들웨어 전에 둔다. → 요청이 어느 경로로 향하든 request body 분석이 이뤄지도록 함.
- `{extended : false}`: 비표준 request body 분석 여부
- body-parser 가 수행. (express 4.x.x 부터 내장으로 body-parser 장착)

<aside>
💡 urlencoded({extended: true or false })는 어떤 옵션일까?

</aside>

자바스크립트에서 데이터를 주고받고 읽을 때는 객체 형태를 띄고, 실제로 데이터를 주고 받을 때도 객체 형태를 선호한다. (따라서 여러 과정을 거쳐 파싱을 거쳐야 하는 것이다 )

extended 옵션의 경우, true일 경우, 객체 형태로 전달된 데이터 내에서 또 다른 중첩된 객체를 허용한다는 말이며, false인 경우에는 허용하지 않는다는 말이다.

그렇다면 true와 false로 세팅했을 때 차이가 무엇일까?

**bodyParser 미들웨어의 여러 옵션 중에 하나로 false 값일 시 node.js에 기본으로 내장된 queryString, true 값일 시 따로 설치가 필요한 npm qs 라이브러리를 사용한다.**

queryString 과 qs 라이브러리 둘 다 url queryString 을 파싱해주는 같은 맥락에 있으나 qs가 추가적인 보안이 가능한, 말 그대로 extended 확장된 형태이다. 기본이 true 값이니 qs 모듈을 설치하지 않는다면 아래와 같이 false 값으로 따로 설정을 해주어야 한다.

즉 qs는 추가적인 보안기능이 있는 모듈로서 필요하다면 모듈 설치후에 사용하면 된다.

아무래도 중첩된 객체를 허용한다는 말은 추가적인 보안기능이 있는 것에 대한 일부 사용에 대한 설명인 것으로 추측된다.

### 미들웨어 실행 제한

현재 `app.use()` 는 POST 요청 뿐만 아니라 GET요청에 대해서도 실행된다.

- `app.get()`
    - `app.use()` 와 기본적으로 동일하지만, GET 요청에만 작동
    - GET 요청만 확인하는 것이 아니라 정확한 경로인지도 확인
- `app.post()`
    - `app.use()` 와 기본적으로 동일하지만, POST 요청에만 작동
    - POST 요청만 확인하는 것이 아니라 정확한 경로인지도 확인
    

## 라우팅

주소(IP)를 정하고, 경로를 선택하고, 패킷을 전달하는 것

→ 목적에 따라 효율적인 프로토콜을 선택하는 것

- 네트워크에서 패킷을 보낼 때 목적지까지 갈 수 있는 여러 경로 중 하나의 경로를 설정해주는 과정
- 파일 크기가 커짐에 따라서 일반적으로 라우팅 코드를 여러 파일로 나누는 것이 좋음 → **유지보수를 위해**서 나눈다, 파일 크기가 커지면 실행에 시간이 오래걸림
    - Express.js는 라우팅을 다른 파일에 위탁하는 방법을 제공
    - 라우팅에 의한 결과 = **라우트**
- 아웃소싱 라우트들은 시작 경로를 공유하는 경우가 있음
    - 아웃소싱 → routes/admin.js 에서 /admin/add-product 라고 써야하는데 /admin을 계속 써야할 수 없으니까 get, post를 각각 아웃소싱됐다고 말함
    - 주어진 파일의 모든 루트가 app.js 파일로 위탁하는 데 사용할 공통 경로를 앞에 추가할 수 있음 → 모든 루트에 대해 반복해서 입력할 필요가 없음
    
    ```jsx
    app.use('/admin',adminRoutes);
    ```
    
    → admin으로 시작하는 라우트만 adminRoutes 파일로 들어감
    
    → /add-product는 /admin/add-product 라우트에 매칭됨
    
    ~> /admin/add-product여야만 폼이 나타남
    

### 오류 페이지

요청처리(미들웨어 실행)는 작성 순서대로 처리하기 때문에 요청을 처리할 미들웨어가 없다면 요청이 처리되지 않는다.

→ 오류 페이지 전송은 맨 아래 모든 요청을 처리하는 미들웨어를 추가

```jsx
app.use((req, res, next) => {
  res.status(404).send("<h1>Page not found</h1>");
});

```

- `status()`:상태코드 설정

### Response Class 에러 코드

- 400번대 에러 코드는 클라이언트 오류로, 클라이언트 요청을 처리할 수 없어 오류가 발생하는 것이다.
- 500번대 에러 코드는 서버에서 처리를 하지 못해 오류가 발생하는 것이다. 클라이언트의 요청에는 문제가 없다.

400, 500번대 에러 코드 중에서 5가지씩만 정리해 보았다.

### 400번대 에러 코드

- 400 - Bad Request

:  클라이언트의 요청 구문이 잘못됨.

- 401 - Unautorized

: 요청 처리를 위해 HTTP인증 정보가 필요함을 알려줌. 최초 요청에는 인증 다이얼로그를 표시하고, 두 번째는 인증 실패 응답을 보냄.

- 403 - Forbidden

접근 금지 응답. 서버 파일 디렉토리 목록 표시 요청 및 관리자 페이지 접근 등을 차단하는 경우의 응답.

- 404 - Not Found

클라이언트가 요청한 리소스가 서버에 없음.

- 405 - Method Not Allowed

허용되지 않는 HTTP 메소드를 사용함.

### 500번대 에러 코드

- 500 - Internal Server Error

서버에서 클라이언트 요청을 처리 중에 에러가 발생함.

- 503 - Service Unavailable

서버가 일시적으로 요청을 처리할 수 없음. 서버가 과부하 상태이거나 점검 중이므로 요청을 처리할 수 없음을 알려줌.

- 504 - Gateway Timeout

서버를 통하는 게이트웨이에 문제가 발생하여 시간이 초과됨.

- 505 - HTTP Version Not Supported

해당 HTTP 버전에서는 지원되지 않는 요청임을 알려줌.

**send**

: 응답 및 any유형의 본문을 첨부가능

- html content type
- 기본 응답 헤더 : text/html
- Express.js에서 입력한 유형에 따라서 HTML content type을 인식하고 선택
- setHeader를 사용해서 수동으로 설정 가능
- write보다 쉬움

### 경로 필터링

일반적으로 라우터파일 속 미들웨어들은 시작 경로를 공유하는 경우가 많다.

```jsx
router.get("/admin/add-product", (req, res, next) => {
  console.log("a");
  next();
});
app.use("/admin/add-product", (req, res, next) => {
  console.log("b");
  res.send("<form></form>");
});

```

- 공통되는 라우트 경로를 빼서 app.js에서 필터를 할 수 있다.
`app.use("/admin", adminRoutes);`

### HTML 서비스

- `res.sendFile()` : 파일을 회신함, 콘텐츠 타입을 자동으로 응답 헤더로 설정

**경로 설정**

- 상대경로 설정을 위해 Node.js에서 제공하는 path모듈을 사용
    - `__dirname` : 절대경로를 프로젝트 폴더로 고정
    - 쉼표로 조합하면서 최종 파일 경로 생성
    
    ```jsx
    res.sendFile(path.join(__dirname, "../", "views", "shop.html"));
    ```
    
    - `path.join()` : 다른 파일 시스템 경로를 운영체제 맞게 변환
- 절대경로: 최상위 위치에서 시작
- 상대경로: app.js에서 시작한다고하면 app.js를 중심으로 찾아가는 것 (나 기준)

### 헬퍼 함수

```jsx
const path = require('path');

module.exports = path.dirname(process.mainModule.filename);
```

__dirname으로 절대 경로를 고정하고 시작하지 않고 다른 파일에 위 코드처럼 미리 경로를 지정하여 원래 코드에 임포트하여 사용할 수 있게 함.