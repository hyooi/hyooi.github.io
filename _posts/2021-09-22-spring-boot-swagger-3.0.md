---
layout: post
title:  "스프링부트에 springfox3.0(swagger) 적용하기"
categories:
- Java
tags:
- swagger
- springfox
---

### 1. 개요
오늘은 백엔드api 문서를 자동으로 생성해주는 swagger에 대해 포스팅하려고 한다.

간만에 사용할 일이 생겨 찾아봤더니, swagger2 스펙을 구현한 springfox가 메이저버전이 3점대로 올라가있었다.
찾아보니 spring5 및 webflux를 지원하고, OAS3을 지원한다고 한다.

그리고 적용 방법이 좀 더 간단해진 것 같았다...
오늘은 그에 따라 springfox3.0 적용과정을 포스팅하려고 한다.

<br/>

### 2. swagger적용
#### 2.1. build.gradle
```bash
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    runtimeOnly 'com.h2database:h2'
    runtimeOnly 'com.oracle.database.jdbc:ojdbc8:21.1.0.0'
    runtimeOnly 'mysql:mysql-connector-java:8.0.26'
    implementation 'io.springfox:springfox-boot-starter:3.0.0'
    implementation 'org.apache.commons:commons-lang3'
    implementation 'com.fasterxml.jackson.datatype:jackson-datatype-jsr310'
    implementation 'com.querydsl:querydsl-apt:5.0.0'
    implementation 'com.querydsl:querydsl-jpa:5.0.0'
    implementation 'com.querydsl:querydsl-core:5.0.0'
    implementation 'org.mapstruct:mapstruct:1.4.2.Final'
    implementation 'org.apache.poi:poi:5.0.0'
    implementation 'org.apache.poi:poi-ooxml:5.0.0'

    annotationProcessor 'com.querydsl:querydsl-apt:5.0.0:general'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'

    testImplementation 'com.fasterxml.jackson.datatype:jackson-datatype-jsr310'
}
```

<br/>

#### 2.2. 프로파일별 설정
```java
@Configuration
public class SwaggerConfig {

  @Bean
  @Profile({"local || test || dev"})
  public Docket enable() {
    return new Docket(DocumentationType.OAS_30)
        .select()
        .apis(RequestHandlerSelectors.basePackage("kello.com.api"))
        .build()
        .apiInfo(new ApiInfoBuilder()
            .title("Survey API")
            .version("0.0.1")
            .build());
  }

  @Bean
  @Profile({"!local && !test && !dev"})
  public Docket disable() {
    return new Docket(DocumentationType.OAS_30).enable(false);
  }
}

```
<var>http://localhost:7070/swagger-ui/</var>

<br/>

### 3. 결론
