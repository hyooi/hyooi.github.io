---
layout: post
title:  "디자인패턴: Proxy"
---

# Proxy pattern
> [ Structural pattern ]
> 
> 다른 객체를 대변하는 객체를 통해서 객체에 대한 접근을 제어한다.


<br/>
### 1. UML

![Proxy%205b89551241bf48609bf903e92fb50875/untitled](/assets/images/designpattern/proxy.png)


### 2. 종류

- 원격 프록시 : 클라이언트에서 원격 객체에 대한 접근을 제어한다. (EX. 네트워크를 통해 접근한 결과를 전달)
- 가상 프록시 : 생성하기 어려운 자원에 대한 접근을 제어한다. 객체가 필요하기 전까지 객체의 생성을 미룰 수 있고, 객체 생성 전이나 객체 생성 도중에 객체를 대신하기도 한다.
- 보호 프록시 : 접근 권한이 필요한 자원에 대한 접근을 제어한다.
- 방화벽 프록시, 캐싱 프록시, 스마트 레퍼런스 프록시, 동기화 프록시 등

<br/>
### 3. 장단점

- Advantages (+)
    - 클라이언트는 객체를 직접 사용하는지, 프록시를 사용하는지 알지 못한다. 클라이언트 모르게 객체를 제어할 수 있다.
    - 세부 구현을 클라이언트로부터 숨길 수 있어 구현이나 변경하기에 용이하다.
    - 프록시를 통해 새로운 기능을 추가할 수 있으므로 OCP를 준수하기 용이하다.
- Disadvantages (–)
    - 다른 래퍼를 쓸 때와 마찬가지로, 클래스와 객체의 수가 늘어난다.

<br/>
### 4. 예제

{% highlight java %}
public class Client {
  public static void main(String[] args) {
    Proxy proxy = new Proxy(new RealSubject()); 
    System.out.println(proxy.operation()); 
  }
}

public class Proxy extends Subject {
  private RealSubject realSubject;
    public Proxy(RealSubject subject) {
      this.realSubject = subject; 
    } 

    @Override 
    public String operation() {
      return "Hello World from proxy and " + realSubject.operation();
    }
  }
}

public class RealSubject extends Subject {
  @Override 
  public String operation() {
    return "RealSubject!"; 
  }
}

public abstract class Subject {
  public abstract String operation();
}

{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern