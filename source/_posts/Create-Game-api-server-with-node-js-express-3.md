---
title: Create Game api server with node.js express-3
date: 2020-01-06 11:28:18
updated:
categories:
   - nodejs
   - express
tags:
   - express
   - apiserver
---

router 를 분리하고 각 라우터에 대한 controller 를 만들자.

<!-- more -->
<!-- toc -->

## 기본라우팅
라우팅은 URI 또는 경로 및 특정한 HTTP 요청 메소드인 특정 엔드포인트에 대한
클라이언트 요청에 애플리케이션이 응답하는 방법을 결정하는 것을 말한다.

라우트 정의에는 다음과 같은 구조가 필요하다.
``` bash
app.METHOD(PATH, HANDLER)
```
METHOD는 get, post, put, delete의 HTTP 요청 메소드
PATH는 서버에서의 경로
HANDLER는 라우트가 일치할 때 실행되는 함수

[기본 라우팅](https://expressjs.com/ko/starter/basic-routing.html) 과 [라우팅](https://expressjs.com/ko/guide/routing.html) 에 자세하게 나와있다.

## 그럼 왜 라우팅을 분리하는가
보통 애플리케이션 레벨 미들웨어나 라우터 레벨 미들웨어로 요청 메소드를 구현하게 되는데,
애플리케이션의 규모가 커지게 되면, 라우터의 분기가 많아지게 되고, 프로덕션 레벨에서 
유지보수 및 오류 발생 시 추적을 쉽게 하기 위해서다.

### 라우터 레벨 미들웨어로 분리
[라우터 레벨 미들웨어](https://expressjs.com/ko/guide/using-middleware.html#middleware.router)로 분리해보자.

제일 먼저 로그인 관련 요청을 만들어야 하므로, /auth 경로에 대한 라우터를 만들자.
로그인 관련 모든 요청은 /auth 에서 시작한다고 가정한다.

1. app.js에 라우터 미들웨어를 바인딩 한다.
routes 폴더 자체를 import 하게 되면 폴더 내에서 index.js를 먼저 찾는다.
{% codeblock app.js lang:objc %}
import indexRouter from './routes';
...
app.use('/', indexRouter);
{% endcodeblock %}

2. 따라서 index.js에 라우터 미들웨어를 만들자.
아래는 /auth 경로에 대한 미들웨어 등록
{% codeblock routes/index.js lang:objc %}
import express from 'express';
import authRouter from './auth.route';

const routeHandler = express.Router();

routeHandler.use('/auth', authRouter);

export default routeHandler;
{% endcodeblock %}

3. path별 요청에 따른 method들을 모듈화 해준다.
{% codeblock routes/auth.route.js lang:objc %}
import express from 'express';
import authController from '../controller/auth.controller';

const authRouter = express.Router();

authRouter.route('/create')
    .post(authController.createUser);

export default authRouter;
{% endcodeblock %}

4. /auth/create 로 요청이 들어오면 실행되는 핸들러를 별도 controller로 분리한다.
{% codeblock controller/auth.controller.js lang:objc %}
exports.createUser = (req, res, next) => {
    const { userid } = req.body;
    const { password } = req.body;
    return res.json({
        message: 'success'
    });
};
{% endcodeblock %}

5. 최종 변경 된 폴더 구조
{% asset_img routing.PNG This is an example image %}
{% asset_img "spaced routing.PNG" "spaced title" %}

### postman으로 라우터 미들웨어 테스트
서버를 실행하자.
``` bash
$ npm start
[nodemon] 2.0.2
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `babel-node src/bin/www`
Server Listening on 3000
```

현재 서버에 추가한 http 경로는 /auth/create 밖에 없다.
그 외는 404 error 를 리턴한다.

postman으로 아래와 같이 send를 서버로 보내면,
{% asset_img post.PNG This is an example image %}
{% asset_img "spaced post.PNG" "spaced title" %}

req.body에 대한 예외처리는 되어 있지 않으므로,
message:success 응답코드가 정상적으로 리턴됨을 알 수 있다.
{% asset_img post2.PNG This is an example image %}
{% asset_img "spaced post2.PNG" "spaced title" %}

서버에서 요청 경로에 대한 로그
``` bash
POST /auth/create 200 16.240 ms - 21
```

그 외 다른 http path를 테스트 해보면, /auth/create/1 
``` bash
Error: Sorry cant find that!
    at D:\test-api-server\src/app.js:17:17
    at Layer.handle [as handle_request] (D:\test-api-server\node_modules\express\lib\router\layer.js:95:5)
    at trim_prefix (D:\test-api-server\node_modules\express\lib\router\index.js:317:13)
    at D:\test-api-server\node_modules\express\lib\router\index.js:284:7
    at Function.process_params (D:\test-api-server\node_modules\express\lib\router\index.js:335:12)
    at Immediate.next (D:\test-api-server\node_modules\express\lib\router\index.js:275:10)
    at Immediate._onImmediate (D:\test-api-server\node_modules\express\lib\router\index.js:635:15)
    at processImmediate (internal/timers.js:441:21)
    at process.topLevelDomainCallback (domain.js:130:23)
POST /auth/create/1 404 16.541 ms - 21
```
서버에서 해당 요청 경로에 대한 정의가 되어 있지 않으므로,
에러스택을 콘솔로 찍으면서 404 에러를 리턴한다.

### 번외 : babel 로 빌드 후 테스트
``` bash
$ npm run build
> test-api-server@0.0.0 build D:\test-api-server
> babel src -w -d dist

Successfully compiled 4 files with Babel.
```
build 된 파일들의 폴더 구조는 src와 같다.
{% asset_img babel.PNG This is an example image %}
{% asset_img "spaced babel.PNG" "spaced title" %}

서버 실행 후, /auth/create로 요청을 보내면,
``` bash
D:\test-api-server>node dist/bin/www
Server Listening on 3000
POST /auth/create 200 15.530 ms - 21
```

잘 동작함을 알 수 있다.

## 로그인 라우터 기능 구현
아직 database 저장이나 session 을 이용한 로그인 / 로그아웃 기능은 없으므로,
기본 구조만 만들어보자.

routes/auth.route.js 는 아래와 같이 작성한다.
{% gist 60aeeb55653c63eaff0b55899b4388f3 %}

controller/auth.controller.js 는 아래와 같이 작성한다.
{% gist a676cec6bdec61d1eaf7a4fc7db86ce2 %}

/auth/login 으로 요청이 들어오면, UserInfo.userid 와 UserInfo.password로
비교하여 로그인 성공/실패 redirect를 리턴한다.

/auth/create는 아무런 동작을 하지 않는다. 

다음 포스팅은 express-session 과 redis, passport를 이용해서 로그인을 마무리한다.

Done.