---
layout: post
title:  "디자인패턴: Memento"
---

# Memento pattern
> [ Behavioral pattern ]
> 
> 캡슐화를 위반하지 않고 객체의 내부 상태를 저장한다. 또, 저장한 상태로 복구할 수 있도록 한다.
> 
> (ex.작업취소)


<br/>
### 1. 문제
- 캡슐화를 위반하지 않고 객체의 내부 상태를 저장하고, 후에 저장한 상태로 복원할 수 있는가?

<br/>
### 2. UML

![Memento%20f88cd887fb6b4effb93b5719111515e7/Untitled.png](/assets/images/designpattern/memento.png)

### 3. 특징

- 시스템에서 핵심적인 기능을 담당하는 객체의 중요한 상태 저장
- 핵심적인 객체의 캡슐화 유지
- 내부 상태를 **저장/복원**하는 작업을 정의. 메멘토를 만든 Originator만이 그 메멘토에 접근할 수 있음
- Caretaker는 Originator에 의해 반환된 메멘토를 보관하고, originator에 전달하여 이전 상태로 복원함. 메멘토에 접근은 할 수 없다.


<br/>
### 4. 장단점

- Advantages (+)
  - 저장된 상태를 originator와는 별도의 객체(memento)에 보관하므로 안전
  - 핵심 객체의 상태를 캡슐화된 상태로 유지
  - 복구 기능 구현이 쉬움
- Disadvantages (–)
  - 상태를 저장하고 복구하는 데 시간이 오래 걸릴 수 있음.
  - 메모리 사용 및 시스템 성능에 영향을 미칠 가능성
  
### 4. 예제

{% highlight java %}
class Caretaker {

    public static void main(String[] args) {
        Originator originator = new Originator();
        Originator.Memento memento; // Memento is inner class of Originator
        
        List<Memento> mementos = new ArrayList<Memento>();
        originator.setState("A");
        memento = originator.createMemento();
        mementos.add(memento); // adding to list
        System.out.println("(1) Saving current state ...... : "
            + originator.getState());
    
        originator.setState("B");
        memento = originator.createMemento();
        mementos.add(memento); // adding to list
        System.out.println("(2) Saving current state ...... : "
            + originator.getState());
    
        memento = mementos.get(0); // getting previous (first) memento from the list
        originator.restore(memento);
        System.out.println("(3) Restoring to previous state : "
            + originator.getState());
    }
}

class Originator {

    private String state;
    
    Memento createMemento() {
        Memento memento = new Memento();
        memento.setState(state);
        return memento;
    }
    
    void restore(Memento memento) {
        state = memento.getState();
    }
    
    String getState() {
        return state;
    }
    
    void setState(String state) {
        this.state = state;
    }
    
    class Memento {
    
        private String state;
    
        private String getState() {
          return state;
        }
    
        private void setState(String state) {
          this.state = state;
        }
    }
}

{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern