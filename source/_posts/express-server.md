---
title: express server
date: 2019-12-17 10:33:11
categories:
   - nodejs
tags:
   - express
---
## express 서버 만드는 두가지 방법

### Express Genarator 이용

``` bash
$ // npm 5.2.0 버전 이상
$ npx express-generator
$
$ // 이전버전
$ npm install -g express-generator
```

참고 : [Express 공식사이트](https://expressjs.com/en/starter/generator.html)

### npm init 이용

``` bash
$ // package.json - custom 생성 여부. -y 옵션 시, 자동생성
$ npm init -y
$ npm install express --save
```
위 방법 이용 시, 해당 작업 디렉토리에 app.js 파일을 하나 생성한 후 아래와 같은 코드를 작성

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