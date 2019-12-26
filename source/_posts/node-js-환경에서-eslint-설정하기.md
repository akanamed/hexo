---
title: node.js 환경에서 eslint 설정하기
date: 2019-12-26 16:08:08
updated:
categories:
   - nodejs
   - eslint
tags:
   - eslint
---

eslint : Javascript 코드에서 발견된 문제 패턴을 식별하기 위한 정적 코드 분석 도구.
문법 에러 뿐만 아니라 코딩 스타일도 정할 수 있어 유용하다.

<!-- more -->
<!-- toc -->

### eslint 설치

npm install 시, -D 옵션을 주게 되면 현재 프로젝트 기준 로컬에 설치되며,
package.json > "devdependencies" 항목에 추가되어
production 으로 배포 시, 해당 모듈은 포함하지 않는다.

{% blockquote %}
-D 는 --save-dev 와 같다.
{% endblockquote %}


``` bash
$ npm install -D eslint
$ 
$ 실행결과
$ + eslint@6.8.0
$ added 120 packages from 81 contributors and audited 305 packages in 11.633s
```

#### node js 환경을 위한 eslint 설정파일 만들기
``` bash
$ .\node_modules\.bin\eslint --init
$ 
// 구문을 확인하고 문제를 찾고 코드 스타일 적용
? How would you like to use ESLint? To check syntax, find problems, and enforce code style
// 프로젝트에 Babel 이 설치되어 있으면(React, Vue 등) Javascript 옵션을 선택하고, 
// babel과 관련없는 node js 프로젝트 및 기타 자바 스크립트 프로젝트이면 CommonJS 선택
? What type of modules does your project use? CommonJS (require/exports)
// node js 를 사용하므로 None of these 사용
? Which framework does your project use? None of these
// TypeScript 사용안함
? Does your project use TypeScript? No
// Node 환경에서 코드 실행
? Where does your code run? Node
// 인기있는 스타일 가이드 ( air-bnb 스타일 가이드 사용을 위해.)
? How would you like to define a style for your project? Use a popular style guide
? Which style guide do you want to follow? Airbnb: 
// .eslintrc 파일의 구성형식 ( js, yml, json 중 선택)
? What format do you want your config file to be in? JSON
Checking peerDependencies of eslint-config-airbnb-base@latest
The config that you have selected requires the following dependencies:
eslint-config-airbnb-base@latest eslint@^5.16.0 || ^6.1.0 eslint-plugin-import@^2.18.2
// 지금 npm 이용해서 위 모듈들 설치
? Would you like to install them now with npm? Yes
Installing eslint-config-airbnb-base@latest, eslint@^5.16.0 || ^6.1.0, eslint-plugin-import@^2.18.2
npm WARN test1@1.0.0 No description
npm WARN test1@1.0.0 No repository field.

+ eslint-config-airbnb-base@14.0.0
+ eslint-plugin-import@2.19.1
+ eslint@6.8.0
added 55 packages from 34 contributors, updated 1 package and audited 513 packages in 5.957s
 
16 packages are looking for funding
  run npm fund for details
 
found 0 vulnerabilities
Successfully created .eslintrc.json file in D:\test1
```

위와 같이 선택했다면, .eslintrc.json 파일이 추가되어 있음을 볼 수 있다.


#### vscode 에 eslint 설치하기

ctrl+shift+x 키를 누르면, extensions 마켓 플레이스 창이 뜨는데, eslint 를 검색 후 install 한다.
install 후 vs code 재시작 및 적용 확인.

### eslint rule 세팅

아래와 같이 간단히 코드를 작성한다.
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

아래 이미지 처럼 주황색 언더라인을 볼 수 있는데, 
air-bnb 코딩스타일 혹은 문법에 맞지 않는다는 표시가 나타난다.
{% asset_img app.PNG This is an example image %}
{% asset_img "spaced app.PNG" "spaced title" %}

#### 스타일 및 문법에 맞게 수정 

아래 이미지 처럼 오류가 생긴 줄 위에 커서를 위치하면, {% hl_text danger %}빠른수정...{% endhl_text %}툴팁이 나오는데,
{% asset_img app1.PNG This is an example image %}
{% asset_img "spaced app1.PNG" "spaced title" %}


툴팁을 클릭 -> Fix this {rules} problem 으로 수정을 선택하면 자동으로 수정을 해준다.

{% blockquote %}
Fix all {rules} problems 는 소스파일에서 발생한 해당 rules 관련 문제를 일괄 수정해준다.
{% endblockquote %}

eslint 공식문서: [Rules Documents](https://eslint.org/docs/rules/)
위 공식문서를 참고해서, Search the docs 에 해당 rule을 검색하면,
오류 원인, Rule Details, Examples, When Not To Use It 등 해당 rule의 가이드를 볼 수 있다.
가이드 문서에는 --fix 옵션으로 command를 입력하게 되면 사소한 에러들은 자동으로 고쳐주므로, 적용해보면
``` bash
$ .\node_modules\.bin\eslint app.js --fix
```
{% codeblock app.js lang:objc %}
const express = require('express');
const app = express();
app.get('/', (req, res) => {
  res.send('Hello World!');
});
app.listen(3000, () => {
  console.log('Example app listening on port 3000!');
});
{% endcodeblock %}

위 와 같이 변경이 된다.

그 외, .eslintrc.json 에서 "rules" 에 검사를 무시할 rule 추가라던가, 세미콜론, 탭 크기 등
여러가지 설정에 관한 Configuration 은 추 후 따로 포스팅으로 정리를 해야겠다.