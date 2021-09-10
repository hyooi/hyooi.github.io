---
layout: post
title:  "그래들 멀티 프로젝트 정의하기"
categories:
  - gradle
tags:
  - 빌드/배포
---

### 1. 개요
자바로 된 프로젝트를 구성할 때, 한가지 모듈만으로 프로젝트를 구성하는 경우는 많지 않다.
일반적으로 root하위에 n개의 모듈이 있는 경우가 대부분이다.

이번 포스트는 멀티 프로젝트에서 어떻게 **공통 디펜던시를 정의하는지**에 대한 내용을 다룬다.
> 사용된 그래들 버전:  7.1

<br/>


### 2. root구성
a와 b모듈로 구성된 프로젝트를 구성한다.

여기서 중요한 점은, 최상위 프로젝트(root)에는 <ins>별도의 코드를 작성하지 않는다</ins>는 점이다.
따라서 최상위 설정에는 하위 프로젝트의 설정만 존재한다.

<br/>

#### 2.1. settings.gradle
```bash
rootProject.name = 'root'

include('a', 'b')
```
먼저 최상위의 `settings.gradle`에 사용할 모듈을 정의한다.

<br/>

#### 2.2. build.gradle
```bash
plugins {
    id 'org.springframework.boot' version '2.5.4' apply false
    id 'io.spring.dependency-management' version '1.0.11.RELEASE' apply false
}

subprojects {
    dependencies {
        annotationProcessor 'org.projectlombok:lombok'
        compileOnly 'org.projectlombok:lombok'

        testCompileOnly 'org.projectlombok:lombok'
        testAnnotationProcessor 'org.projectlombok:lombok'
    }
}

```

최상위의 `build.gradle`에는 **하위 모듈에 공통적으로 적용되는 설정**을 작성한다.

해당 프로젝트는 `org.springframework.boot` 및 `io.spring.dependency-management`플러그인이 공통적으로 적용되어있으나,
<ins>apply false</ins>로 **자기자신을 제외한 하위 모듈에만 해당 플러그인을 적용**하도록 했다.

플러그인 적용에 대한 내용은 [공식 문서](https://docs.gradle.org/current/userguide/plugins.
html#sec:subprojects_plugins_dsl)에도 
아래와 같이 안내되어 있다.

> Applying external plugins with same version to subprojects
If you have a multi-project build, you probably want to apply plugins 
> to some or all of the subprojects in your build, but not to the root project. 
> The default behavior of the plugins {} block is to immediately resolve and apply the plugins. 
> But, **you can use the apply false syntax to tell Gradle not to apply the plugin to the current 
> project** and then use the plugins {} block without the version in subprojects' build scripts:

여기서 `io.spring.dependency-management`는 별도로 version을 지정하지 않아도 해당 스프링부트에 맞는 의존성을 자동 관리해준다. 
다른 버전을 지정해줄 수도 있으나, 호환성 관련 문제가 발생할 수 있으므로 기본 버전을 사용하는 것이 권장된다.

또한, <var>subprojects</var>의 <var>dependencies</var>에 하위 모듈에 대한 디펜던시를 적용한다. <var>io.spring.
dependency-management</var> 플러그인 설정에 따라 버전은 설정하지 않았다.

> 만약 <var>subprojects</var>대신 <var>allprojects</var>를 사용하면 자기자신도 포함해 정의할 수 있다.

<br/>

### 3. a프로젝트 구성

#### 3.1. build.gradle
```bash
plugins {
    id 'org.springframework.boot'
    id 'io.spring.dependency-management'
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
}

```
상위와 같이 plugin은 적용해주어야 하나, 상위와는 다르게 버전은 누락되어 있다.
하지만 이미 버전이 명시되어 있으므로 자동 적용된다.

<br/>

### 4. 결론
plugin 적용 옵션과 최상위 dependencies를 이용한 공통 디펜던시 적용법에 대해 포스팅했다.

해당 방법 이외에 root에 ext로 버전을 지정한 후, 해당 버전을 하위모듈에서 사용하는 방법도 있으나,
그보다는 자동으로 버전관리되는 쪽을 택했다.

혹시나 <var>io.spring.dependency-management</var>플러그인 사용이 어려운 경우엔 공통 variable을 사용해도 괜찮을 것 같다.
