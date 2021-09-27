---
layout: post
title:  "Spring bean 동적 등록하기"
categories:
- Java
tags:
- Intellij
---

### 1. 개요
스프링은 ApplicationContext의 beanFactory에서 오브젝트를 빈으로 관리한다.

해당 빈들은 어플리케이션이 기동될 때 @Component, @Controller, @Service 등이 붙은 클래스를 찾아 등록되게 되는데,
오늘은 그 빈들을 동적으로 관리하는 방법에 대해 정리한다.

<br/>

### 2. 스프링 빈 동적 등록/삭제하기
#### 2.1. 빈 등록
```java
@SpringBootTest
@Slf4j
public class DynamicBeanRegisterTest {

  @Autowired
  private ApplicationContext applicationContext;

  @Test
  void test() {
    val appContext = (ConfigurableApplicationContext) applicationContext;
    val beanFactory = appContext.getBeanFactory();

    beanFactory.registerSingleton("testBean", new Person("kello"));

    val newBean = beanFactory.getBean("testBean", Person.class);
    assertThat(newBean).isNotNull();
    assertThat(newBean.name).isEqualTo("kello");
  }

  private static class Person {
    private final String name;

    public Person(String name) {
      this.name = name;
    }
  }
}
```
<br/>

#### 2.2. 빈 삭제

<br/>

### 3. 결론
