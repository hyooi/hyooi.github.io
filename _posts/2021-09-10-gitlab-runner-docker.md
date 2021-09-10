---
layout: post
title:  "gitlab runner 도커로 설치 후 gitlab ci설정하기"
categories:
  - git
tags:
  - 빌드/배포
---

### 1. 개요
지난 포스트에 이어 이번엔 gitlab runner를 도커로 설치하는 과정을 정리하려고 한다.

runner는 gitlab의 **ci/cd작업을 실행**해주는 도구이다.
gitlab과 다른 서버에 설치해도 되고,
온프레미스 형태로도 설치 가능하나 <ins>도커로 설치하게 되면 필요한 환경마다 다른 이미지를 사용할 수 있어 편리</ins>하다.

굳이 러너가 설치된 서버에 jdk를 설치하지 않아도, 파이썬을 설치하지 않았어도 그에 맞는 이미지를 사용할 수 있는데,
이 점 때문이라도 도커로 러너를 띄우는게 가장 좋은 것 같다.

<br/>

### 2. gitlab runner 도커 설치
[공식문서](https://docs.gitlab.com/runner/install/docker.html)에는 이번엔 도커 컴포즈파일은 올라와있지 않으나
이번에도 도커 컴포즈파일로 설치하는 방식을 택했다.

<br/>

#### 2.1. docker-compose up
```shell
gitlab-runner:
  image: 'gitlab/gitlab-runner:v14.2.0'
  container_name: gitlab-runner
  restart: always
  environment:
    TZ: Asia/Seoul
  volumes:
    - '/var/run/docker.sock:/var/run/docker.sock'
    - '/home/gitlab-runner/srv/gitlab-runner/config:/etc/gitlab-runner'
```

상위 내용대로 <var>docker-compose.yml</var>를 만들어 <var>docker-compose up</var>한다.

먼저 버전은 [도커허브](https://hub.docker.com/r/gitlab/gitlab-runner/tags?page=1&ordering=last_updated)에 
올라온 우분투 버전 중 가장 최신 버전을 택했는데, 경량리눅스인 알파인 버전이 훨씬 용량은 작긴 했으나.. 
알파인을 쓰면 항상 배보다 배꼽이 더 커지는(?) 상황을 맞이했었기 때문에 이번엔 우분투 버전 중 가장 최신 버전으로 선택했다.
> 도커 이미지에는 보통 lastest버전이 있는데, 버전을 바로 확인하기 어려워 버전이 명시된 이미지로 사용하고 있다.

그리고 runner에서 다른 도커 이미지를 사용하게 되므로, <ins>반드시 host pc의 <var>docker.sock</var>을 마운트</ins>시켜주어야 한다.

<br/>

#### 2.2. 러너 등록
##### 2.2.1. 토큰 확인
먼저 gitlab에 접속해 root로 접속한다.
그리고 **admin area > overview > runners**에 접속하면,
아래처럼 <ins>shared runner</ins>를 등록하는 화면이 나온다.

아직 러너를 설치만 했고 등록하지 않았으므로 no runners found로 조회된다.
> shared runner뿐만이 아니라 group runner, 프로젝트의 specific runner도 지정할 수 있으나,
> 모든 프로젝트에서 사용할 수 있는 runner를 설치하고자 shared runner를 설치했다.

![gitlab-main](/assets/runner0.png)

<br/>

##### 2.2.2. 러너 등록
먼저 설치한 러너의 컨테이너에 접속해 gitlab-runner 등록 커맨드를 입력한다.
```bash
$ docker exec -it gitlab-runner /bin/bash
$ gitlab-runner register
```

2.2.1에서 확인한 gitlab url, 토큰 및 내용을 순서대로 입력한다.

도커 러너는 특별하게 **default image**를 입력하게 되는데,
난 가장 많이 사용하는 [메이븐 이미지](https://hub.docker.com/_/maven?tab=description)를 입력했다.
해당 이미지는 ci설정 yml에서도 별도 설정이 가능하다.

<br/>

##### 2.2.3. 등록된 러너 확인
2.2.1.의 화면에 방금 등록한 러너가 조회되면 정상 등록된 것이다. 
![gitlab-main](/assets/runner1.png)

<br/>

### 3. gitlab-ci.yml설정
실제 러너를 이용해 ci/cd작업을 하기 위해서는 프로젝트의 최상위에 <var>gitlab-ci.yml</var>파일을 추가해주어야 한다.

이 역시 [공식문서](https://docs.gitlab.com/ee/ci/)를 참고해 작업했는데,
도커 러너 사용 시 여러개의 job을 사용하게 되면 **각 job마다 도커 컨테이너가 다르게 기동**됐다.
즉, **이전에 빌드된 파일이 사라지는 것**은 물론이고, 만약 job 간 공유하고자 했던 파일이 있다면 삭제됐다.


무엇보다 중요했던건, 파이프라인마다 .m2를 계속 새로받아온다는 점이었다(...)

디펜던시걸린 라이브러리가 적다면 상관없겠지만,
라이브러리가 늘어나면 늘어날수록 늘어나는 파이프라인 실행 시간에 해결책이 필요했고,
아래의 예제처럼 maven옵션에 레파지토리 경로를 지정해 처리했다.

해당 파일이 커밋되어있으면, runner는 알아서 maven컨테이너를 실행시켜 build, test job을 수행한다.
```shell
variables:
  MAVEN_OPTS: -Dmaven.repo.local=/cache/.m2

default:
  image: maven:3.6.3-jdk-11

stages:
  - build
  - test

Build:
  stage: build
  script:
    - mvn install -DskipTests

Test:
  stage: test
  script:
    - mvn test
```

<br/>

### 4. 결론
jenkins도 사용해봤고 gitlab ci, github action도 사용해봤었는데,
아직까지는 그래도 <ins>jenkins가 기능이 많다.</ins>(gitlab ce만 써서 모르는 것일 수도 있다.)

그렇지만 git이 호스팅되는 화면에서 1. job결과를 바로 확인할 수 있고,
그다지 2. 어렵지 않게 적용이 가능하며, 3. 별도의 소프트웨어를 설치하지 않아도 된다는 점에서는
gitlab ci와 github action도 추천하고 싶다.

무료 소프트웨어 중에서는 아직까진 가장 마음에 들어서,
당분간은 계속 gitlab ci를 사용하지 않을까 싶다.
