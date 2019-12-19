---
title: express server
date: 2019-12-17 10:33:11
categories:
   - nodejs
tags:
   - express
---
## express 서버 만드는 두가지 방법

node js가 설치가 되어 있다는 가정하에
초 간단 express 웹 서버를 만드는 방법의 포스팅이다.


<!-- more -->
<!-- toc -->

### Command로 현재 버전확인
``` bash
$ node --version  
v10.16.3

$ npm --version
6.13.4
```

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

npm init 커맨드는 package.json을 생성하는데, -y 옵션을 주게 되면 알아서 내용을 채워준다.

#### npm init
``` bash
$ npm init
$
$ This utility will walk you through creating a package.json file.
$ It only covers the most common items, and tries to guess sensible defaults.

$ See `npm help json` for definitive documentation on these fields
$ and exactly what they do.

$ Use `npm install <pkg>` afterwards to install a package and
$ save it as a dependency in the package.json file.

$ Press ^C at any time to quit.
$ package name: (test1)
$ version: (1.0.0)
$ description: test project
$ entry point: (index.js)
$ test command:
$ git repository:
$ keywords:
$ author:
$ license: (ISC)
$ About to write to package.json:

$ {
$   "name": "test1",
$   "version": "1.0.0",
$   "description": "test project",
$   "main": "index.js",
$   "scripts": {
$     "test": "echo \"Error: no test specified\" && exit 1"
$   },
$   "author": "",
$   "license": "ISC"
$ }


$ Is this OK? (yes)
```

#### npm init -y
``` bash
$ npm init -y

$ Wrote to package.json:

$ {
$   "name": "test1",
$   "version": "1.0.0",
$   "description": "",
$   "main": "index.js",
$   "scripts": {
$     "test": "echo \"Error: no test specified\" && exit 1"
$   },
$   "keywords": [],
$   "author": "",
$   "license": "ISC"
$ }
```
#### express install
``` bash
$ npm install express --save

$ npm notice created a lockfile as package-lock.json. You should commit this file.
$ + express@4.17.1
$ added 50 packages from 37 contributors and audited 126 packages in 2.785s
$ found 0 vulnerabilities
```
package.json -> dependencies 에 express 가 추가 되었으며, package-lock.json 도 생성이 되었다.
해당 관련 설명은 아래 링크 참조.
[package.json 설명 링크](https://programmingsummaries.tistory.com/385)


해당 작업 디렉토리에 app.js 파일을 하나 생성한 후 아래와 같은 코드를 작성한다.

{% codeblock app.js lang:objc %}
var express = require('express');
var app = express();

app.get('/', function (req, res) {
    res.send('Hello World!');
});

app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
});
{% endcodeblock %}


### 서버 실행

기본적인 실행방법은 아래 커맨드.
``` bash
$ node app.js
```

package.json 에 스크립트를 추가하여 서버 실행하는 방법.
{% codeblock package.json lang:objc %}
...
"scripts": {
    "start": "node app.js"
},
...
{% endcodeblock %}
``` bash
$ npm start
```

Done.