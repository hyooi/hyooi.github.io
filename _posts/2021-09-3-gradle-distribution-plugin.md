---
layout: post
title:  "Gradle distribution plugin"
---

# 그래들 배포 플러그인
개발한 프로그램은 단독 빌드 파일만으로는 배포하기 어려울 때가 많다.

기동 스크립트가 필요할 때가 있고, 프로덕션 버전의 설정 파일이 필요할 때도 있다.
때로는 프로그램 기동에 필요한 sql이 필요하기도 하다.

maven은 assembly plugin이 있어, assembly할 내용을 xml로 작성해 이와 같은 작업이 가능하다.
그렇다면 gradle은 어떻게 해야할까?

그래들은 이와 같은 작업을 위해 distribution plugin을 제공한다.
메이븐처럼 별도의 assembly.xml이 필요하지 않고 `build.gradle`에 작성할 수 있어 훨씬 간결하다.


## 플러그인 적용
```
plugins {
    id 'distribution'
}
```
상위 플러그인의 적용만으로도 간편하게 task를 사용할 수 있다.
zip파일로 배포 시엔 `gradle distZip`을, tar로 배포 시엔 `gradle distTar`를 사용하면 된다.
물론, `gradle assembleDist` 명령으로 zip과 tar모두 배포 가능하다.

해당 명령어를 이용하면 즉시 배포판이 생성되어 `build/distribution`디렉토리 하위에 배포판이 생성된다.


## 배포판 파일 정의
`src/main/dist`에 위치한 파일은 기본적으로 배포판에 포함되나, 다른 파일은 직접 지정해주어야 한다.
하위와 같이 하위 디렉토리도 지정 가능하며, fileMode를 통해 파일 권한 설정도 가능하다.

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


상기와 같이 지정한 배포파일 압축 해제 시의 디렉토리 구조는 아래와 같다.
```
drwxr-xr-x 1 HK 197121   0  9월  3 18:53 config/
drwxr-xr-x 1 HK 197121   0  9월  3 18:53 lib/
-rwxr-xr-x 1 HK 197121 631  9월  3 18:59 boot.sh*
```
