---
layout: post
title:  "디자인패턴: Observer"
---

# Observer pattern
> [ Behavioral pattern ]
> 
> 객체 간에 일대다 의존성을 정의하여, 한 객체의 상태가 변경될 때 모든 의존객체들이 자동으로 알림 및 갱신되도록 한다. 
> 이러한 작용을 publish-subscribe라고도 함 ex. GUI



<br/>
### 1. 문제
- 객체 간의 밀접한 결합없이 어떻게 일대다 의존성을 정의할 수 있는가?
- 한 객체를 의존하는 다른 의존 객체들에게 어떻게 통지할 수 있는가?

### 2. UML

![Observer](/assets/images/designpattern/Observer.png)


### 3. 특징

- 이벤트가 발생할 때 등록된 모든 observer에게 자동으로 알려서(update를 호출) 유연한 알림-등록 작용을 하도록 한다.
- Subject와 Observer는 느슨하게 결합되어 있어 구현, 변경, 테스트 및 재사용이 쉽다.
- Observer는 다수가 될 수 있으며 새로운 Observer를 추가할 수 있고, 기존 Observer를 제외할 수도 있다.
- Subject의 상태가 변경될 때 관련 state를 보내는 push방식과, Observer엔 알림만 하고 Observer가 state를 가져가는 pull방식이 있음. 둘 중 pull방식이 더 옳은 것으로 간주됨
- OCP

<br/>
### 4. 장단점

- Advantages (+)
  - Observers로부터 Subject 분리. 느슨하게 연결되어 있어 구현. 변경. 테스트가 쉬움
  - Observer를 추가/삭제하기 쉬움. Subject의 책임은 Observer목록을 보관하고, 상태가 변경되면 이를 notify하는 것
- Disadvantages (–)
    - update가 복잡해질 수 있음. Subject의 변경으로 observers와 observers의 의존객체들에 연쇄적으로 발생할 수 있다.

<br/>
### 5. 예제

{% highlight java %}
1  public class MyApp {  
2      public static void main(String[] args) {  
3          Subject s1 = new Subject1();           
4          Observer o1 = new Observer1(s1);
5          Observer o2 = new Observer2(s1);
6             
7          System.out.println("Changing state of Subject1 ...");
8          s1.setState(100);
9      }
10  }

1  public abstract class Subject {  
2      private List<Observer> observers = new ArrayList<>();
3      public void attach(Observer o) {  
4          observers.add(o);
5      }
6
7     public void notifyObservers() {
8         for (Observer o : observers)
9             o.update();
10      }
11  }

1  public class Subject1 extends Subject {  
2      private int state = 0;
3      public int getState() {  
4          return state;
5      }  
6
7      void setState(int state) {  
8          this.state = state;
9          System.out.println(
10              "Subject1 : State changed to : " + state +
11              "\n           Notifying observers ...");
12          notifyObservers();
13      }
14  }

1  public abstract class Observer {  
2      public abstract void update();
3  }

1  public class Observer1 extends Observer {  
2      private int state;
3      private Subject1 subject;
4
5      public Observer1(Subject1 subject) {  
6          this.subject = subject;
7          subject.attach(this);  
8      }
9
10      public void update() {
11          this.state = subject.getState();
12          System.out.println(
13              "Observer1: State updated to : " + this.state);
14      }
15  }

1  public class Observer2 extends Observer {  
2      private int state;
3      private Subject1 subject;
4
5      public Observer2(Subject1 subject) {  
6          this.subject = subject;
7          subject.attach(this);  
8      }
9      public void update() {
10          this.state = subject.getState();
11          System.out.println(
12              "Observer2: State updated to : " + this.state);
13      }
14  }

{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern