---
layout: post
title:  "Liquibase offline 적용하기"
categories:
  - Database
tags:
  - Liquibase
  - 형상관리
---

### 1. 개요
대부분의 개발자는 개발한 코드를 git, svn 등의 소프트웨어로 형상관리한다. 
회사마다 별도의 상업용 소프트웨어를 구매해 사용하기도 하고, 해당 이력은 항상 중요하게 관리된다.

그에 비해, DB형상관리는 그다지 보편적이지는 않다.

그런데 만약 다양한 종류의 데이터베이스에 맞는 SQL을 관리해야 한다면 어떨까?
1번 서버에는 Oracle, 2번 서버에는 Mysql, 3번 서버에는 H2를 사용한다고 가정하자.
Oracle에서 정상적으로 실행됐던 SQL이 Mysql에서도 정상 실행될까?

답은 **아니다**. 같은 RDBMS라도 문법이 조금씩 다르기 때문에, 각각 맞는 SQL을 별도로 관리해주어야 한다.
이 때, liquibase를 사용하면 <ins>sql이 아닌 xml이나 json으로 데이터셋을 관리</ins>하고,
**각각의 DBMS에 맞는 sql을 자동 생성**되도록 할 수가 있다.
> jpa를 사용하면 엔티티 설정을 통해 자동으로 ddl을 실행하도록 해줄 수가 있다.
> 그런데 spring data jdbc는 이러한 방식이 불가능하므로, 공식 문서에서는 flyway나 liquibase같은 db형상관리 솔루션을 별도로 사용하는 것을 
> 권장하고 있다.

<br/>

### 2. Liquibase 오프라인 모드란?
flyway와 liquibase를 비교해둔 글을 흔히 찾아볼 수 있다. 사실 사용하기 쉬운 쪽은 flyway인데, liquibase쪽이 기능은 훨씬 많다.

이러한 일반적인 DB형상관리 솔루션 적용 사례는 <ins>실제DB에 연동해 빌드 시 변경 내용을 자동 적용</ins>하도록 하는 것이다.
그런데 나는 실제DB에는 연동하지 않고, 빌드 시 XML설정파일을 읽어 SQL을 파일로 생성되도록 하고자 했다.
관리자가 직접 확인한 후 실행하길 의도한 것이다.

따라서 **offline모드**를 별도로 지원하는 <ins>liquibase</ins>를 선택했다.  

<br/>

### 3. build.gradle

#### 3.1. dependencies
```bash
plugins {
    id 'org.springframework.boot' version '2.5.3'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id 'org.liquibase.gradle' version '2.0.4'
}

dependencies {
    liquibaseRuntime 'org.springframework.boot:spring-boot-starter-data-jpa'
    liquibaseRuntime 'org.liquibase:liquibase-core:4.3.5'
}
```
liquibase core 라이브러리를 liquibaseRuntime dependency로 적용하게 되면,
**liquibase에서 제공하는 task**들을 사용할 수 있게 된다. 

해당 문서에서 사용할 <var>updateSQL</var>은 sql을 생성할 때 사용하는 task이다.
<var>gradle updateSQL</var> 커맨드로 실행 가능하며, 반드시 <ins>오프라인으로 설정</ins>을 잡아주어야 한다.
> 다른 태스크는 [공식문서](https://docs.liquibase.com/commands/home.html)를 참조한다.

<br/>

#### 3.2. activities
```bash
liquibase {
    activities {
        def changeLog = "${project.name}/src/main/resources/changelog/master.xml"
        oracle {
            changeLogFile "${changeLog}"
            url "offline:oracle?changeLogFile=${project.name}/output/changelog1.csv"
            outputFile 'output/oracle.sql'
        }

        h2 {
            changeLogFile "${changeLog}"
            url "offline:h2?changeLogFile=${project.name}/output/changelog3.csv"
            outputFile 'output/h2.sql'
        }
    }
}
```
3.1.에서 추가한 <ins>updateSQL</ins> 태스크를 실행하게 되면, 해당 설정에 따라 태스크가 실행되게 된다.

일단 oracle 및 h2에 맞는 sql을 각각 생성하는데, chageLogFile은 <ins>master.xml이라는 동일한 설정 파일을 사용</ins>하고 있다.
그리고 <var>offline:oracle?changeLogFile=${project.name}/output/changelog1.csv</var>와 같이 설정하게되면,
실제 db엔 연동하지 않고 oracle sql을 생성해준다.
> url의 changeLogFile은 **sql의 생성 로그**이다. 해당 파일이 남아있다면 이미 생성한 것으로 간주하고 더 이상 생성하지 않는다.
> 실제 DB에 연동했을 때 사용하는 update태스크는 해당 로그를 DB에 관리한다.

<br/>

#### 3.3. clean
```bash
clean {
    delete(fileTree('output') {
        include('*.sql', '*.csv')
    })
}
```
3.2에서 설명했듯이, 이전 생성 로그가 csv파일로 남아있다면 <ins>sql이 더이상 생성되지 않는다.</ins>
따라서 clean 태스크를 실행할 때마다 이전 sql 및 csv파일을 삭제하고자 했고,
위와 같이 태스크를 재설정했다. 

<br/>


### 4. master.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
  http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

  <include file="./ddl.xml" relativeToChangelogFile="true"/>
</databaseChangeLog>
```
3.2에서 사용하는 master.xml이다. 해당 예제에서는 하나의 xml만을 include하고 있는데,
xml파일마다 context를 설정해 **요청 조건에 따라 다른 xml을 사용**하도록 설정할 수도 있다.

<br/>

### 5. ddl.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
  http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

  <property name="date" dbms="oracle" value="DATE"/>
  <property name="date" dbms="h2" value="TIMESTAMP"/>

  <changeSet author="kello" id="person">
    <comment>사람 테이블</comment>
    <createTable tableName="PERSON">
      <column name="ID" remarks="ID" type="VARCHAR2(10)">
        <constraints nullable="false"/>
      </column>
      <column name="NAME" remarks="이름" type="VARCHAR2(20)"/>
      <column name="AGE" remarks="나이" type="NUMBER"/>
      <column name="REG_DTM" remarks="등록일시" type="${date}"/>
    </createTable>
    <addPrimaryKey columnNames="ID" constraintName="PK_PERSON"
      tableName="PERSON"/>
  </changeSet>

</databaseChangeLog>

```
적용할 sql의 메타데이터를 정의한다.
위와 같이 생성할 테이블 및 기본키를 설정할 수 있는데, 기본적으로 <ins>설정한 dbms에 맞는 sql을 생성</ins>한다.

단, dbms별로 다른 타입을 사용하고자 한다면 위와 같이 property로 설정이 가능하다.

<br/>

### 6. 결론
사실 liquibase는 자체 서비스를 한다면 사용할일이 그다지 많지는 않을거라 예상해본다.
하지만 패키지를 만들어 여러 사이트에 배포하는 입장에서는 굉장히 유용하다.

기본 형상관리 목적 외에도 생성된 sql로 소프트웨어를 dockerize할 수 있었으며,
매일 기동되는 CD프로세스에는 update태스크를 적용해 DB변경내용을 자동 적용할 수 있었다.

<ins>무료버전만으로도 많은 기능을 사용할 수 있다는 점에서 강력 추천</ins>하는데, 
단 가장 최신 버전은 좀 검증이 필요하다.
메이저 버전의 초기 버전은 빌드 속도가 한없이 늦다던지, sql파일이 생성되지 않는 등의 
버그를 발견한 적이 종종 있다..

이러한 점에 유의한다면 편리하게 사용이 가능한 것 같다. :)
