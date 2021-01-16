---
layout: post
title:  "디자인패턴: Mediator"
---

# Mediator pattern
> [ Behavioral pattern ]
> 
> 서로 관련된 **객체 사이의 상호 작용**을 캡슐화한다.


<br/>
### 1. 문제

1. 상호작용하는 객체들 사이의 밀접한 결합을 어떻게 피할 수 있는가?
2. 객체 간의 상호작용을 어떻게 독립적으로 변경할 수 있는가?


<br/>
### 2. UML

![Mediator](/assets/images/designpattern/mediator.png)

### 3. 특징

1. 객체는 서로 직접적으로 상호작용하는 대신, Mediator와 상호작용한다.
2. 새로운 colleague를 추가할 수 있고, 새로운 mediator의 서브클래스를 통해 기존 colleague들의 상호작용을 독립적으로 변경할 수 있다.
3. SRP, OCP


<br/>
### 4. 장단점

- Advantages (+)
    - 각 객체를 분리시킴으로써 구현, 변경, 테스트, 재사용 용이
    - 제어 로직을 모아두기 때문에 관리 수월

- Disadvantages (–)
    - Colleage의 복잡성과 갯수에 따라 미디에이터 객체가 너무 복잡해질 수 있음.


<br/>
### 5. 패턴 간 비교

|패턴|용도|
|------|---|
|Observer|객체간 동적 단방향 연결을 설정|
|Mediator|시스템 구성 요소간의 상호 종속성 제거. 대신 컴포넌트들은 단일 mediator 오브젝트에 종속됨|


<br/>
### 6. 예제

{% highlight java %}
1  public class MyApp {  
2      public static void main(String[] args) {  
3          Mediator1 mediator = new Mediator1();  
4          Colleague1 c1 = new Colleague1(mediator); 
5          Colleague2 c2 = new Colleague2(mediator); 
6        
7          mediator.setColleagues(c1, c2); 
8          
9          System.out.println("(1) Changing state of Colleague1 ..."); 
10         c1.setState("Hello World1!"); 
11          
12         System.out.println("\n(2) Changing state of Colleague2 ..."); 
13         c2.setState("Hello World2!"); 
14      } 
15  } 

1  public abstract class Mediator {  
2      public abstract void mediate(Colleague colleague); 
3  }  

1  public class Mediator1 extends Mediator {  
2      private Colleague1 colleague1; 
3      private Colleague2 colleague2; 
4
5      void setColleagues(Colleague1 colleague1, Colleague2 colleague2) {  
6          this.colleague1 = colleague1; 
7          this.colleague2 = colleague2; 
8      }  
9
10     public void mediate(Colleague colleague) { 
11         System.out.println("Mediator  : Mediating the interaction ..."); 
12         if (colleague == colleague1) {  
13              String state = colleague1.getState(); 
14              colleague2.action2(state); 
15         } 
16
17         if (colleague == colleague2) { 
18              String state = colleague2.getState(); 
19              colleague1.action1(state); 
20         } 
21     } 
22  }

1  public abstract class Colleague {  
2      Mediator mediator; 
3      public Colleague(Mediator mediator) {  
4          this.mediator = mediator; 
5      }  
6  }  

1  public class Colleague1 extends Colleague {  
2      private String state;
3
4      public Colleague1(Mediator mediator) {  
5          super(mediator);
6      }  
7
8      public String getState() {  
9          return state; 
10     } 
11
12     void setState(String state) { 
13          if (state != this.state) { 
14              this.state = state; 
15              System.out.println("Colleague1: My state changed to: " 
16                      + this.state + " Calling my mediator ..."); 
17              mediator.mediate(this); 
18          } 
19      } 
20      void action1 (String state) { 
21          this.state = state; 
22          System.out.println("Colleague1: My state synchronized to: " + this.state); 
23      } 
24  }  

1  public class Colleague2 extends Colleague {  
2      private String state; 
3
4      public Colleague2(Mediator mediator) { 
5          super(mediator); 
6      }  
7
8      public String getState() {  
9          return state; 
10     } 
11
12      void setState(String state) { 
13          if (state != this.state) { 
14              this.state = state; 
15              System.out.println("Colleague2: My state changed to: " 
16                      + this.state + " Calling my mediator ..."); 
17              mediator.mediate(this); 
18          } 
19      } 
20      void action2 (String state) { 
21          this.state = state; 
22          System.out.println("Colleague2: My state synchronized to: " + this.state); 
23      } 
24  }
{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.
  
  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern