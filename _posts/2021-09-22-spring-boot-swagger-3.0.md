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
    implementation 'io.springfox:springfox-boot-starter:3.0.0'
}
```
spring boot는 적용이 정말 간단하다.

springfox boot starter 디펜던시만 추가해주면, 알아서 관련된 swagger2, swagger ui등의 라이브러리를 자동으로 가져온다.

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
            .title("Kello API")
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
api문서는 **프로덕션 환경에서는 노출되지 않도록 설정**하고자 했다.

스프링 프로파일을 이용해 해결했는데, 상위처럼 <ins>프로파일이 local, test, dev 중 하나인 경우에만 api문서가 활성화</ins>되도록 했다.
그리고 <var>kello.com.api</var> 패키지 하위에 있는 api들만 문서화하도록 설정했다.

이렇게 설정하고나서 별도 포트를 사용하지 않는다면 <var>http://localhost:8080/swagger-ui/</var>로 접속하면 바로 api문서를 
확인할 
수 있다.
> 해당 설정은 spring boot를 사용한 경우에만 해당하며, 스프링만 사용하는 경우엔 별도의 어노테이션을 추가로 설정해주어야 한다.
 
<br/>

### 3. 결론
사실 커스터마이징하게되면 끝도없이 늘어나는 어노테이션 탓에, 개인적으로 swagger를 그닥 좋아하지는 않는다.

그래서 원래는 spring rest docs를 사용해보려고 했는데, 생각보다 작업량이 많아 swagger로 노선을 틀었다.
역시 간단하게 문서화하는데엔 swagger만한 것이 없는 것 같다...

다음에는 꼭 spring rest docs 사용기를 정리해봐야겠다.
