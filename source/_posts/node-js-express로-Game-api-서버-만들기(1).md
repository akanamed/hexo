---
title: node.js+express로 Game api 서버 만들기(1)
date: 2019-12-30 10:47:11
updated:
categories:
   - nodejs
   - express
tags:
   - express
   - apiserver
---

Game api 서버를 만들어보자.
[공식 홈페이지](https://nodejs.org/ko/)에서 Node.js 최신버전을 다운한다.
이 글을 쓰는 현재 공홈 최신 LTS 버전은 12.14.0 이지만,
난 로컬에 이미 설치한 버전은 10.16.3 이므로 그대로 진행한다.
( node.js 버전업은 나중에... )

<!-- more -->
<!-- toc -->

## server-side framework 만들기 with express-generator

Express 애플리케이션 생성기라고 불리우는, express-generator 를 이용해 서버를 만들게 되면,
Server-side 용으로 쓰기엔 불필요한 파일(템플릿 엔진 등 )들이 많아서 별도의 작업이 필요한데,
[해당 링크](https://www.freecodecamp.org/news/how-to-enable-es6-and-beyond-syntax-with-node-and-express-68d3e11fe1ab/) 를 다시 재해석해서 프레임워크를 만들어보자.

### 프로젝트 생성

express 생성기를 글로벌 패키지로 설치하자.
``` bash
$ npm install -g express-generator
```
혹은 npm 5.3 버전 이상부터는 npx 커맨드가 지원되는데,
[npx](https://geonlee.tistory.com/32)는 글로벌 설치가 아니라 1회성 설치,
즉 최신 버전에 해당하는 패키지를 설치하여 실행하고, 실행 후에 패키지를 제거한다고 나와있다.
``` bash
$ npx express-generator
```

[Express](https://expressjs.com/ko/starter/generator.html) 에는 템플릿 엔진을 사용한 가이드가 있지만,
server-side 로 만들 예정이므로, 아래처럼 커맨드를 입력해서 생성하자.
``` bash
$ express test-api-server --no-view

   create : test-api-server\
   create : test-api-server\public\
   create : test-api-server\public\javascripts\
   create : test-api-server\public\images\     
   create : test-api-server\public\stylesheets\
   create : test-api-server\public\stylesheets\style.css
   create : test-api-server\routes\
   create : test-api-server\routes\index.js
   create : test-api-server\routes\users.js
   create : test-api-server\public\index.html
   create : test-api-server\app.js
   create : test-api-server\package.json
   create : test-api-server\bin\
   create : test-api-server\bin\www

   change directory:
     > cd test-api-server

   install dependencies:
     > npm install

   run the app:
     > SET DEBUG=test-api-server:* & npm start

```

### 프로젝트 세팅 및 변경
템플릿 엔진관련 폴더는 생성되지 않고 만들어졌다.
test-api-server 로 이동 후 npm install 커맨드를 실행한다.
public 폴더는 제거하고, src 폴더 생성 후 bin , routes 폴더를 src 폴더 밑으로 이동한다.
그리고 routes 폴더의 users.js 도 제거한다.

src 폴더를 생성한 이유는 babel 이나 webpack 을 이용해 빌드를 해서, 
dist 폴더로 구분하기 위해서다.

여기까지의 폴더 구조는 아래와 같다.
{% asset_img test-api-1.PNG This is an example image %}
{% asset_img "spaced test-api-1.PNG" "spaced title" %}

#### 코드 수정
이제 코드를 수정하자. 
src/bin/www 파일을 열어서 아래와 같이 app모듈의 참조 경로를 수정해준다.
{% codeblock src/bin/www lang:objc %}
var app = require('../../app');
{% endcodeblock %}

package.json 의 npm script 경로를 아래와 같이 바꿔준다.
``` bash
"scripts": {
    "start": "node src/bin/www"
  },
```
app.js 파일 수정
아래에 사용하지 않는 코드는 다 지워준다.
{% codeblock src/bin/app.js lang:objc %}
var usersRouter = require('./routes/users');
app.use(express.static(path.join(__dirname, 'public')));
app.use('/users', usersRouter);
{% endcodeblock %}

그리고 템플릿 엔진 사용하지않고 프로젝트를 만들었기 때문에,
에러관련 처리 코드도 추가해줘야한다.
{% codeblock src/bin/app.js lang:objc %}
// error handler
app.use(function(err, req, res, next) {
    console.error(err.stack);
    res.status(500).send('Something broke!');
});
app.use(function(req, res, next) {
    res.status(404).send('Sorry cant find that!');
});

{% endcodeblock %}

app.js 에서 routes/index 의 참조 경로를 바꿔준다.
``` bash
var indexRouter = require('./src/routes/index');
```

src/routes/index.js 에서 router.get 부분에 res.render를 send로 바꾼다.
템플릿 엔진이 없으므로 render 는 사용되지 않는다.
{% codeblock src/routes/index.js lang:objc %}
res.send({ title: 'Express' });
{% endcodeblock %}
 
### 로컬 실행 및 확인

여기까지 왔다면, 이제 아래 커맨드 입력 시, 서버가 정상 실행된다.

``` bash
$ npm start

> test-api-server@0.0.0 start D:\test-api-server
> node src/bin/www
```

[http://localhost:3000/](http://localhost:3000/) 로 브라우저에서 접속해보면,
{"title":"Express"} 문자열이 찍히는 걸 볼 수 있다.

### nodemon 설치
개발을 하다보면 수정 및 변경 후 확인을 위해 매번 npm start 커맨드를 입력하기가 번거로우므로
[nodemon](https://akanamed.github.io/2019/12/27/VSCode%EC%97%90%EC%84%9C-Node-js-%EB%94%94%EB%B2%84%EA%B9%85%ED%95%98%EA%B8%B0/#nodemon-%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%94%94%EB%B2%84%EA%B7%B8-%EC%84%A4%EC%A0%95) 을 설치하자.
package.json 의 npm script 를 아래와 같이 바꿔준다.
``` bash
"scripts": {
    "start": "nodemon src/bin/www"
  },
```

서버를 실행해보자.
``` bash
$ npm start
D:\test-api-server>npm start

> test-api-server@0.0.0 start D:\test-api-server
> nodemon src/bin/www

[nodemon] 2.0.2
[nodemon] to restart at any time, enter 'rs'
[nodemon] watching dir(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting 'node src/bin/www'
```

http://localhost:3000/1 을 입력하게 되면 app.js에 추가했었던 에러관련 코드도 볼 수 있다.

여기까지 기본적인 Server-side 프레임워크를 만들어보았다.
[해당 링크](https://www.freecodecamp.org/news/how-to-enable-es6-and-beyond-syntax-with-node-and-express-68d3e11fe1ab/) 에서는 babel관련 패키지, npm-run-all, rimraf 패키지, 빌드환경 설정등이 나와있지만, 기본 설정은 여기까지만으로도 충분하다고 생각된다.

번외)
해당 링크대로 심플하게 Server-side 용으로 프로젝트를 구성하였지만,
express (프로젝트명) --view=pug 로 생성 후 customizing 해도 될 것 같다.
위 생성과 차이점이라면, http-errors 패키지 포함 여부밖에 없다.

이제 기본 설정된 프로젝트를 개선하고 추가할 일만 남았다.

Done.
