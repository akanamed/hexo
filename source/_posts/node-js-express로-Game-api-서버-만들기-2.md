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
<!-- more -->

[1편](https://akanamed.github.io/2019/12/30/node-js-express%EB%A1%9C-Game-api-%EC%84%9C%EB%B2%84-%EB%A7%8C%EB%93%A4%EA%B8%B0(1)/)에서 
만들었던 기본 구조에 babel 을 설정해보자.
ES5 로만 코드를 작성한다면, babel은 필요없을 지 모르나, 
ES6+ 의 새로운 기능을 사용하기 위해 설치하자.
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

Done.