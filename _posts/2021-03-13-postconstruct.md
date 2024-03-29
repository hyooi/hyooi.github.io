---
layout: post
title:  "PostConstruct와 PreDestory"
categories:
- spring
---

### 1. PostConstruct
spring빈에서 PostConstruct어노테이션이 붙은 메소드는, **spring bean이 생성된 직후 한번 호출**된다. 
따라서 하단의 예제는 EmployeeManagerImpl bean을 생성한 이후 onInit메소드를 한번 호출한다.

아래와 같이 초기화할 대상이 없는 경우에도 실행 가능하며,
public/private/default/protected 모두 가능하지만 <ins>static메소드에는 사용 불가</ins>하다.

```java
@Service
class EmployeeManagerImpl implements EmployeeManager {

    @PostConstruct
    public void onInit() {
        System.out.println("EmployeeManagerImpl bean is created.");
    }
}
```

<br/>

### 2. PreDestroy
PreDestroy어노테이션이 붙은 메소드는 PostConstruct와는 반대로 **bean이 삭제되기 직전에 실행**된다.

PostConstruct와 마찬가지로 public/private/default/protected 모두 가능하지만 static은 사용 불가하다.

```java
@Component
public class UserRepository {

    private DbConnection dbConnection;

    @PreDestroy
    public void preDestroy() {
        dbConnection.close();
    }
}
```

<br/>

### 3. PostConstruct, PreDestroy 사용이 불가능할 때
Jdk11이상을 사용한다면 위의 어노테이션은 jdk에 포함이 되어있지 않다.

그 이유는 해당 어노테이션들이 포함된 <ins>Java EE가 Java9부터는 deprecated</ins>되었고, **java11부터는 삭제**되었기 때문이다.

따라서 해당 어노테이션을 사용하기 위해서는 해당 디펜던시를 별도로 추가해주어야 한다.

```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```
