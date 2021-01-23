---
layout: post
title:  "디자인패턴: Bridge"
---

# Bridge pattern
> [ Structural pattern ]
> 
> 추상화와 구현을 분리하여 각각을 독립적으로 변형할 수 있게 한다


<br/>
### 1. 문제
- 추상화와 구현이 어떻게 독립적으로 정의될 수 있는가?
- 런타임에 구현을 선택하고 교환할 수 있는 방법은 무엇인가?

<br/>
### 2. UML

![Bridge%20afc4ef8cd7bb481eb91997298f869473/Untitled.png](/assets/images/designpattern/bridge.png)

### 3. 특징

- 추상화(Abstraction)과 구현(Implementor)가 분리됨
- Abstraction의 위임을 통해 Impletmentor를 구현함
- 구현 인터페이스는 저수준의 Operation을 제공하며, 추상화는 이러한 요소를 기반으로 고수준의 Operation을 정의한다.


<br/>
### 4. 장단점

- Advantages (+)
  - 구현을 인터페이스에 완전히 결합시키지 않았으므로 구현/추상화된 부분을 분리시킬 수 있음
  - 추상화된 부분과 실제 구현 부분을 독립적으로 확장할 수 있음
  - 추상화된 부분을 구현한 구상 클래스를 바꿔도 클라이언트 쪽에는 영향을 끼치지 않음
- Disadvantages (–)
  - 디자인이 복잡해짐
  
### 4. 예제

{% highlight java %}
1  package com.sample.bridge.basic;
2  public class MyApp {  
3      public static void main(String[] args) {  
4          Abstraction abstraction = new Abstraction1(new Implementor1());
5          System.out.println(abstraction.operation());
6      }
7  }

1  package com.sample.bridge.basic;
2  public interface Abstraction {  
3      String operation();
4  }

1  package com.sample.bridge.basic;
2  public class Abstraction1 implements Abstraction {  
3      private Implementor imp;
4
5      public Abstraction1(Implementor imp) {  
6          this.imp = imp;
7      }  
8
9      public String operation() {           
10          return "Abstraction1: Delegating implementation to an implementor.\n"
11                  + imp.operationImp();
12      }
13  }

1  package com.sample.bridge.basic;
2  public interface Implementor {  
3      String operationImp();
4  }

1  package com.sample.bridge.basic;
2  public class Implementor1 implements Implementor {  
3      public String operationImp() {  
4          return "Implementor1: Hello World1!";
5

{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern