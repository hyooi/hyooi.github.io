---
layout: post
title:  "PostConstruct와 PreDestory"
---

### PostConstruct
PostConstruct어노테이션이 붙은 메소드는 spring **bean이 생성된 직후 한번 호출**된다. 
하단의 예제는 EmployeeManagerImpl bean을 생성한 이후 onInit메소드를 한번 호출한다.

아래와 같이 초기화 대상이 없는 경우에도 실행되며, 
public/private/default/protected 모두 가능하지만 static은 사용 불가하다.

{% highlight java %}
@Service
class EmployeeManagerImpl implements EmployeeManager {

    @PostConstruct
    public void onInit() {
        System.out.println("EmployeeManagerImpl bean is created.");
    }
}
{% endhighlight %}
<br/>

### PreDestroy
PreDestroy어노테이션이 붙은 메소드는 PostConstruct와는 반대로 **bean이 삭제되기 직전에 실행**된다.
PostConstruct와 마찬가지로 public/private/default/protected 모두 가능하지만 static은 사용 불가하다.

{% highlight java %}
@Component
public class UserRepository {

    private DbConnection dbConnection;

    @PreDestroy
    public void preDestroy() {
        dbConnection.close();
    }
}
{% endhighlight %}
<br/>

### Java 소멸자와 PreDestroy
상기 어노테이션들은 각각 자바의 생성자, 소멸자와 유사하게 느껴진다.
그러나 이펙티브 자바에서는 소멸자를 사용해 오브젝트의 리소스를 닫는 로직은 권장하지 않는다.
가비지 컬렉터가 언제 해당 오브젝트를 메모리에서 삭제할 지 알 수가 없기 때문.

유사한 이유로 상단의 예제처럼 DBConnection을 bean소멸 시기에 닫는 로직은 바람직하지 않은 듯 하다.
(물론 실무에서 저렇게 커넥션을 관리하는 로직을 짤 일은 없겠지만...)
<br/><br/>

### PostConstruct, PreDestroy 사용이 불가능할 때
JDK11이상을 사용하는 경우, PostConstruct와 PreDestroy가 jdk에 포함이 되어있지 않다.

그 이유는 해당 어노테이션들이 포함된 Java EE가 Java9부터는 deprecated되었고, java11부터는 삭제되었기 때문이다.

따라서 해당 어노테이션을 사용하기 위해서는 해당 디펜던시를 별도로 추가해주어야 한다.

{% highlight xml %}
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
{% endhighlight %}