---
title: Create Game api server with node.js express-4
date: 2020-01-07 12:47:46
updated:
categories:
   - nodejs
   - express
tags:
   - express
   - apiserver
---

mongoDB를 이용하여, session 저장과 유저 정보를 저장한다.

<!-- more -->
<!-- toc -->

## install mongoDB
Windows10 환경에서 개발진행하므로, 
[MongoDB Download Center](https://www.mongodb.com/download-center/community)에서 MSI를 다운받는다. 현재 시점에서 Version 은 4.2.2 이다.
설치부터 dbpath , 확인까지는 [여기](https://javacpro.tistory.com/64) 에 잘 나와있다.

## install npm package 
세션 및 DB 저장을 위해, 아래 패키지들을 설치하고 package.json을 확인하자.
```
$ npm install express-session connect-mongodb-session mongoose --save

// package.json
"dependencies": {
    "connect-mongodb-session": "^2.2.0",
    "express-session": "^1.17.0",
    "mongoose": "^5.8.4",
```
[express-session](https://www.npmjs.com/package/express-session) : 세션관리 미들웨어
[mongoose](https://mongoosejs.com/) : 가장 유명한 MongoDB ODM(Object Document Mapping)
[connect-mongodb-session](https://github.com/mongodb-js/connect-mongodb-session#readme) : 세션정보를 MongoDB에 저장. connect-mongo 를 좀 더 경량화 시킨 패키지라서 선택했는데 다양한 옵션을 더 주고 싶다면 connect-mongo 를 쓰면된다.

## 유저 정보 저장
/auth/create 로 요청이 들어올 때, mongoDB에 저장하는 기능을 구현한다.
예외처리가 완벽하지 않은 코드이지만, 기능 구현에 초점을 두었다.

### mongoose connect
testdb로 connection 을 만들어준다.
단, connect가 성공하더라도 아직 데이터가 저장되기 전이므로 db 생성은 되지 않는다.
{% gist cb4f2b596ef386436313c72c05c578a2 %}

app.js 에 서버 실행 시, mongodb 에 접속할 수 있도록 코드를 추가한다.
{% codeblock app.js lang:objc %}
import mongodb from './db/mongo.connect';

const app = express();
mongodb();
{% endcodeblock %}

아래 서버 실행으로 connect 가 정상임을 알 수 있다.
``` bash
$ npm start

> nodemon src/bin/www --exec babel-node

[nodemon] 2.0.2
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `babel-node src/bin/www`
Server Listening on 3000
connect mongodb
```

### mongoose : create user schema
testdb 에 저장할 user schema 를 생성하고 , 
{% gist 92a2ef8df58b1522c568007e4bacf369 %}

생성한 model을 매개변수로 받아 데이터 저장 및 조회하는 클래스를 만든다.
아래 create, findOne은 mongodb 의 메소드를 매핑한 함수이다.
{% gist 11c93cc334931a44db014f4d6d52f8c2 %}

### controller 의 handler 수정
[3편](https://akanamed.github.io/2020/01/06/Create-Game-api-server-with-node-js-express-3/)에서 만들었던 auth.controller.js에 mongodb에 넣고 조회하는 로직을 추가한다.
authcontroller.createUser 와 authcontroller.login 부분이 UserRepository의 메소드를 호출해서
리턴하도록 수정되었다.
{% gist 5d8df091d0d5f3517a00a2e4c9177c8b %}

findOne함수에서 주의해야 할 점은, 매개변수 user.userid에 값이 없을 때,
db에서 제일 첫번째 값을 불러오므로, 리턴 결과값이 null 인지 체크를 해서 
null 이라면 fail로 넘겨줘야한다.

### postman으로 테스트
서버 실행 후, postman으로 /auth/create , /auth/login 을 요청하면
아래와 같이 정상작동 됨을 알 수 있다.
{% asset_img post3.PNG This is an example image %}
{% asset_img "spaced post3.PNG" "spaced title" %}
{% asset_img post4.PNG This is an example image %}
{% asset_img "spaced post4.PNG" "spaced title" %}

서버 로그
``` bash
POST /auth/create 200 21.691 ms - 79
abcde, 123456
{
  _id: 5e141648e0936e75a4d4c8b3,
  userid: 'abcde',
  password: '123456',
  __v: 0
}
GET /auth/login 302 18.499 ms - 41
GET /auth/login/success 200 0.532 ms - 27
```

mongodb - testdb collection 확인
{% asset_img mongodb.PNG This is an example image %}
{% asset_img "spaced mongodb.PNG" "spaced title" %}

## Session 이란
[쿠키와 세션의 특징 및 차이](https://hahahoho5915.tistory.com/32) 에 잘 나와있으며,
요약하면 세션은 접속중인 웹서버에 저장되며, 속도는 쿠키보다 느리지만 보안은 쿠키보다 좋다.

### mongoDBStore 생성
mongodb에 세션을 저장하기 위해, 아래와 같이 코드를 작성한다.
{% gist 7113ce089a91709d5c835c234b22d032 %}

### express-session 미들웨어 등록
{% codeblock app.js lang:objc %}
import { mongoStore, session } from './db/mongo.session';

app.use(session({
    secret: 'ThIsIsMy$eCrEt',
    cookie: {
        maxAge: 1000 * 60
    },
    store: mongoStore,
    resave: false,
    saveUninitialized: false,
    unset: 'destroy'
}));
{% endcodeblock %}

express-session 미들웨어는 store 옵션이 없으면 메모리에 저장이 되며,
store 에 redis, mongodb 의 스토리지 방식과 파일 방식으로 저장할 수 있다.
resave는 클라이언트가 접속할 때마다 sid를 새로 발급할 것인지의 여부. false로 지정
saveUninitialized는 세션이 store에 저장되기 전에 초기화 되지 않는 상태로 저장하는 옵션.
db에 값이 있을 때만 세션을 저장하므로, false로 지정해준다.
unset 옵션은 기본값이 'keep'이지만 'destroy' 옵션을 줘서 logout 테스트 확인용으로 
/auth/logout 요청 시 delete session; 으로 db에서 삭제하는 용도로 지정해줬다.
expire 타임은 1분으로 지정했으며, 1분이후에는 자동으로 db에서 삭제된다.

### 요청 메소드 수정
auth.route.js 에 /logout 테스트용으로 추가해준다.
{% codeblock auth.route.js lang:objc %}
authRouter.route('/logout')
    .get(authController.logout);
{% endcodeblock %}


auth.controller.js에 /auth/login 요청 시, db에 저장된 유저라면, 
req.session.user에 값을 저장하고 로그인 성공 응답을 보내준다.
그리고, 세션이 mongodb에 저장되어 있을 때, logout요청이 들어오면 delete 로 세션을 삭제한다.
{% codeblock auth.controller.js lang:objc %}
exports.login = (req, res, next) => {
    const user = req.body;
    console.log('%s, %s', user.userid, user.password);
    UserRepository.findOne(user.userid)
        .then((searchUser) => {
            if (searchUser === null) {
                return res.redirect('/auth/login/fail');
            }
            console.log(searchUser);
            req.session.user = user;
            return res.redirect('/auth/login/success');
        })
        .catch((error) => {
            console.log(error);
            res.redirect('/auth/login/fail');
        });
};

exports.logout = (req, res, next) => {
    delete req.session;
    return res.json({
        message: 'success logout'
    });
};
{% endcodeblock %}

### test
db에 저장되어 있는 계정으로 /auth/login 요청을 하면, testdb/sessions 에 
세션정보가 저장된다.

{% asset_img session.PNG This is an example image %}
{% asset_img "spaced session.PNG" "spaced title" %}

req.session 에 세션 정보가 저장되어 있으며, 
로그인 성공 시 req.session.user 에 저장했던 값도 볼 수 있다.

Done.