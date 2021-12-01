---
layout: post
title:  "Nginx ip접속 시 도메인 강제 리다이렉트"
categories:
- 서버
tags:
- nginx
---

### 1. 개요
이전 포스트인 [Nginx http to https 강제 리다이렉트](https://hyooi.github.io/%EC%84%9C%EB%B2%84/2021/12/01/nginx-http-to-https.html)와 합치려다가,
별도의 포스트로 분리하면서 분량이 많이 적어졌다.

이번에는 발급받은 도메인이 아닌 **ip를 이용해 접속했을 때, nginx에서 강제로 도메인으로 포워딩**하도록 설정한다.

<br/>

### 2. nginx설정
역시 <var>/etc/nginx/nginx.conf</var>를 편집한다.

이전 포스트와 거의 유사한데, http나 https로 ip(15.0.0.2)를 접속했을 시,
강제로 <var>https://gitlab.kello.com</var>로 리다이렉트했다.

```bash

http {
    server {
        listen 80;
        listen 443 ssl;

        ssl_certificate      /etc/letsencrypt/live/gitlab.kello.com/fullchain.pem;
        ssl_certificate_key  /etc/letsencrypt/live/gitlab.kello.com/privkey.pem;

        server_name 15.0.0.2;

        return 301 https://gitlab.kello.com;
    }
}
```

<br/>

### 3. 유효성 검사 및 리로드
역시 유효성 검사 후 변경된 설정을 리로드한다.

```bash
$ nginx -t
$ systemctl reload nginx
```
