---
layout: post
title:  "스프링 lazy init"
---

# 1. Lazy init
- Ioc컨테이너에 빈을 느리게 초기화하도록 함
- 실제 bean에 처음 요청할 때 bean이 초기화됨
<br/><br/>


# 2. 설정방법
## 1-1. 소스상 설정
- 클래스가 아닌 개별 bean혹은, @autowired시 정의할 수도 있음 
- @autowired시 정의하기 위해서는 주입되는 클래스와 주입하는 클래스 양쪽에 lazy정의 필요

{% highlight java %}
@Lazy
@Configuration
@ComponentScan(basePackages = "com.baeldung.lazy")
public class AppConfig {

    @Bean
    public Region getRegion(){
        return new Region();
    }

    @Bean
    public Country getCountry(){
        return new Country();
    }
}
{% endhighlight %}
<br/>

## 1-2. xml 설정
{% highlight java %}
default-lazy-init = "true"
{% endhighlight %}
<br/>

## 1-3. spring boot설정
### 1-3-1. application.yml
{% highlight java %}
spring:
  main:
    lazy-initialization: true
{% endhighlight %}
<br/>

### 1-3-2. SpringApplicationBuilder
- 우선순위 : SpringApplicationBuilder 및 SpringApplication설정 < application.yml설정
{% highlight java %}
SpringApplicationBuilder(Application.class)
  .lazyInitialization(true)
  .build(args)
  .run();
{% endhighlight %}
<br/>

### 1-3-3. SpringApplication
{% highlight java %}
SpringApplication app = new SpringApplication(Application.class);
app.setLazyInitialization(true);
app.run(args);
{% endhighlight %}
<br/>

# 3. lazy init효과
1. 시작 시 생성되는 빈의 수를 줄일 수 있으므로 **시작 시간 개선**
2. bean생성 시 발견되는 버그가 어플리케이션 기동 시점이 아닌, **런타임 시 발견**됨
3. 웹 어플리케이션에서 요청 시 빈을 생성하는 경우, **http 대기시간 증가**