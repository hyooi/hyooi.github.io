---
layout: post
title:  "디자인패턴: Composite"
---

# Composite pattern
> [ Structural pattern ]
>
> 객체들을 트리구조로 구성하여 **부분과 전체를 나타내는 계층 구조**로 만든다.
> 클라이언트에서 개별 객체와 다른 객체들로 구성된 **복합 객체**를 같은 방법으로 다룰 수 있다.


<br/>
### 1. 문제

- 부분-전체 계층구조에서 클라이언트가 어떻게 개별 객체를 처리하도록 할 수 있는가?


<br/>
### 2. UML

![Composite%201586a2940ee34643ab2f10f535949d12/Untitled.png](/assets/images/designpattern/composite1.png)


### 3. 특징

1. 개별 객체와 복합 객체를 모두 담을 수 있는 구조를 제공한다.
2. 클라이언트에서 **개별 객체와 복합 객체를 같은 방식으로 다룰 수 있다**. (클라이언트는 leaf 또는 composite 객체로 작업하는지 알 필요가 없음)
3. 클라이언트는 계층의 최상위 component로 요청하며, 개별객체의 경우 요청을 직접 처리하고, 복합객체의 경우 해당 요청을 하위 component로 재귀적으로 전달한다.


<br/>
### 4. 하위 연산 정의
![Composite%201586a2940ee34643ab2f10f535949d12/Untitled%201.png](/assets/images/designpattern/composite2.png)

- Design for Uniformity
  - 하위 연산은 Component에 정의된다. 
  - 이를 통해 클라이언트는 개별 / 복합 객체를 균일하게 취급할 수 있다. 
  - 그러나 클라이언트가 Leaf에서 하위 연산을 수행할 수 있기 때문에 타입 안전성이 
    상실된다.

- Design for Type Safety
  - 하위 연산은 Composite에서만 정의된다. 
  - 클라이언트는 개별/복합 객체를 다르게 취급해야 한다. 
  - 그러나 Client가 Leaf에서 하위 연산을 수행할 수 없기 때문에 타입 안전성이 확보된다.

- Composite 패턴은 타입 안전성보다 **균일성을 강조**한다.


<br/>
### 5. 장단점
- Advantages (+)
    - 클라이언트는 계층의 모든 오브젝트를 균일하게 처리할 수 있어 **코드가 단순**해진다.
    - 새로운 component를 추가하거나 기존 클래스를 확장할 때, 클라이언트를 변경할 필요가 없다. (OCP)
    - 런타임에 복잡한 계층 구조를 동적으로 구축 및 변경할 수 있다.

- Disadvantages (–)
  - 타입 안정성에 비해 균일성을 강조한다.


<br/>
### 6. 예제

{% highlight java %}
1  package com.sample.composite.basic;
2  public class Client {  
3      public static void main(String[] args) {
4          Component composite2 = new Composite("composite2 ");
5          composite2.add(new Leaf("leaf3 "));
6          composite2.add(new Leaf("leaf4 "));
7          composite2.add(new Leaf("leaf5 "));

8          Component composite1 = new Composite("composite1 ");
9          composite1.add(new Leaf("leaf1 "));
10         composite1.add(composite2);
11         composite1.add(new Leaf("leaf2 "));

12         System.out.println("(1) " + composite1.operation());    
13         System.out.println("(2) " + composite2.operation());
14      }
15  }

1  package com.sample.composite.basic;
2  import java.util.Collections;
3  import java.util.Iterator;
4  public abstract class Component {  
5      private String name;
6      public Component(String name) {  
7          this.name = name;
8      }

9      public abstract String operation();
10      
11     public String getName() {
12         return name;
13     }

14     public boolean add(Component child) {
15         return false;
16     }
17  }

1  package com.sample.composite.basic;
2  public class Leaf extends Component {
3      public Leaf(String name) {  
4          super(name);
5      }

6      @Override
7      public String operation() {
8          return getName();
9      }  
10 }

1  package com.sample.composite.basic;
2  import java.util.*;
3  public class Composite extends Component {  
4      private List<Component> children = new ArrayList<Component>();
5       
6      public Composite(String name) {  
7          super(name);
8      }

9      @Override
10     public String operation() {
11          Iterator<Component> it = children.iterator();
12          String str = getName();
13          Component child;

14          while (it.hasNext()) {
15              child = it.next();
16              str += child.operation();
17          }

18          return str;
19      }

20      @Override
21      public boolean add(Component child) {
22          return children.add(child);
23      }
24  }

{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern