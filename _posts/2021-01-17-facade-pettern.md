---
layout: post
title:  "디자인패턴: Facade"
---

# Facade pattern
> [ Structural pattern ]
> 
> 서브시스템의 일련의 인터페이스에 대한 **통합된 인터페이스**를 제공한다. 
> 퍼사드는 서브시스템을 보다 쉽게 사용할 수 있는 **고수준의 인터페이스**를 정의한다.


<br/>
### 1. 문제

1. 복잡한 서브시스템에 대해 어떻게 **간단한 인터페이스**를 제공할 수 있는가?
2. 클라이언트와 하위 시스템의 개체 간의 긴밀한 결합을 피할 수 있는 방법은 무엇인가?


<br/>
### 2. UML

![Facade%20786d2b0261ff489cbb7e68b8884e00c5/Untitled.png](/assets/images/designpattern/facade.png)


### 3. 특징

- 서브시스템 클래스를 캡슐화하지 않음. 서브시스템의 기능을 사용할 수 있는 간단한 인터페이스를 제공하므로, 클라이언트에서 특정 인터페이스가 필요한 경우 서브시스템 클래스를 그대로 사용하면 됨.
- **클라이언트 구현과 서브시스템을 분리**할 수 있음. 서브시스템이 크게 변경되더라도 클라이언트의 변경 없이 퍼사드만 바꾸어 사용 가능
- 서브시스템의 클래스는 여러 클래스가 될 수 도 있고, 복잡한 인터페이스를 가지고 있는 단 한 개의 클래스가 될 수도 있음


<br/>
### 4. 최소지식의 원칙(Principle of Least Knowledge, Law of Demeter)
퍼사드 패턴에는 항상 데메테르의 원칙 혹은 , 디미터의 원칙이라고 번역되는 바로 이 법칙의 설명이 동반된다.

최소지식의 원칙이란, **객체 사이의 상호작용은 될 수 있으면 아주 가까운 친구 사이에서만 허용하는 것이 좋다**는 것이다. 객체지향 프로그래밍에서 결합을 최대한 적게 하기 위한 기법이다.

- Advantages(+)
  - 여러 클래스들이 복잡하게 얽혀있는 상황에서, 시스템의 한 부분을 변경했을 때 다른 부분까지 고쳐야 하는 상황을 미리 방지할 수 있음

- Disadvantages(-)
  - 다른 구성요소에 대한 메소드 호출을 처리하기 위해 래퍼 클래스를 더 만들어야 할 수 있어 시스템이 복잡해질 수 있다.

- 가이드라인 
  - 어떤 메소드에서든지 다음 네 종류의 객체의 메소드만을 호출한다.
    - 객체 자체
    - 메소드에 매개변수로 전달된 객체
    - 그 메소드에서 생성하거나 인스턴스를 만든 객체
    - 그 객체에 속하는 구성요소


<br/>
### 5. 장점
- 클라이언트는 Facade 객체를 통해 작업함으로써 **서브시스템에서 분리**된다.
- 고객은 단순한 Facade 인터페이스를 참조하고 알고 있을 뿐이며 복잡한 서브시스템과는 무관하다.
- 이를 통해 고객은 구현, 변경, 테스트 및 재사용하기가 쉬워진다.

<br/>

### 6. 패턴 간 비교

|패턴|용도|
|------|---|
|Decorator|원본 코드를 래핑하여 인터페이스에 동적으로 책임(기능) 추가|
|Adapter|한 인터페이스를 다른 인터페이스로 변환|
|Facade|간소화된 인터페이스 제공|
|Proxy|클래스에 대한 접근 제어|


<br/>
### 7. 예제

{% highlight java %}
1  package com.sample.facade.basic;
2  public class Facade1 extends Facade {  
3      private Class1 object1;
4      private Class2 object2;
5      private Class3 object3;
6   
7      public Facade1(Class1 object1, Class2 object2, Class3 object3) {  
8          this.object1 = object1;
9          this.object2 = object2;
10         this.object3 = object3;
11      }

12      public String operation() {
13          return "Facade forwards to ... "
14                  + object1.operation1()
15                  + object2.operation2()
16                  + object3.operation3();
17      }

18  }

{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern