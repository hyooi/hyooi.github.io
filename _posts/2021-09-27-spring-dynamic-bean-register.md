---
layout: post
title:  "Spring bean 동적 등록/삭제"
categories:
- Java
tags:
- Intellij
---

### 1. 개요
스프링은 ApplicationContext의 beanFactory에서 오브젝트를 빈으로 관리한다.

해당 빈들은 어플리케이션이 기동될 때 @Component, @Controller, @Service 등이 붙은 클래스를 찾아 등록되는데,
오늘은 그 빈들을 동적으로 관리하는 방법에 대해 정리해보려고 한다.

<br/>

### 2. 스프링 빈 동적 등록/삭제하기
#### 2.1. 빈 등록
{% highlight java linenos %}
@SpringBootTest
@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)
@Slf4j
public class DynamicBeanRegisterTest {

  @Autowired
  private ApplicationContext applicationContext;

  @Test
  void test() {
    val appContext = (ConfigurableApplicationContext) applicationContext;
    val beanFactory = appContext.getBeanFactory();
    val beanName = "testBean";

    assertThat(beanFactory.containsBean(beanName)).isFalse();

    beanFactory.registerSingleton(beanName, new Person("kello"));

    val person = beanFactory.getBean(beanName, Person.class);
    assertThat(beanFactory.containsBean(beanName)).isTrue();
    assertThat(person.name).isEqualTo("kello");
  }

  @RequiredArgsConstructor
  private static class Person {
    private final String name;
  }
} 
{% endhighlight %}
먼저 런타임에 빈을 등록하는 방법이다.

17라인에 보면 beanFactory의 registerSingleton메소드를 통해 kello라는 이름을 가진 Person오브젝트를 등록한다.

확인해보면 <var>DefaultSingletonBeanRegistry</var>클래스에서 오브젝트를 등록하는데,
**실제 object는 ConcurrentHashMap으로 캐시해두고, 이름은 별도로 LinkedHashSet을 통해 저장**한다.

이름만 저장한 LinkedHashSet은 레지스트리에 등록된 빈네임이나, 사이즈를 리턴할 때 사용된다.
실제 object를 가지고 있는 ConcurrentHashMap의 접근을 최소화하기 위함인 것으로 판단된다.

확인해보면 registerSingleton하기 이전에는 없었던 빈이(15라인), 등록 후에는 있는 것을 확인할 수 있다.(20라인)

<br/>

#### 2.2. 빈 삭제
{% highlight java linenos %}
@Test
void remove_test() {
    val appContext = (ConfigurableApplicationContext) applicationContext;
    val beanFactory = (DefaultListableBeanFactory) appContext.getBeanFactory();
    val beanName = "testBean";
    
    beanFactory.registerSingleton(beanName, new Person("kello"));
    assertThat(beanFactory.containsBean(beanName)).isTrue();
    
    beanFactory.destroySingleton(beanName);
    assertThat(beanFactory.containsBean(beanName)).isFalse();
}
{% endhighlight %}
반대로 등록한 빈을 삭제하기 위해서는 destroySingleton메소드를 사용하면 된다.

<var>ConfigurableListableBeanFactory</var>인터페이스에서는 해당 메소드를 접근할 수 없으므로,
<var>DefaultListableBeanFactory</var>로 캐스팅해 사용하도록 한다.

등록 시엔 있었던 빈이(8라인), 삭제 후엔 없는 것(11라인)을 확인할 수 있다.

<br/>
