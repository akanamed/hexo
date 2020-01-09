---
title: Create Game api server with node.js express-5
date: 2020-01-09 16:50:31
updated:
categories:
   - nodejs
   - express
tags:
   - express
   - apiserver
---

지금까지 만든 프로젝트에 서버 및 DB 접속 정보나 포트가 하드코딩되어있다.
dotenv 패키지를 이용해 환경변수 파일을 만들고 관리한다.

<!-- more -->
<!-- toc -->

## Node.js process
Node.js의 process 객체는 global이다.
즉, require() 를 사용하지 않고, 접근할 수 있으며, 현재 프로세스에 대한 정보와 제어를
할 수 있다.

process 객체 중 이번 포스팅에서 다루는 건 env 환경 변수인데, dotenv 패키지를 이용해
프로젝트에서 사용할 환경 변수들의 정보를 .env 파일에 모아서 관리한다.

### dotenv 설치
dotenv 패키지를 설치한다.
``` bash
$ npm i dotenv --save
```
### create .env file
현재 프로젝트에서 관리할 정보들을 환경변수로 등록하기 위해 프로젝트 디렉토리에
.env 파일을 만들고 dotenv 패키지를 import 해준다.

.env 파일에 아래와 같이 'key=value'로 정의해주자.
``` bash 
HOST=127.0.0.1
PORT=3900
SECRET_KEY=ThIsIsMy$eCrEt
MONGODB_CON_HOST=localhost
MONGODB_SESSION_HOST=localhost
MONGODB_PORT=27017
MONGODB_NAME=testdb
```

### process.env 출력
console.log(process.env); 를 코드 내에 추가하고 출력결과를 확인해보면,
현재 컴퓨터에서 사용 중 인 환경변수뿐만 아니라 .env 에 정의한 key,value도 볼 수 있다.
``` bash
{
    ALLUSERSPROFILE: 'C:\\ProgramData',
    APPDATA: 'C:\\Users\\akanamed\\AppData\\Roaming',
    COLORTERM: 'truecolor',
    CommonProgramFiles: 'C:\\Program Files\\Common Files',
    'CommonProgramFiles(x86)': 'C:\\Program Files (x86)\\Common Files',
    CommonProgramW6432: 'C:\\Program Files\\Common Files',
    COMPUTERNAME: 'AKANAMED-PC',
    ComSpec: 'C:\\Windows\\system32\\cmd.exe',
    dp0: 'D:\\test-api-server\\node_modules\\.bin\\',
    DriverData: 'C:\\Windows\\System32\\Drivers\\DriverData',
    HOME: 'C:\\Users\\akanamed',
    HOMEDRIVE: 'C:',
    HOMEPATH: '\\Users\\akanamed',
    OS: 'Windows_NT',
    ProgramData: 'C:\\ProgramData',
    ProgramFiles: 'C:\\Program Files',
    'ProgramFiles(x86)': 'C:\\Program Files (x86)',
    ProgramW6432: 'C:\\Program Files',
    ...
    MONGODB_CON_HOST: 'localhost',
    MONGODB_NAME: 'testdb',
    MONGODB_PORT: '27017',
    MONGODB_SESSION_HOST: 'localhost',
    SECRET_KEY: 'ThIsIsMy$eCrEt',
}
```

## .env 환경 변수로 코드 수정
app.js에서 첫번째 줄에 아래와 같이 import 해준다.
{% codeblock app.js lang:objc %}
import 'dotenv/config';
{% endcodeblock %}

### import hoisting
ES6에서 import는 순서에 따라 모듈을 불러오므로, 만약 환경변수를 사용하는 모듈보다 
나중에 import 하게되면 아래와 같이 undefined 에러가 뜬다.
``` bash
Error: Error connecting to db: failed to connect to server [undefined:27017]
```
### code 수정
.env의 key를 사용하는 곳에 process.env.{key} 형태로 수정한다.
localhost:3000/testdb 같은 문자열에서 여러군데를 수정해야 한다면
backtick 문자를 사용하는 ES6의 템플릿 문자열로 아래와 같이 수정한다.

{% codeblock db/mongo.session.js lang:objc %}
const uriString = `mongodb://${process.env.MONGODB_SESSION_HOST}:${process.env.MONGODB_PORT}/${process.env.MONGODB_NAME}?connectTimeoutMS=5000`;
const mongoStore = new MongoDBStore(
    {
        uri: uriString,
        databaseName: process.env.MONGODB_NAME
        collection: process.env.MONGODB_COLLECTION
    },
{% endcodeblock %}

한가지 더 주의해야 할 점은 process.env.PORT 키의 값인 3900은 문자열이다.
그래서 express-generator로 생성한 프로젝트의 bin/www 내부에 normalizePort() 함수가 있는데,
내부에 parseInt() 로 문자열 매개변수를 숫자로 변환하는 로직을 볼 수 있다.

### .env 를 .gitignore에 추가
외부로 소스코드를 공개 시, 공유하면 안되는 혹은 중요한 정보보호를 위해 .env에 키를 추가하고
git 사용자라면 .gitignore 에 .env를 포함하면 된다.
프로덕션 레벨의 서버 빌드 및 배포로 webpack사용 시, dotenv-webpack 패키지를 사용하면 된다.

Done.