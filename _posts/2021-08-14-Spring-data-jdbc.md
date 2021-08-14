---
layout: post
title:  "Spring data jdbc적용 후기"
---

# Spring data jdbc
- Spring data jdbc는 spring data jpa처럼 db연동 시 사용할 수 있는 spring프레임워크이다.
- **도메인 중심 설계**에 맞춰진 db사용 시 적합하다.
- 후술하겠지만 Jpa보다 훨씬 기능이 없다. 그렇지만 그 덕에 jpa보다 훨씬 사용하기가 쉽다. 
 처음 jpa공부할 때의 당황스러움은 없을거라고 생각되나, jpa와는 사용법이 다른 부분이 많아 헷갈림..
<br/>


## 요구사항
- JDK 8이상
- Spring 5.3.9이상
<br/>


## 지원DB
- DB2
- H2
- HSQL DB
- Maria DB
- Microsoft SQL Server
- MySQL
- Oracle
- Postgres
<br/>


## jpa와 비교

||JPA|Spring data jdbc|비고|
|----|----|----|----|
|**영속성 컨텍스트**|있음|없음||
|**지연로딩**|가능|불가|jdbc는 기본 Eager|
|**스키마 자동생성**|가능|불가|jdbc는 flyway나 liquibase필요|
|**dialect설정**|외부 설정 가능|캡슐화되어있음||
|**복합키**|가능|불가||
|**조인**|entity에 설정(ex. manytoone, onetoone)|jdbc는 스키마에 foreign key설정필요||

<br/>


# 기타
- query dsl지원이 불가하다. 공식문서에 query dsl예제도 나와있으나 실제로 사용해보면 에러가 뜬다.
- [스프링 프로젝트의 깃허브 예제](https://github.com/spring-projects/spring-data-examples)에는 mybatis, jooq과 
  연동된 예시가 있는데, jooq는 오라클을 사용하려면 상용버전을 사용해야 한다.(오픈소스 사용하려면 다 돈인 오라클 ㅜㅜ)
- spring data jdbc도 차후 적용 예제를 깃허브에 업로드할 예정이다.
