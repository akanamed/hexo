---
title: Create Game api server with node.js express-6
date: 2020-01-15 19:07:03
updated:
categories:
   - nodejs
   - express
tags:
   - express
   - apiserver
---

passport 인증 미들웨어를 추가하고, User Schema에서 예외처리 및
중복관련 처리를 보완한다.
<!-- more -->
<!-- toc -->

## mongoose schema validate
이전까지 만든 UserSchema 구조는 userid에 빈 문자열이 들어와도 DB에 저장이 된다.
[Mongoose Custom Validators](https://mongoosejs.com/docs/validation.html#custom-validators) 문서를 참고하여
아래와 같이 조건검사를 추가한다.
userid는 regexp를 이용하여 영문자,숫자 조합만 허용하며, 4~12 길이제한을 둔다.
password는 4글자 이상만 허용한다.

{% gist f928883c14de488d5b003f1de89618c7 %}

postman으로 post 요청(http://127.0.0.1:3900/auth/create)을 아래와 같이 보내보면,
``` bash
request:
{
    "userid": "cc!!@#",
	"password": "1235"
}

response:
{
    "message": "User validation failed: userid: user id is not allow text, password: password is short, at least 4characters"
}

result:
ValidationError: User validation failed: userid: user id is not allow text, password: password is short, at least 4characters
    at new ValidationError (D:\test-api-server\node_modules\mongoose\lib\error\validation.js:31:11)
    at model.Document.invalidate (D:\test-api-server\node_modules\mongoose\lib\document.js:2461:32)
    at p.doValidate.skipSchemaValidators (D:\test-api-server\node_modules\mongoose\lib\document.js:2310:17)
    at D:\test-api-server\node_modules\mongoose\lib\schematype.js:1064:9
    at processTicksAndRejections (internal/process/task_queues.js:75:11)
POST /auth/create 422 40.593 ms - 122
```
ValidationError 를 볼 수 있다.

### /auth/create 중복 처리
이제 validation 까지 처리는 되었지만, 중복가입에 대한 처리는 되어 있지 않다.
this.model.findOne 으로 검색 후, db에 값이 없다면 신규 생성이므로 create를 이어서 요청하고,
db에 값이 있다면, /auth/login/fail 로 redirect 해준다.

{% codeblock auth.controller.js lang:objc %}
exports.createUser = (req, res, next) => {
    const user = req.body;

    UserRepository.findOne(user.userid)
        .then((searchUser) => {
            if (searchUser === null) {
                console.log('search user is null');
                return UserRepository.create(user);
            }
            return res.redirect('/auth/login/fail');
        })
        .then((createUser) => {
            res.json(createUser);
        })
        .catch((error) => {
            res.status(422).json({
                error
            });
        });
};
{% endcodeblock %}

### 번외 : async / await 를 이용한 방법
위와 동일한 코드지만, Node.js v7 부터 지원되는 async/await 로 아래와 같이 변경도 가능하다.
{% codeblock auth.controller.js lang:objc %}
exports.createUser = async (req, res, next) => {
    try {
        const user = req.body;
        const searchUser = await UserRepository.findOne(user.userid);
        if (searchUser) {
            res.redirect('/auth/login/fail');
        } else {
            const createUser = await UserRepository.create(user);
            res.json(createUser);
        }
    } catch (error) {
        console.log('error');
        error.status = 422;
        next(error);
    }
};
{% endcodeblock %}

주의해야 할 점은 express 라우터에서 async 함수 내에서 에러 발생 시, 에러 처리 미들웨어가
오류를 인식하지 못한다는 점이다. 그래서 try / catch 로 감싸줘야 한다.
[라우팅핸들러 async/await](https://programmingsummaries.tistory.com/399) 의 링크에 잘 나와있다.

## passport 미들웨어 적용
구글 검색하면 passport 관련 설명은 워낙 잘되어 있으므로 생략하고,
[passport 공식 문서](http://www.passportjs.org/docs/)를 참고하여,
passport 관련 미들웨어들을 설치하고 localstrategy 를 이용해 인증처리를 한다.
``` bash
npm i passport passport-local --save
```

아래와 같이 passport 모듈을 만들어 준다.
{% codeblock config/passport.js lang:objc %}
import passportLocal from 'passport-local';
const LocalStrategy = passportLocal.Strategy;
import UserRepository from '../models/repository/userRepository';

module.exports = (passport) => {
    passport.serializeUser((user, done) => {
        console.log('serializeUser:%O', user);
        done(null, user);
    });
    passport.deserializeUser((id, done) => {
        console.log('deserializeUser: %O', id);
        done(null, id);
    });

    passport.use(new LocalStrategy({
        usernameField: 'userid',
        passwordField: 'password'
    }, (userid, password, done) => UserRepository.findOne(userid)
        .then((resultUser) => {
            if (resultUser === null) {
                console.log('resultUser');
                return done(null, false);
            }
            console.log('call localstrategy');
            return done(null, { userid: resultUser.userid });
        })
        .catch((err) => done(err))));
};
{% endcodeblock %}

app.js 에 passport 관련 미들웨어를 등록하는데, 세션 저장소를 사용하고 있다면,
반드시 아래와 같이 session 등록 이후에 passport 미들웨어를 등록해야 한다.
{% codeblock app.js lang:objc %}
app.use(session( ... ))
app.use(passport.initialize());
app.use(passport.session());
passportConfig(passport);
{% endcodeblock %}

마지막으로, /auth/login 시, passport.authenticate('local') 방식으로 인증처리 되도록
수정한다.

{% codeblock auth.controller.js lang:objc %}
exports.login = (req, res, next) => {
    const user = req.body;
    console.log('%s, %s, %O, %O', user.userid, user.password, req.user, req.session.passport);
    passport.authenticate('local', { session: false }, (err, result, info) => {
        if (err) {
            return next(err);
        }
        if (result === null || result === false) {
            return res.redirect('/auth/login/fail');
        }
        return req.login(result, (error) => {
            console.log('call req.login, %O, %O', req.user, req.session.passport);
            if (error) {
                next(error);
            }
            res.redirect('/auth/login/success');
        });
    })(req, res, next);
}
{% endcodeblock %}

### passport 인증 동작 방식 확인
수정한 코드에 유저 로그인 시 어떠한 방식으로 동작되는지 console.log를 추가하였는데
서버를 실행 후 postman 으로 요청해보면 아래와 같은 로그가 출력된다.

``` bash
bbbbb, 12345, undefined, undefined
call localstrategy
serializeUser:{ userid: 'bbbbb' }
call req.login, { userid: 'bbbbb' }, { user: 'bbbbb' }
GET /auth/login 302 25.412 ms - 41
deserializeUser: 'bbbbb'
GET /auth/login/success 200 0.971 ms - 27
```
출력된 로그를 분석해보면
1. { userid: bbbbb, password: 12345 } 의 body 정보를 담아 /auth/login 으로 request 보냄
    ( 아직 req.user , req.session.passport는 undefined )
2. passport.authenticate 함수 호출해서 존재하는 유저라면, done 콜백함수에 userid만 저장 후 리턴.
3. serializeUser 함수 호출 : done 콜백함수에서 user 정보를 이용해 세션에 저장.
    ( 이때 req.user 및 req.session.passport.user에 정보가 저장된다. )
4. req.login 함수 호출 : req.user 와 req.session.passport 정보가 출력된다.
5. redirect( /auth/login/success ) 요청에 의해 deserializeUser가 호출되어 세션에 저장된 정보가 출력.

즉, passport 동작 방식은
login request -> passport.authenticate -> serializeUser -> req.login -> response  이다.
인증이 통과한 유저는 웹페이지 요청 시 deserializeUser 가 호출됨을 알 수 있다.

### req.isAuthenticated
서버로 보내는 모든 웹 요청에 유저가 인증이 되었는지 확인 하는 함수가 req.isAuthenticated 이다.
아래 코드처럼 /auth/logininfo 요청이 들어오면 서버에서 ensuerAuthenticated 함수로 
인증여부를 확인 한 후 세션 정보가 없거나 만료되었다면 로그인이 필요하다는 응답을 보내주고, 
세션 정보가 있는 유저라면 loginInfo 컨트롤러 함수를 실행해서 응답해준다.

{% codeblock auth.route.js lang:objc %}
function ensureAuthenticated(req, res, next) {
    if (!req.isAuthenticated()) {
        res.status(403).json({ message: 'need login session' });
    } else {
        next();
    }
}
authRouter.route('/logininfo')
    .get(ensureAuthenticated, authController.loginInfo);

{% endcodeblock %}

{% codeblock auth.controller.js lang:objc %}
exports.loginInfo = (req, res, next) => {
    console.log('call loginInfo, %O, %O', req.user, req.session.passport);
    const userInfo = req.user;
    return res.json({
        user: userInfo
    });
};
{% endcodeblock %}

세션정보가 있는 유저가 /auth/logininfo 요청을 해보면
``` bash
deserializeUser: 'bbbbb'
call loginInfo, 'bbbbb', { user: 'bbbbb' }
GET /auth/logininfo 200 1.032 ms - 16
```
passport 인증단계는 이미 거쳤으므로 deserializeUser 만 호출되어 응답한다.

Done.
