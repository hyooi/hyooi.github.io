---
layout: post
title:  "IntelliJ로 웹프로젝트 기동하기"
categories:
- Java
tags:
- Intellij
---

### 1. 개요
WAS에 올려 기동하던 일반 스프링 웹 프로젝트를 IntelliJ로 기동할 일이 생겼다.

IntelliJ뿐만 아니라 Android studio 및 Pycharm, WebStorm 등에서도 사용 가능한 플러그인인 IDEA Jetty Runner의
사용방법을 정리해본다.

<br/>

### 2. Jetty runner적용
#### 2.1. IDEA Jetty Runner설치
![jetty-runner](/assets/images/jetty-runner.png)
<var>File > Settings > Plugins</var>에서 IDEA Jetty Runner를 설치한다.

<br/>

#### 2.2. Jetty runner설정
![jetty-runner](/assets/images/jetty-runner2.png)
설치 후 <var>Run > Edit Configurations</var>에서 +버튼을 클릭한 후, 
**Jetty Runner**를 클릭하면 러너 설정화면을 확인할 수 있다.

상단 캡쳐처럼 설정하면 되는데, 
<ins>jetty runner jar파일은 maven레파지토리에서 직접 다운로드받아 설정</ins>해주어야 하며,
classes folder에는 빌드된 파일이 위치하는 경로를 설정한다.

이 때, 다중 모듈로 구성되었다면 <var>all modules</var>이 아닌, <var>기동할 모듈</var>을 선택해야 함에 유의하자.
정상 설정했다면 <var>http://localhost:8080</var>에서 기동된 화면을 확인할 수 있다.
> servlet-api는 2.2부터, JAVA는 8부터 지원하며, Window / Mac / Linux에 상관없이 사용 가능하다.
> 다만 jetty jar다운로드 시, Jetty9점대는 자바8부터 지원한다던지 Jetty10은 최소 Java11이 필요하다던지 하는 요구사항이 있으니 이 점만 체크해주면 된다.

<br/>

#### 2.3. 디버그 브레이크포인트 오류
디버그 모드로 러너를 기동해도 브레이크포인트가 동작하지 않는 이슈가 있었다.

깃허브 이슈창을 뒤져보니 IntelliJ가 2020년에 릴리즈된 이후부터 생긴 이슈인 듯 한데, 
2.2의 Configuration에서 VM Args에 아래와 같은 옵션을 추가한 후, **IntelliJ debug창에서 attach debuger**를 클릭하면 정상 동작했다.
```bash
-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=1044
```

<br/>

### 3. 결론
실제 WAS를 기동하고 싶지 않거나 Jetty maven plugin등의 설정을 pom.xml에 추가하고 싶지 않을 때 사용하기 편리하다.

디버그 이슈가 있는 점이 조금 아쉽지만 그 외에 오동작하는 부분은 없는 듯 하다.
