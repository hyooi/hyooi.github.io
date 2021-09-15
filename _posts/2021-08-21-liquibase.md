---
layout: post
title:  "Liquibase offline 적용하기"
categories:
  - Sql
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

답은 **아니다**이다. 같은 RDBMS라도 문법이 조금씩 다르기 때문에, 각각 맞는 SQL을 별도로 관리해주어야 한다.
이 때, liquibase를 사용하면 <ins>sql이 아닌 xml이나 json으로 데이터셋을 관리</ins>하고,
**각각의 DBMS에 맞는 sql을 자동 생성**되도록 할 수가 있다.
> jpa를 사용하면 하이버네이트가 엔티티 설정을 통해 자동으로 ddl을 실행하도록 해줄 수가 있다.
> 그런데 spring data jdbc같은 경우 이러한 방식이 불가능하므로, 공식 문서에서는 flyway나 liquibase같은 db형상관리 솔루션을 별도로 사용하는 것을 
> 권장하고 있다.

<br/>

### 2. Liquibase 오프라인 모드
flyway와 liquibase를 비교해둔 글을 흔히 찾아볼 수 있다. 사실 사용하기 쉬운 쪽은 flyway인데, liquibase쪽이 기능은 훨씬 많다.

이러한 일반적인 DB형상관리 솔루션 적용 사례는 <ins>실제DB에 연동해 빌드 시 변경 내용을 자동 적용</ins>하도록 하는 것이다.
그런데 나는 실제DB에는 연동하지 않았고, 빌드 시 XML설정파일을 읽어 SQL을 파일로 생성되도록 하고자 했다.
관리자가 직접 확인한 후 실행하길 의도한 것이다.

따라서 **offline모드**를 별도로 지원하는 <ins>liquibase</ins>를 선택했다.  

<br/>

### 3. build.gradle

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

clean {
    delete(fileTree('output') {
        include('*.sql', '*.csv')
    })
}

liquibase {
    activities {
        def changeLog = "${project.name}/src/main/resources/changelog/master.xml"
        oracle {
            changeLogFile "${changeLog}"
            url "offline:oracle?changeLogFile=${project.name}/output/changelog1.csv"
            outputFile 'output/oracle.sql'
        }

        mysql {
            changeLogFile "${changeLog}"
            url "offline:mysql?changeLogFile=${project.name}/output/changelog2.csv"
            outputFile 'output/mysql.sql'
        }

        h2 {
            changeLogFile "${changeLog}"
            url "offline:h2?changeLogFile=${project.name}/output/changelog3.csv"
            outputFile 'output/h2.sql'
        }
    }
}
```
- build파일 중 중요한 것은, **liquibaseRuntime과** **activities이다**. 
- liquibaseRuntime dependency를 적용하게 되면 liquibase의 task를 사용할 수 있게 되는데,
이 중 updateSQL은 오프라인 모드로 적용했을 때 실행 가능한 task이다.
- 그리고 activities에는 dbms별 설정을 추가했다. liquibase가 어떤 dbms를 사용하는지를 확인하고 알아서 그에 맞는 sql을 생성해준다.
- 마지막으로 updateSQL은 수행 전에 무조건 clean task를 실행하게 되어있다.
liquibase offline모드 사용 시 자동으로 변경한 change log를 csv로 export하게되는데, 
해당 csv파일에 이전에 export된 이력이 남아있으면 빈 sql만 출력되기 때문에 무조건 삭제 후 실행하도록 설정했다.

<br/>

### 4. 결론
