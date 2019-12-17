---
title: express server
date: 2019-12-17 10:33:11
categories:
   - nodejs
tags:
   - express
---
## express 서버 만드는 두가지 방법

이 글을 포스팅 하는 시점의 node 버전은 10.16.3 이며, 설치가 되어 있다는 가정하에
초 간단 express 웹 서버를 만드는 두가지 방법은 아래와 같다.
<!-- excerpt -->
<!-- toc -->
### Express Genarator 이용

npm 버전이 5.2.0 이상인 경우 아래의 {% hl_text danger %}npx{% endhl_text %} 커맨드로 쉽게 생성할 수 있다.
npx 설명 링크 : [npx란 무엇인가?](https://ljh86029926.gitbook.io/coding-apple-react/undefined/npm-npx)

``` bash
$ npx express-generator
```

npm 버전이 이전 버전인 경우 글로벌 설치 후 express 서버를 만든다.

``` bash
$ npm install -g express-generator
$ express --view=pug [폴더명]
```

참고 : [Express 공식사이트](https://expressjs.com/en/starter/generator.html)

### npm init 이용

npm init 커맨드는 -y 옵션 여부에 따라 package.json 을 수동으로 생성할 지, 
자동으로 생성할 지 선택할 수 있다.
-y 옵션을 붙이면 package.json 파일만 자동으로 생성된다.

[package.json 설명 링크](https://programmingsummaries.tistory.com/385)

``` bash
$ npm init -y
$ npm install express --save
```

해당 작업 디렉토리에 app.js 파일을 하나 생성한 후 아래와 같은 코드를 작성한다.

{% codeblock lang:objc %}
var express = require('express');
var app = express();

app.get('/', function (req, res) {
    res.send('Hello World!');
});

app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
});
{% endcodeblock %}


서버 실행
``` bash
$ node app.js
```

Done.