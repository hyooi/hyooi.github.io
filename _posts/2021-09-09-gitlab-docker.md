---
layout: post
title:  "gitlab ce 도커로 설치하기"
categories:
  - git
tags:
  - 빌드/배포
---

### 1. 개요
git은 버전관리 시스템 중 현재 가장 인기있는 소프트웨어이다.
리눅스를 만들었고, 능력은 있지만 성질있기로 유명한 **그** 리누스 토발스가 만들었다.

오늘은 git을 호스팅할 수 있는 소프트웨어인 <ins>gitlab을 도커로 설치하는 방법</ins>을 정리해보려고 한다.

<br/>

### 2. gitlab 도커 설치
해당 내용은 [공식 문서](https://docs.gitlab.com/ee/install/docker.html)에 상세히 설명되어 있다.

첫 줄부터 <ins>모든 필요한 서비스를 단일 컨테이너에서 기동하는 깃랩 모놀리틱 이미지</ins>라고 설명하고 있는데,
기동해보면 grafana, prometheus, postgres, 각종 exporter들이 한개의 컨테이너에서 기동된다...wow

개인적으로 도커 이미지 하나에 저렇게 많은 서비스를 포함시키는걸 좋아하진 않는다.
덕분에 gitlab 이미지는 무려 2GB가 넘는다.(gitlab runner도 무거움 ㅋㅋ)

slim버전 좀 내주거나 필요한 것만 띄울 수 있게 해주지..ㅠㅠ

<br/>

#### 2.1. docker-compose up
```yaml
gitlab:
  image: 'gitlab/gitlab-ce:14.1.5-ce.0'
  restart: always
  hostname: '123.456.789.000'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
            external_url 'http://123.456.789.000:1111'
            gitlab_rails['gitlab_shell_ssh_port'] = 2222
  ports:
    - '1111:1111'
    - '2222:22'
  volumes:
    - '/home/gitlab/config:/etc/gitlab'
    - '/home/gitlab/logs:/var/log/gitlab'
    - '/home/gitlab/data:/var/opt/gitlab'
```
언제 버전을 올릴지 모르고, 언제 설정을 변경해 재기동해야할지 모르니 docker compose파일로 정의해 기동하기로 했다.

공식문서에는 친절하게 예제가 올라와있는데, **http port 및 ssh port**를 변경하고자 했으므로 적당히 변경해 사용했다.
상기 내용을 **docker-compose.yml**파일로 작성하면 된다.

외부 접속이 필요하니 http(https), ssh는 포트 바인딩이 필수고,
레파지토리가 얼마나 많아질지, 데이터가 얼마나 커질지 모르니 볼륨 설정도 필수다.
또한 gitlab 커뮤니티 에디션 이미지의 버전은 [도커 허브](https://hub.docker.com/r/gitlab/gitlab-ce)를 참고했다.

> ip, port는 실제와 다릅니다.

<br/>

그리고 실행시키면 끝!
```bash
docker-compose up -d
```

<br/>

#### 2.2. 접속실패!
인줄 알았는데, 접속이 안된다.

알고보니 기본 포트(80, 443, 22)를 사용하지 않는 경우엔 추가 설정이 필요하다고 한다.([공식 문서](https://docs.gitlab.
com/ee/install/docker.html#expose-gitlab-on-different-ports))
이건 직접 컨테이너 내부에 접속해 <var>gitlab.rb</var>파일을 편집해주어야 한다.

##### 2.2.1. 컨테이너 접속
```bash
docker exec -it gitlab /bin/bash
```
gitlab컨테이너에 접속한다. 상위 명령어 중 name을 다르게 주었으면 gitlab이 아닌 다른 이름으로 접속해야 한다.


##### 2.2.2. gitlab.rb편집
```shell
external_url "http://123.456.789.000:1111"
gitlab_rails['gitlab_shell_ssh_port'] = 2222
```
<var>/etc/gitlab/gitlab.rb</var>에서 상기 설정을 찾아 주석을 해제하고 수정해준다.

docker-compose.yml파일에 이미 설정해둔 내용이니 자동적용되는게 맞지않나 싶지만... 
저 설정만으로는 작동이 안된다.


##### 2.2.3. gitlab 재설정
```shell
gitlab-ctl reconfigure
```
<var>gitlab-ctl</var>로 gitlab을 재설정한다.
반드시 **컨테이너 내부에서 실행**해야 한다.

<br/>

### 3. 웹페이지 접속

![gitlab-main](/assets/gitlab-main.png)
이제 <var>http://123.456.789.000:1111</var>로 접속하면 드디어 멀쩡한 로그인 페이지가 등장한다.

무려 <var>complete devops platform</var>이라고 설명하고 있다.
하위 버전에서는 open source software라고 나왔었는데, 그새 더 멋있는 타이틀이 붙었다.

초기 root 패스워드는 하기와 같이 확인할 수 있는데, <ins>24시간 이후면 해당 파일은 삭제</ins>되므로 얼른 접속해서 
root패스워드를 변경해준다.

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

<br/>

### 4. 결론
약간 설정하기 귀찮았던 부분은 있지만 역시 도커가 짱이다. 컨테이너 만세.

지금은 일단 http로 설정해뒀지만, 차후 도메인을 발급받게 되면 인증서를 적용해 https로 접속할 수 있게
설정해야 할 것 같다.

다음번엔 **gitlab runner를 공용 러너로 설정하는 법**에 대해 정리할 예정이다.
