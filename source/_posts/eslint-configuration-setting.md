---
title: eslint configuration setting
date: 2019-12-31 10:36:22
updated:
categories:
   - nodejs
   - eslint
tags:
   - eslint
---

.eslintrc.json 파일에서 rules 관련 옵션들을 추가한다.

<!-- more -->
<!-- toc -->

이전글 : [node.js 환경에서 eslint 설정하기](https://akanamed.github.io/2019/12/26/node-js-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-eslint-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/)
eslint 패키지를 설치하고 init 을 하면 아래와 같은 json 파일이 생성된다.

``` bash
{
    "env": {
        "commonjs": true,
        "es6": true,
        "node": true
    },
    "extends": [
        "airbnb-base"
    ],
    "globals": {
        "Atomics": "readonly",
        "SharedArrayBuffer": "readonly"
    },
    "parserOptions": {
        "ecmaVersion": 2018
    },
    "rules": {
    }
}
```
eslint.json 의 rules 항목에는,
추가한 rule을 error 로 설정해서 따르지 않으면 수정하게 에러를 뱉어내거나
rule을 turn off 으로 설정해서 검사를 안하게 하는 등의 옵션들을 추가할 수 있다.

### rule 관련 수정
[Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) 깃헙 문서에 잘 나와있다.

airbnb 스타일은 들여쓰기에 대해 강제하지는 않는 것 같다.
보기좋게 4칸으로 고정하는 옵션을 추가한다.
``` bash
"rules": {
    "indent": [
        "error",
        4
    ],
```

아래 샘플로 작성한 파일이다. airbnb 스타일의 eslint 에서 에러로 표시되는 항목은 다음과 같다.
{% codeblock test.js lang:objc %}
var express = require('express');
var tempFunc = function(err) {
  return 1;
};

var testFunc = (err, function(next) {
  console.log('err');
});
{% endcodeblock %}

1. no-var : var 대신에 let, const 사용
```bash
const express = require('express');
```
2. import/newline-after-import : import 및 require 선언 후 한줄 공백이 필요
```bash
const express = require('express');

const tempFunc = function(err) {
```
3. space-before-function-paren : 함수 괄호 앞에 공백이 필요
```bash
const tempFunc = function (err) {
```
4. prefer-arrow-callback: 콜백함수는 화살표 함수로 사용
```bash
const testFunc = (err, (next) => {
```
5. no-undef: 정의되지 않은 변수 사용, 여기서는 err 변수
```bash
const err = undefined;
const testFunc = (err, (next) => {
```
6. func-names: 함수 표현식에 이름이 있어야함
``` bash
const tempFunc = (err) => 1;
or
const tempFunc = function test(err) {
```

no-console 은 콘솔 사용금지인데, node.js 환경에서 디버그 용도로 확인해야하므로
turn off 설정해주자.
no-unused-vars: 함수안의 파라미터가 선언은 되었지만 사용은 하지 않음인데, 
마찬가지로 rules 에 추가하여 turn off 로 제외시키자.

"off" or 0 - 규칙 해제
"warn" or 1 - 규칙을 경고로 설정
"error" or 2 - 규칙을 오류로 설정

``` bash
"rules": {
    "no-unused-vars": 0,
    "no-console": 0
```

최종 수정된 코드
{% codeblock test.js lang:objc %}
const express = require('express');

const tempFunc = (err) => 1;
const err = undefined;
const testFunc = (err, (next) => {
    console.log('err');
});

{% endcodeblock %}

### --fix 옵션으로 수정

--fix 옵션으로 자동 수정을 해보자.
``` bash
$ .\node_modules\.bin\eslint test.js --fix
// 결과
D:\test\test.js
  3:18  warning  Unexpected unnamed function  func-names
  7:19  error    'err' is not defined         no-undef
  7:24  warning  Unexpected unnamed function  func-names
```

거의 대부분은 다 자동으로 잡아주지만,
func-names 는 warning 으로, no-undef 는 error 로 출력이 되었다.

공식 도큐먼트에는 func-names 는 디버깅에 도움이 되도록 함수 표현식 이름을
지정하라고 나와있다.
함수 이름을 생략하면 함수에서 예외 발생 시, 스택에서 (anonymous function) 으로 표시된다.
따라서 이름을 주던지, arrow function 으로 바꾸면 해결 할 수 있다.
no-undef 는 변수 선언 시, 항상 const 혹은 let을 사용하라고 나와있다.

최종 수정된 코드
{% codeblock test.js lang:objc %}
const express = require('express');

const tempFunc = (err) => 1;
const err = undefined;
const testFunc = (err, (next) => {
    console.log('err');
});

{% endcodeblock %}

### rules configuration

거의 대부분은 airbnb 스타일에 따라 수정해가면 되지만, 그 외 조금 불필요한 옵션들이나 
설정해야하는 옵션들은 turn off 혹은 error 처리로 정의했다.

``` bash
"rules": {
    "semi": [
            "error",
            "always"
    ],
    "indent": [
        "error",
        4
    ],
    "no-unused-vars": 0,
    "no-console": 0,
    "keyword-spacing": 0,               // if문 다음에 공백이 필요함.
    "comma-dangle": ["error", "never"], // 후행 쉼표 허용안함
}
```