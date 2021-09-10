---
layout: post
title:  "Liquibase offline모드 적용하기"
published: false
categories:
  - Database
tags:
  - Liquibase
  - 형상관리
---

# DB형상관리?
- 개발을 하는 사람이라면 대부분 형상관리를 위해 git을 사용하는 데 비해, DB형상관리는 보편적이지 않은 것 같다.
flyway나 liquibase를 활용하게 되면, 업무를 하는데 **필요한 기본 dataset을 별도로 관리**하고, **실제 db에 연동해뒀다가 필요할 때 적용**하도록 할 수 있다.
- jpa를 사용하는 경우, 약간의 설정으로도 하이버네이트가 엔티티 설정을 읽어들여 자동으로 ddl을 실행하도록 해줄 수가 있다.
그런데 spring data jdbc같은 경우 이러한 방식이 불가능하므로, 공식 문서에서는 이러한 db형상관리 솔루션을 별도로 사용하는 것을 권장하고 있다.
<br/>


## Liquibase
- flyway와 liquibase를 비교해둔 글을 흔히 찾아볼 수 있다. 사용하기 쉬운 쪽은 flyway인데, liquibase쪽이 기능은 훨씬 많다. 
- 보통 db형상관리는 실제db에 연동해 빌드할 때마다 적용되도록 하는 경우가 많은데, 나는 실제db와는 연동하지 않고 빌드 시 sql을 export하도록 하고자 했다.
따라서 offline모드를 별도로 지원하는 liquibase를 선택했다. 
- 또한 우리팀처럼 다양한 데이터베이스 환경을 지원해야 하는 경우엔 liquibase사용이 적합하다. sql로만 이력관리가 가능한 flyway에 비해, 
liquibase는 xml, json, yml등의 방식으로도 관리가 가능하다. 개인적으로 이 중에는 xml로 설정하는 것이 가장 보기 편한 것 같다... 
<br/>


## Liquibase 적용
- spring boot를 활용한 프로젝트에 바로 liquibase를 붙이기도 하는데, 나같은 경우엔 별도의 erd용 프로젝트를 두고 관리하고 있다.
먼저 build.gradle 파일을 보자. 
{% highlight xml %}
plugins {
    id 'org.springframework.boot' version '2.5.3'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id 'org.liquibase.gradle' version '2.0.4'
}

repositories {
    mavenCentral()
}

dependencies {
    liquibaseRuntime 'org.springframework.boot:spring-boot-starter-data-jpa'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.liquibase:liquibase-core:4.3.5'
    **liquibaseRuntime 'org.liquibase:liquibase-core:4.3.5'**
    testCompileOnly 'org.projectlombok:lombok:1.18.20'
    testAnnotationProcessor 'org.projectlombok:lombok:1.18.20'
    testRuntimeOnly 'com.h2database:h2'
}

clean {
    delete(fileTree('output') {
        include('*.sql', '*.csv')
    })
}

test {
    useJUnitPlatform()
}

liquibase {
    **activities** {
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

updateSQL {
    dependsOn 'clean'
    doLast {
        copy {
            from "${project.projectDir}/output/h2.sql"
            into "${project.rootDir}/server/src/main/resources"
            rename { String fileName ->
                fileName.replace("h2.sql", "schema-h2.sql")
            }
        }
    }
}
{% endhighlight %}
- build파일 중 중요한 것은, **liquibaseRuntime과** **activities이다**. 
- liquibaseRuntime dependency를 적용하게 되면 liquibase의 task를 사용할 수 있게 되는데,
이 중 updateSQL은 오프라인 모드로 적용했을 때 실행 가능한 task이다.
- 그리고 activities에는 dbms별 설정을 추가했다. liquibase가 어떤 dbms를 사용하는지를 확인하고 알아서 그에 맞는 sql을 생성해준다.
- 마지막으로 updateSQL은 수행 전에 무조건 clean task를 실행하게 되어있다.
liquibase offline모드 사용 시 자동으로 변경한 change log를 csv로 export하게되는데, 
해당 csv파일에 이전에 export된 이력이 남아있으면 빈 sql만 출력되기 때문에 무조건 삭제 후 실행하도록 설정했다.
