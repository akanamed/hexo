---
title: Debugging Nodejs in VSCode
date: 2019-12-27 16:10:30
updated:
categories:
   - nodejs
tags:
   - vscode
   - debugging
   - nodemon
---

visual studio code 에서 node.js 디버깅은 정말 간단하다.
그 외 nodemon 을 이용한 디버그 방법도 있다.
여기서는 기본 디버그 설정과 nodemon 디버그 설정 2가지만 다룬다.

<!-- more -->
<!-- toc -->
## 기본 launch 를 이용한 디버그 설정

ctrl + shift + D 키를 눌러서 상단 톱니바퀴를 누르거나,
메뉴 - 디버그 - 구성열기를 선택하면
해당 프로젝트의 .vscode 폴더 아래에 launch.json 파일이 생성된다.

### launch.json 파일

생성된 launch.json 파일은 아래와 같다.

{% codeblock launch.json lang:objc %}
{
    // IntelliSense를 사용하여 가능한 특성에 대해 알아보세요.
    // 기존 특성에 대한 설명을 보려면 가리킵니다.
    // 자세한 내용을 보려면 https://go.microsoft.com/fwlink/?linkid=830387을(를) 방문하세요.
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "프로그램 시작",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "program": "${workspaceFolder}\\bin\\www"
        }
    ]
}
{% endcodeblock %}

F5 (디버그 시작) 을 누르면, 아래 디버그 콘솔에 Debugger attached.가 출력되며, 
디버깅을 할 수 있는 상태가 된다.

``` bash
C:\Program Files\nodejs\node.exe --inspect-brk=6936 bin\www 
Debugger listening on ws://127.0.0.1:6936/69776105-213c-4108-95d8-05da7bf9fdab
For help, see: https://nodejs.org/en/docs/inspector
Debugger attached.
```

## nodemon 을 이용한 디버그 설정

nodemon 이란, 소스의 변경사항을 모니터링하고 서버를 자동으로 다시 시작하는 유틸리티입니다.
라고 [공식홈페이지](https://nodemon.io/) 에 나와있다.
글로벌 설치로 가이드 되어 있지만, 로컬설치를 하게되면 시스템 경로에서 nodemon을 사용할 수는
없고 아래 설명하겠지만, package.json 에 npm scripts로 추가하여 호출할 수 있다.

``` bash
$ npm install --save-dev nodemon 
```

package.json에 npm 스크립트를 추가해야하는데, start 와 debug 를 아래와 같이 추가한다.
{% codeblock package.json lang:objc %}
{
    "scripts": {
        "start": "nodemon ./bin/www",
        "debug": "nodemon --inspect ./bin/www"
    }
}
{% endcodeblock %}

npm script 를 이용한 nodemon으로 서버 실행하는 방법
``` bash
$ npm start

[nodemon] 1.19.3
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `./bin/www`
2019-12-27 16:45:20.157 +0900 [test] info : API Server Listening on 127.0.0.1:3000
```

### launch.json에 nodemon 디버그 설정

생성된 launch.json에 nodemon 설정을 아래와 같이 추가해준다.

{% codeblock launch.json lang:objc %}
{
    "version": "0.2.0",
    "configurations": [
        {
            ...
        },
        {
            "type": "node",
            "request": "attach",
            "name": "Node: Nodemon",
            "processId":"${command:PickProcess}",
            "restart": true,
            "protocol": "inspector",
            "skipFiles": [
                "<node_internals>/**"
            ]
        }
    ]
}
{% endcodeblock %}

기본 디버깅과 차이점은 "request": "attach" 항목이다.
서버를 실행한 후 디버그에서 프로세스 연결을 해줘야 한다.

### nodemon 으로 디버깅하기

아래와 같이 command를 입력. 디버그 프로세스를 실행한다.

``` bash
$ npm run debug

[nodemon] 1.19.3
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `--inspect ./bin/www`
Debugger listening on ws://127.0.0.1:9229/d62df68b-7c8e-416a-9a78-8351eabc2871
For help, see: https://nodejs.org/en/docs/inspector
2019-12-27 16:55:42.882 +0900 [test] info : API Server Listening on 127.0.0.1:3000
```

그럼 이제 마지막으로 프로세스 연결을 위해서 디버그로 이동한다음 아래 이미지처럼

{% asset_img nodemon.PNG This is an example image %}
{% asset_img "spaced nodemon.PNG" "spaced title" %}

상단 Node:Nodemon을 선택하고 F5를 실행하거나 초록색 플레이 버튼을 누르게 되면, 실행중인 
node processes 리스트들이 나오는데 --inspector로 실행된 프로세스를 선택하면
Debugger attached. 가 터미널에 출력되며, 디버깅 가능한 상태가 된다.


Done.

[ 참고 자료 ]
[Node.js debugging in VS Code with Nodemon](https://github.com/Microsoft/vscode-recipes/tree/master/nodemon)
[Node.js debugging in VS Code](https://code.visualstudio.com/docs/nodejs/nodejs-debugging)