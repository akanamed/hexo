---
title: mongoDB config 세팅
date: 2019-12-20 15:23:32
categories:
   - mongodb
tags:
   - mongodb
---

MongoDB 를 설치하게 되면 기본 구성파일은 Windows 기준으로 /install directory/bin/mongod.cfg 에 위치한다.
<!-- more -->
<!-- toc -->

mongod.cfg 는 YAML 형식을 사용한다.
( YAML 은 들여 쓰기, 탭 문자가 지원되지 않고, 공백을 사용해야 한다.)

MongoDB 3.6부터는 기본적으로 localhost에 바인딩 되는데, 
외부에서도 접속하게 허용할려면 cfg를 수정해야한다.

### command로 현재 버전 확인

``` bash
$ mongo --version

MongoDB shell version v4.2.1
git version: edf6d45851c0b9ee15548f0f847df141764a317e
allocator: tcmalloc
modules: none
build environment:
    distmod: 2012plus
    distarch: x86_64
    target_arch: x86_64

```

#### cfg 파일을 직접 변경

에디터로 파일을 열게 되면, 아래와 같은 내용을 볼 수 있다.
{% codeblock mongod.conf %}
 - for documentation of all options, see:
   http://docs.mongodb.org/manual/reference/configuration-options/

storage:
  dbPath: C:\Program Files\MongoDB\Server\4.2\data
  journal:
    enabled: true
  engine:
  mmapv1:
  wiredTiger:

- where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path:  C:\Program Files\MongoDB\Server\4.2\log\mongod.log

- network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1

processManagement:
security:
operationProfiling:
replication:
sharding:
-- Enterprise-Only Options:
auditLog:
snmp:
{% endcodeblock %}

net: bindIp를 0.0.0.0 으로 바꾸면 모든 ip주소에 바인딩이 되며, 외부에서도 접속할 수 있다.
보안상의 이유로 허용할 ip만 접속하게 한다면, 아래처럼 기입하면 된다.
{% codeblock mongod.conf %}
net:
  port: 27017
  bindIp: 127.0.0.1, 10.0.0.10, 10.0.0.25
{% endcodeblock %}

변경 후 mongo db server를 재시작 및 확인.


#### mongo shell 에서 변경

아래 처럼 --bind_ip 를 옵션으로 주게 되면, cfg 의 net: bind_ip 를 override 해서 mongodb 를 재시작한다.

``` bash
$ mongod --bind_ip 0.0.0.0
```

Done.