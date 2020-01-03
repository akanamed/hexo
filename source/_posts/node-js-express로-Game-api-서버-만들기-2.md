---
title: node.js+express로 Game api 서버 만들기(2)
date: 2020-01-03 12:00:33
updated:
categories:
   - nodejs
   - express
tags:
   - express
   - apiserver
---

[1편](https://akanamed.github.io/2019/12/30/node-js-express%EB%A1%9C-Game-api-%EC%84%9C%EB%B2%84-%EB%A7%8C%EB%93%A4%EA%B8%B0(1)/)에서 
만들었던 기본 구조에 babel 을 설정해보자.
ES5 로만 코드를 작성한다면, babel은 필요없을 지 모르나, 
ES6+ 의 새로운 기능을 사용하기 위해 설치하자.
<!-- more -->
<!-- toc -->

## Babel Configuration
[바벨 가이드](https://babeljs.io/docs/en/usage) 대로 설정하자.

### install babel package
``` bash
$ npm install -D @babel/core @babel/cli @babel/preset-env @babel/node
```
node 환경이므로 @babel/node도 추가해준다.
package.json 에 아래와 같이 추가되었다.
``` bash
"devDependencies": {
    "@babel/cli": "^7.7.7",
    "@babel/core": "^7.7.7",
    "@babel/node": "^7.7.7",
    "@babel/preset-env": "^7.7.7",
```

### preset 설정
.babelrc 파일을 만들고 아래와 같이 정의해준다.
( 프로젝트의 root 에 파일을 만든다. )

``` bash
{
    "presets": [
        [
            "@babel/preset-env", {
                "targets": {
                    "node": "current"
                }
            }
        ]
    ]
}
```
Node.js 에서 babel 을 실행하기 위해 package.json 을 
아래와 같이 수정하고 서버를 실행해보자.
``` bash
"scripts": {
    "start": "babel-node src/bin/www"
  },
```

``` bash
$ npm start
> test-api-server@0.0.0 start D:\test-api-server
> babel-node src/bin/www

Server Listening on 3000
```

### nodemon 을 이용한 babel-node

개발환경에서는 코드 수정이 빈번하므로 nodemon을 이용한 babel-node 실행을 추가한다.
package.json
``` bash
"scripts": {
    "start": "nodemon src/bin/www --exec babel-node"
  },
```

## 그러면 express 에서 babel은?

사실 이부분이 제일 중요할 것 같은데, 
express-generator 로 생성한 프로젝트는 기본 실행이 bin/www 에서 이루어진다.
아래 app.js에서 29라인을 export default app; 으로 수정 후
bin/www 에서 import App from "../app"; 으로 선언하게 되면,
SyntaxError: Cannot use import statement outside a module 에러가 뜬다.

{% gist b7a59fa048b6a9f9fa8db9050b789ac2 %}

즉, generator로 생성한 bin/www 는 단순히 서버 구동을 위한 코드이므로,
app.js 파일을 가져와 http 객체와 연결시켜주는 동작밖에 없다.
app.js에서의 export는 예외적으로 commonJS 문법으로 bin/www에 넘겨주어야 한다.

Done.