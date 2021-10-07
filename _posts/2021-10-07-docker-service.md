---
layout: post
title:  "도커 이미지가 사라졌어요"
categories:
- docker
tags:
- docker
---

### 1. 개요
그간 너무 무탈하다 싶었다. 출근시간도 전에 슬랙에 알림이 울렸다.

<var>A: gitlab작업 중인가요?</var>

작업담당자인 나는 새됐다를 외치며 PC를 켜서 gitlab에 접속했다. 
그리고 보이는 <var>404 not found</var>(...)

서둘러 도커 상태를 확인하니, gitlab은 커녕 도커로 기동한 **모든 컨테이너가 삭제되었을 뿐 아니라 이미지도 사라져있었다**.
난생 처음 겪는 장애라 혼이 반쯤 빠져나간채로 급한 gitlab부터 복구를 시작했다.

<br/>

### 2. 급한 불부터 끄자
먼저 gitlab을 재기동했다.

다행히 docker compose파일을 만들어두기도 했고, 파일시스템을 마운트시켜뒀기 때문에 이 부분은 5분만에 복구가 가능했다.
같은 방식으로 나머지 컨테이너들도 하나씩 복구를 해나갔다.

<br/>

### 3. 왜 삭제됐지?
복구를 하면서 도커 디스크가 부족한 상태라는 것을 깨달았다.

<var>난 도커 디렉토리 디스크 충분한 디렉토리로 변경해뒀는데?</var>

그 때부터 멘탈이 돌아왔다.
먼저 적용되어있는 도커 파일시스템 기본 경로를 확인했다.
```bash
$ docker info
Server:
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
```

리눅스에서 도커는, 기본 디렉토리를 <var>/var/lib/docker</var>로 설정한다 ([참고](https://docs.docker.
com/config/daemon/#docker-daemon-directory)).
아니나다를까, 도커 데몬 디렉토리가 원복되어있는게 아닌가.

도커는 설정한 <ins>데몬 디렉토리에 컨테이너 및 다운로드한 이미지, 네트워크 설정과 같은 모든 데이터</ins>를 가지고있는데,
디렉토리가 원복되면서 비어있는 경로를 가리키니 당연히 컨테이너가 모두 없는 상황이었다.

변경한 디렉토리에 기존 데이터들이 모두 살아있었고, 다행히 경로를 다시 변경해 원복할 수 있었다.

이 때 종교도 없으면서 하나님 부처님을 찾음...ㅠㅠ

<br/>

### 4. 그럼 왜 원복됐지?
일단 급한 불은 껐지만, 왜 이 설정이 원복이 된건지 찾아야했다.

기본 디렉토리 설정은 도커데몬이 재기동되어야만 적용되기 때문에, 관련한 설정이 있는지 찾아보았다.
도커 이미지 자동 삭제, 자동 원복 등 별별 키워드를 한글과 영어를 섞어가며 구글링을 해봤는데 관련 문서는 나오지 않았다.

그래서 하나씩 다시 찾아보기로 하고, 컨테이너가 언제 내려갔는지 부터 파악하기로 했다.

<br/>

#### 4.1. 힌트 발견1
![grafana.png](/assets/images/grafana.png)
예전에 모니터링 용도로 만들어두었던 대시보드가 큰 역할을 했다.
대시보드를 확인하니, 새벽 6시 20분 경에 사건이 발생했음을 알 수 있었다.

<br/>

#### 4.2. 힌트 발견2
도커 데몬 로그는 리눅스의 경우, <var>/var/log/syslog</var>에서 확인할 수 있다. [공식문서 참조](https://docs.docker.com/config/daemon/#read-the-logs)

vi를 열고 로그를 확인했더니, 해당 시간에 뭔가 살벌한 로그가 쌓여있었다.
{% highlight bash linenos %}
Oct  7 06:17:01 userver CRON[1526502]: (root) CMD (   cd / && run-parts --report /etc/cron.hourly)
Oct  7 06:20:17 userver systemd[1]: Starting Daily apt upgrade and clean activities...
Oct  7 06:20:21 userver systemd[1]: Stopping containerd container runtime...
Oct  7 06:20:21 userver containerd[3263683]: time="2021-10-07T06:20:21.521903779+09:00" level=info msg="Stop CRI service"
Oct  7 06:20:21 userver dockerd[1067496]: time="2021-10-07T06:20:21.521938060+09:00" level=error msg="Failed to get event" error="rpc error: code = Unavailable desc = transport is closing" module=libcontainerd namespace=plugins.moby
Oct  7 06:20:21 userver dockerd[1067496]: time="2021-10-07T06:20:21.522072885+09:00" level=info msg="Waiting for containerd to be ready to restart event processing" module=libcontainerd namespace=plugins.moby
Oct  7 06:20:21 userver dockerd[1067496]: time="2021-10-07T06:20:21.521961985+09:00" level=error msg="Failed to get event" error="rpc error: code = Unavailable desc = transport is closing" module=libcontainerd namespace=moby
Oct  7 06:20:21 userver dockerd[1067496]: time="2021-10-07T06:20:21.522260380+09:00" level=info msg="Waiting for containerd to be ready to restart event processing" module=libcontainerd namespace=moby
Oct  7 06:20:21 userver containerd[3263683]: time="2021-10-07T06:20:21.526758264+09:00" level=info msg="Stop CRI service"
Oct  7 06:20:21 userver containerd[3263683]: time="2021-10-07T06:20:21.526857639+09:00" level=info msg="Event monitor stopped"
Oct  7 06:20:21 userver systemd[1]: containerd.service: Succeeded.
Oct  7 06:20:21 userver systemd[1]: Stopped containerd container runtime.
Oct  7 06:20:22 userver dockerd[1067496]: time="2021-10-07T06:20:22.522262740+09:00" level=warning msg="grpc: addrConn.createTransport failed to connect to {unix:///run/containerd/containerd.sock  <nil> 0 <nil>}. Err :connection error: desc = \"transport: Error while dialing dial unix:///run/containerd/containerd.sock: timeout\". Reconnecting..." module=grpc
Oct  7 06:20:22 userver dockerd[1067496]: time="2021-10-07T06:20:22.522288172+09:00" level=warning msg="grpc: addrConn.createTransport failed to connect to {unix:///run/containerd/containerd.sock  <nil> 0 <nil>}. Err :connection error: desc = \"transport: Error while dialing dial unix:///run/containerd/containerd.sock: timeout\". Reconnecting..." module=grpc
Oct  7 06:20:22 userver dockerd[1067496]: time="2021-10-07T06:20:22.751237175+09:00" level=warning msg="Health check for container 979e41048696965d85a4a1b1c8c6e5707f381aa097763cd0421cb10dfd31a9a6 error: connection error: desc = \"transport: Error while dialing dial unix:///run/containerd/containerd.sock: timeout\": unavailable"
Oct  7 06:20:24 userver systemd[1]: Reloading.
Oct  7 06:20:25 userver dockerd[1067496]: time="2021-10-07T06:20:25.126248796+09:00" level=warning msg="grpc: addrConn.createTransport failed to connect to {unix:///run/containerd/containerd.sock  <nil> 0 <nil>}. Err :connection error: desc = \"transport: Error while dialing dial unix:///run/containerd/containerd.sock: timeout\". Reconnecting..." module=grpc
Oct  7 06:20:25 userver systemd[1]: Starting Message of the Day...
Oct  7 06:20:25 userver dockerd[1067496]: time="2021-10-07T06:20:25.273284296+09:00" level=warning msg="grpc: addrConn.createTransport failed to connect to {unix:///run/containerd/containerd.sock  <nil> 0 <nil>}. Err :connection error: desc = \"transport: Error while dialing dial unix:///run/containerd/containerd.sock: timeout\". Reconnecting..." module=grpc
Oct  7 06:20:25 userver dockerd[1067496]: time="2021-10-07T06:20:25.273488765+09:00" level=warning msg="Health check for container f7c4b4b8c21f270785f601e75ce6c1f748a7f30c223287e287b0cd21554af4e7 error: connection error: desc = \"transport: Error while dialing dial unix:///run/containerd/containerd.sock: timeout\": unavailable"
Oct  7 06:20:25 userver systemd[1]: Reloading.
Oct  7 06:20:25 userver systemd[1]: Started Reminder for degraded MD arrays.
Oct  7 06:20:25 userver systemd[1]: mdmonitor-oneshot.service: Succeeded.
{% endhighlight %}

정확히 OS가 6시 20분부터 **daily apt update**작업을 수행했음을 확인할 수 있었다.
그리고 도커가 이런저런 작업을 한게 보인다.

전날 syslog의 동일 시간과 비교했더니 해당 로그는 확인되지 않았기 때문에,
이 로그가 사건 발생 이유의 근거겠거니 싶었다.

<br/>

#### 4.3. 해결
4.2에서 찾은 키워드로 구글링을 하니, 나같은 장애를 겪은 사람들을 무수히 많이 만났다.ㅠㅠ(키워드의 중요성)

우분투OS는 매일 apt update를 진행해서 업데이트할 내용이 있는지 찾게 되는데,
이 때 도커가 같이 재기동되는 경우가 종종 있다고 한다.

다음과 같이 <ins>daily apt update job을 disable 처리</ins>해주었다.
```bash
$ systemctl list-timers
NEXT                        LEFT                 LAST                        PASSED       UNIT                  
Thu 2021-10-07 20:03:47 KST 6h left              Thu 2021-10-07 11:38:00 KST 1h 27min ago apt-daily.timer      
Fri 2021-10-08 06:26:52 KST 17h left             Thu 2021-10-07 06:20:17 KST 6h ago       apt-daily-upgrade.time>

$ systemctl stop apt-daily-upgrade.timer
$ systemctl list-timers
NEXT                        LEFT                 LAST                        PASSED       UNIT                  
Thu 2021-10-07 20:03:47 KST 6h left              Thu 2021-10-07 11:38:00 KST 1h 28min ago apt-daily.timer       

$ systemctl disable apt-daily-upgrade.timer
Removed /etc/systemd/system/timers.target.wants/apt-daily-upgrade.timer.

$ systemctl daemon-reload
```

<br/>

### 5. 결론
도커가 정말 편하긴 하지만, 뭘 쓰더라도 잘 알고 써야하는 것 같다.

OS업데이트되면서 이런 일이 일어날 줄은 상상도 못했다.
해킹당한건가, 요즘엔 도커 초기화하는 해커도 있나 별별 생각을 다했다.ㅎㅎ

아무튼 원인파악은 됐으니 오늘은 맘편히 잘 수 있을 듯하다.
