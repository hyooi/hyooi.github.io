---
layout: post
title:  "Nginx http to https 강제 리다이렉트"
categories:
- 서버
tags:
- nginx
---

### 1. http? https?
http는 IT와 관련없는 사람일지라도 본 적이 있을 정도의 필수적인 프로토콜이다.

웹에 있어 없어서는 안될 프로토콜이지만 그 자체로는 보안이 되지 않으므로,
흔히 말하는 **인증서**를 붙여 안전한 통신이 가능하도록 하는데 이 때 쓰는 프로토콜을 **https**라고 한다.

이번에는 nginx를 통해 서버 접근 시 http를 사용했다면,
<ins>https로 강제 리다이렉트</ins>하도록 설정하는 내용을 포스팅하려고 한다.

<br/>

### 2. nginx설정
<var>/etc/nginx/nginx.conf</var>를 편집한다.

<ins>kello.com으로 끝나는 url의 80포트로 접근했다면 무조건 https로 포워딩</ins>하도록 했다.
ex) gitlab.kello.com

http의 기본 포트는 80, https의 기본 포트는 443이므로 가능한 설정이다.
> [301](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/301)은 리다이렉트를 의미하는 http상태코드이다. 

```bash

http {
    server {
        listen 80;
        server_name *.kello.com;

        return 301 https://$host$request_uri;
    }
}
```

<br/>

### 3. 유효성 검사
nginx의 설정 파일이 유효한 상태여야 정상 작동되므로 재기동 전 유효성 검사를 실행한다.

**syntax is ok**로 나타나면 정상적으로 설정된 것이다.

```bash
$ nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

<br/>

### 4. 재기동
nginx서버를 리로드하거나 재기동해야 변경한 설정이 적용된다.

reload는 **기존 커넥션을 종료하지 않으면서** 변경된 설정을 적용하고,
restart는 **기존 커넥션을 종료**하고 nginx서버를 재시작하여 설정을 적용한다는 차이점이 있다.

```bash
$ systemctl reload nginx
$ systemctl restart nginx
```
