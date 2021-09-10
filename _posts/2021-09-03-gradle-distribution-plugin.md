---
layout: post
title:  "Gradle distribution plugin으로 배포판 만들기"
published: false
categories:
  - gradle
tags:
  - 빌드/배포
---

### 1. 개요
개발한 프로그램은 단독 빌드 파일만으로는 배포하기 어려울 때가 많다.

기동 스크립트가 필요하기도 하고, 프로덕션 버전의 설정 파일이 필요할 때도 있다.
때로는 프로그램 기동에 필요한 sql이 필요하기도 하다.

maven은 [assembly plugin](http://maven.apache.org/plugins/maven-assembly-plugin/)이 있어, assembly할 
내용을 xml로 작성해 이와 같은 작업이 가능하다.

그렇다면 **gradle**은 어떻게 해야할까?

그래들은 이와 같은 작업을 위해 <ins>distribution plugin</ins>을 제공한다.
메이븐처럼 별도의 <var>assembly.xml</var>이 필요하지 않고 <var>build.gradle</var>에 작성할 수 있어 훨씬 간결하게 사용할 수 있다.

<br/>

### 2. 플러그인 적용
```
plugins {
    id 'distribution'
}
```
build.gradle에 **distribution plugin**을 적용한다.

상위 플러그인의 적용만으로도 즉시 관련 task를 사용할 수 있다.

배포 파일을 zip파일로 생성하면 <var>gradle distZip</var>을, tar로 생성하면 <var>gradle distTar</var>를 사용하면 된다.
물론, <var>gradle assembleDist</var> 명령으로 zip과 tar를 한번에 생성할 수도 있다.

해당 명령어를 이용하면 즉시 배포판이 생성되어 <var>build/distribution</var>디렉토리 하위에 배포판이 생성된다.
> 빌드 경로를 기본 경로가 아닌 다른 경로로 변경했다면 위치가 변경된다.

<br/>

### 3. 배포판 파일 정의
```bash
distributions {
    main {
        contents {
            from {"${project.rootDir}/back/scripts/boot.sh" }
            fileMode = 0755

            into('config') {
                from {"${project.rootDir}/back/src/main/resources/application.yml" }
            }
            into('lib') {
                from {"${project.rootDir}/back/build/libs/server-${version}.jar" }
            }
        }
    }
}
```

<ins>src/main/dist</ins>에 위치한 파일은 기본적으로 배포판에 포함되지만, 다른 파일은 직접 지정해주어야 한다.

위와 같이 build.gradle에 하위 디렉토리도 지정 가능하며, fileMode를 통해 파일 권한 설정도 가능하다.
저렇게 해두면 어느 계정에서도 **boot.sh**의 읽기, 실행이 가능해진다.

상기와 같이 지정한 배포파일을 압축 해제했을 시의 디렉토리 구조는 아래와 같다.
```
drwxr-xr-x 1 HK 197121   0  9월  3 18:53 config/
drwxr-xr-x 1 HK 197121   0  9월  3 18:53 lib/
-rwxr-xr-x 1 HK 197121 631  9월  3 18:59 boot.sh*
```

<br/>

### 4. 결론
maven의 assembly plugin도 유용하게 사용했었지만,
xml로 설정하는 특성상 설정파일의 길이가 길어지는 점과, 별도의 파일이 생성되는 점이 항상 불만이었다.

그런데 gradle distribution plugin은 훨씬 간결하게 작성할 수 있어 더욱 유용하게 사용할 수 있을 것 같다.
