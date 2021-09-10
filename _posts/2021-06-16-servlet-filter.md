---
layout: post
title:  "서블릿 필터"
published: false
---

# 서블릿 필터?
- Java웹앱에서 HTTP통신 시 요청,응답을 사전, 사후 처리한다.
- EX. 로깅, 인증, 세션, 인코딩 등

![img.png](/assets/images/servlet-filter.png)
<br><br>


# 구현방법
### 1. javax.servlet.filter 인터페이스의 구현 
{% highlight java %}
public class FirstFilter implements Filter {
    public void init(FilterConfig fConfig) throws ServletException {
        System.out.println("init()");
    }
 
    // 서블릿이 실행될 때마다 호출되는 메소드
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("before doFilter()");
        chain.doFilter(request, response);
        System.out.println("after doFilter()");
    }
 
    public void destroy() {
        System.out.println("destory()");
    }
}
{% endhighlight %}
<br/>
   
### 2. 필터 등록(webfilter어노테이션을 사용할 수도 있음)
{% highlight xml %}
<filter>
  <filter-name>filter1</filter-name>
  <filter-class>com.FirstFilter</filter-class>
</filter>
{% endhighlight %}
<br/>

### 3. 필터 실행 순서 정의
{% highlight xml %}
<filter-mapping>
  <filter-name>filter1</filter-name>
  <url-pattern>/test</url-pattern>
</filter-mapping>
<filter-mapping>
  <filter-name>filter2</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
{% endhighlight %}
<br/>

# 필터 장점
- 웹앱의 공통 로직 분리
- 각자 다른 기능을 하는 필터를 만들어 필요한 케이스에만 매핑할 수 있음. (전체or일부)
