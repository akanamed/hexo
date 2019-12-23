---
title: hexo tranquilpeak 테마 적용 삽질기
date: 2019-12-23 11:30:58
updated: 2019-12-23 11:30:58
categories:
   - hexo
tags:
   - hexo
   - tranquilpeak
---

## hexo 를 이용해 github 블로그 만들기

얼마전 부터 뭔가 정리를 하나씩 해야겠다는 생각에 github 블로그를 만들어보고자 했다.
결론부터 얘기하면 쉽지 않았다.
<!-- more -->
<!-- toc -->
github 블로그는 거의 대부분 jekyll 테마를 적용하는 듯 해서 시도를 해보았지만,
ruby 설치부터 적용까지가 쉽지 않았다. ( 결과적으로는 실패했다. )
그러던 중에 hexo 테마를 검색하게 되었고, nodejs 기반이라 익숙하기도 했으며,
조금 난관에 부딪히기도 했지만 성공적으로 만들게 되었다.

아래는 hexo tranquilpeak 테마 적용에 대해 도움을 많이 받은 사이트
[Github 블로그 만들기 with Hexo](https://mingpd.github.io/2019/04/14/github-blog-with-hexo-1/)

정말 위 사이트로 들어가서 목차대로만 하면 금방 만들어진다.

하지만 지금 이 블로그의 tranquilpeak 테마 버전의 최신 release는 3.1.1 이며, 
내가 수정한 것에 대한 삽질기를 기록으로 남긴다.

### title 클릭 시 깨짐 문제
``` bash
$ hexo s
$ http://localhost:4000/
```
로컬 실행모드로 블로그가 제대로 동작하는걸 확인했으나, 한가지 문제가 있었다.
포스트 상단의 header 를 클릭하면, home 이 동작해야 하는데 %20 관련 GET 에러가 뜬다.

위 에러 관련 수정은 아래와 같다.
{% codeblock themes/tranquilpeak/layout/header.ejs %}

    <div class="header-title">
        <a
            class="header-title-link"
            href="<%- url_for('') %>"
            aria-label="<% __('global.go_to_homepage') %>"
        >
            <%= config.title %>
        </a>
    </div>

{% endcodeblock %}

위 href 의 url_for('%20') 부분을 url_for('') 로 변경하면 된다.

### utterances 댓글 기능 추가
기본적으로 tranquilpeak 테마에는 utterances 기능이 없는 듯 하여, 추가하였다.

아래는 utterances 설정에 대한 참조사이트
[hexo utterances 댓글 추가](https://swtpumpkin.github.io/git/hexo/hexoCommentUtterances/)

이제 tranquilpeak 테마에 적용하기 위한 작업은 아래와 같다.
#### utterances.ejs 파일 생성

위 링크대로 진행하게 되면 최종 script가 생성되는데 , Copy 버튼을 눌러 복사한 후
themes/tranquilpeak/layout/_partial/post 폴더 아래에 utternaces.ejs 파일을 하나 생성한 다음
복사한 script를 붙여넣고 저장한다.

{% codeblock themes/tranquilpeak/layout/_partial/post/utterances.ejs %}

    <script src="https://utteranc.es/client.js"
            repo="{userid}/blog-comments"
            issue-term="title"
            theme="github-light"
            crossorigin="anonymous"
            async>
    </script>

{% endcodeblock %}

위 repo의 {userid}는 본인의 github의 사용자 계정명이다.

#### _config.yml 에 utterances enable:true 설정

아래 comment systems 주석을 찾아서 utterances 항목처럼 추가

{% codeblock themes/tranquilpeak/_config.yml %}
# ---------------
# Comment systems
# ---------------
...
utterances:
    enable: true
...
{% endcodeblock %}

#### post.ejs 변경

아래 post.comments 를 찾아 else if 구문처럼 추가

{% codeblock themes/tranquilpeak/layout/_partial/post.ejs %}

    <article class="post">
        ...
        <% if (post.comments) { %>
            ...
            <% } else if (theme.utterances.enable) { %>
                <%- partial('post/utterances') %>
            <% } %>
        <% } %>
    ...
{% endcodeblock %}

#### 로컬에서 최종 적용 확인 및 deploy
``` bash
$ hexo s
$ hexo g
$ hexo d
```

Done.