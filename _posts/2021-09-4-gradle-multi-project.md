---
layout: post
title:  "Gradle multi project"
---

# 멀티 프로젝트 그래들로 정의하기
자바로 된 프로젝트를 구성할 때, 한가지 모듈만으로 프로젝트를 구성하는 경우는 많지 않다.
보통 root하위에 n개의 모듈이 있는 경우가 대부분인 것 같다.

이번에는 멀티 프로젝트의 경우, 공통 디펜던시를 어떻게 정의할 수 있는지에 대해 포스팅하려고 한다.


## root > settings.gradle
먼저 최상위의 `settings.gradle`에 사용할 모듈을 정의한다.
해당 프로젝트는 a모듈과 b모듈로 구성된다.
```
rootProject.name = 'root'

include('a', 'b')
```


## root > build.gradle
최상위의 `build.gradle`에는 하위 모듈에 공통적으로 적용되는 로직을 작성한다.

해당 프로젝트는 `org.springframework.boot:2.5.4`플러그인이 공통적으로 적용되어있으나,
`apply false`로 자기자신을 제외한 하위 모듈에만 해당 플러그인을 적용하도록 했다.

또한, `subprojects`로 하위 모듈에 디펜던시를 적용한다.

> `subprojects`대신 `allprojects`를 사용하면 자기자신도 포함해 정의할 수 있다.

```
plugins {
    id 'org.springframework.boot' version '2.5.4' apply false
}

subprojects {
    apply plugin: 'java'

    dependencies {
        annotationProcessor 'org.projectlombok:lombok'
        compileOnly 'org.projectlombok:lombok:1.18.20'

        testCompileOnly 'org.projectlombok:lombok:1.18.20'
        testAnnotationProcessor 'org.projectlombok:lombok:1.18.20'
    }
}

```


## a > build.gradle
이제 a모듈의 build.gradle에는 하기와 같이 정의한다.
상위 설정과는 다르게 버전이 누락되어있는데, 이렇게되면 상위와 동일하게 설정되게 된다.

```
plugins {
    id 'org.springframework.boot'
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
}

```
