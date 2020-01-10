---
title: Node.js 에서 exports vs Module.exports
date: 2020-01-10 14:53:43
updated:
categories:
   - nodejs
tags:
   - nodejs
   - module
   - export
---

Node.js에서 exports 와 Module.exports의 차이에 대해 알아보자.

<!-- more -->
<!-- toc -->

## module
test.js 파일을 만들고 node 커맨드로 콘솔 로그로 출력한다.
{% codeblock test.js lang:objc %}
console.log(module);
{% endcodeblock %}

``` bash
D:\test-api-server>node src/test.js
Module {
  id: '.',
  path: 'D:\\test-api-server\\src',
  exports: {},
  parent: null,
  filename: 'D:\\test-api-server\\src\\test.js',
  loaded: false,
  children: [],
  paths: [
    'D:\\test-api-server\\src\\node_modules',
    'D:\\test-api-server\\node_modules',
    'D:\\node_modules'
  ]
}
```
module 객체에 대한 정보를 볼 수 있다.
컨텍스트 정보 중 exports 객체는 모듈로 가져올 프로퍼티, 메소드들이 저장되는 곳인데
아직 아무런 할당을 하지 않았으므로 빈 오브젝트로 초기화 되어있다.

### exports
test.js 를 수정해서 exports 객체로 프로퍼티를 할당해보자.
{% codeblock test.js lang:objc %}
exports.a = 'A';
exports.b = 'B';
console.log(module);
{% endcodeblock %}

``` bash
Module {
  id: '.',
  exports: { a: 'A', b: 'B' },
  ...
```
exports 객체로 프로퍼티들을 할당하면 modules.exports에도 추가되어 있다.
Node.js의 모듈 시스템에서 실제로 익스포트 되는 객체는 module.exports 이고,
exports 객체는 modules.exports를 참조하고 있는 변수에 불과하다.

### module.exports
그럼 이번에는 module.exports에 프로퍼티를 추가해보자.

{% codeblock test.js lang:objc %}
module.exports.a = 'A';
module.exports.b = 'B';
console.log(module);
{% endcodeblock %}

``` bash
Module {
  id: '.',
  exports: { a: 'A', b: 'B' },
  ...
```
결과는 동일하다.


## exports 와 module.exports
그럼 exports 객체는 항상 module.exports를 참조하고 있을까?

1.
{% codeblock test.js lang:objc %}
module.exports = { a: 'A', b: 'B' };
console.log(exports === module.exports);
console.log(exports);
console.log(module);
{% endcodeblock %}

``` bash
false
Module {
  id: '.',
  exports: { a: 'A', b: 'B' },
  ...
}
{}
```

위 예제들과 같은 프로퍼티 값들이 할당되어 있다.
하지만 module.exports에 프로퍼티가 아닌 객체를 할당하면 참조하고 있는 exports 객체는 
관련없는 객체가 되어버린다.
따라서 exports 객체가 module.exports 와 같지 않고, 비어있다.

2.
{% codeblock test.js lang:objc %}
module.exports = {
    a: 'A'
};
exports = {
    b: 'B'
}

console.log(exports === module.exports);
console.log(module);
console.log(exports);
{% endcodeblock %}

``` bash
false
Module {
  id: '.',
  exports: { a: 'A' },
  ...
}
{ b: 'B' }
```

exports 객체는 별도로 저장되고, 외부로 전달되지 않는다.
test.js에서는 무시되며 module.exports와 전혀 관련없는 객체임을 알 수 있다.

## 정리
exports 객체는 module.exports 를 참조하고 있으며 최종적으로 리턴되는 값은
module.exports 이다.


참고링크
[Node.js: exports vs module.exports](https://www.hacksparrow.com/nodejs/exports-vs-module-exports.html)
[Understanding module.exports and exports in Node.js](https://www.sitepoint.com/understanding-module-exports-exports-node-js/)
[node.js의 module.exports와 exports](https://edykim.com/ko/post/module.exports-and-exports-in-node.js/)