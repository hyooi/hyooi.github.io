---
layout: post
title:  "docker gitlab에 smtp적용하기"
categories:
- 서버
tags:
- git
---

### 1. 개요
[서버 인증서 적용기4](/git/nginx/2021/09/17/gitlab-ssl-4.html)에 이어 <ins>gitlab에 smtp서버를 설정</ins>한다.

별도로 사용하는 smtp서버가 없어 기존 **docker compose파일에 smtp 도커 이미지를 추가 적용해 사용**하도록 했다.
아래의 설정은 [gitlab공식문서](https://docs.gitlab.com/omnibus/settings/smtp.html)를 참조했는데,
굳이 나처럼 도커로 smtp를 띄우지 않더라도 gmail, mailgun, aws ses등의 서비스를 이용할 수도 있다.

<br/>

### 2. gitlab smtp적용
#### 2.1. docker-compose.yml
{% highlight yaml linenos %}
gitlab:
  image: 'gitlab/gitlab-ce:14.4.1-ce.0'
  container_name: gitlab
  restart: always
  hostname: 'gitlab.kello.com'
  shm_size: '1gb'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.kello.com'
        gitlab_rails['gitlab_shell_ssh_port'] = 2222
        nginx['enable'] = true
        nginx['listen_https'] = false
        nginx['listen_port'] = 1111
        gitlab_rails['smtp_enable'] = true
        gitlab_rails['smtp_address'] = 'smtp'
        gitlab_rails['smtp_port'] = 25
        gitlab_rails['smtp_domain'] = 'kello.co.kr'
        gitlab_rails['smtp_tls'] = false
        gitlab_rails['smtp_openssl_verify_mode'] = 'none'
        gitlab_rails['smtp_enable_starttls_auto'] = false
        gitlab_rails['smtp_ssl'] = false
        gitlab_rails['smtp_force_ssl'] = false
        gitlab_rails['gitlab_email_from'] = 'gitlab@kello.co.kr'
        gitlab_rails['gitlab_email_reply_to'] = 'noreply@kello.co.kr'

  ports:
      - '1111:1111'
      - '2222:22'
  volumes:
    - '/home/gitlab/config:/etc/gitlab'
    - '/home/gitlab/logs:/var/log/gitlab'
    - '/home/gitlab/data:/var/opt/gitlab'
  links:
      - "smtp:smtp"

smtp:
  image: namshi/smtp
  container_name: smtp
  environment:
    RELAY_NETWORKS: :192.168.0.0/24:10.0.0.0/8
{% endhighlight %}

14~24라인, 33~34라인이 gitlab의 smtp설정이다. 

보통 smtp서비스를 사용하면 인증서가 설정되어 있겠지만, 내부에서 간단하게 사용할 목적이라 인증서는 모두 false해두었다.
위와 같이 설정하면 <var>gitlab@kello.co.kr</var>에서 메일이 오고, 
회신 요청 시 <var>noreply@kello.co.kr</var>로 가게 되는데, 이 메일은 유효하지 않은 도메인을 설정해도 무관하다.

그리고 smtp address를 <var>smtp</var>로 설정했는데, 
이것은 도커 내부 컨테이너에서 smtp서버의 container명을 <var>smtp</var>로 설정했기 때문이다.
실제 gitlab컨테이너에 접속해 <var>ping smtp</var>를 하게 되면, 실제 smtp서버의 ip를 확인할 수 있다.

그리고 smtp서버의 기본 포트가 25이므로 smtp port 또한 25로 설정했다.

smtp는 도커허브에서 가장 별이 많은 [namshi/smtp](https://hub.docker.com/r/namshi/smtp/)이미지를 사용했으며,
<var>links</var>를 걸어야 gitlab이 smtp서버에 정상적으로 요청함에 유의한다.

<br/>

#### 2.2. gitlab.rb편집
2.1의 옵션 역시 도커 컨테이너에 접속해 <var>gitlab.rb</var>를 **직접 수정 후 리로드**해주어야 한다.
기본 <var>gitlab.rb</var>파일은 거의 대부분의 설정이 주석처리되어 있다.

['docker gitlab설치' 챕터의 2.2](/git/2021/09/09/gitlab-docker.html)에서 상세 내용을 참조할 수 있다.

<br/>

### 3. 결론
![mail](/assets/images/git/smtp.PNG)

위와 같이 설정하니, 비밀번호 초기화 및 파이프라인 실패 등의 이슈가 있을 때 메일을 받을 수 있었다.
(이제 더이상 수정할 일이 없길...)
