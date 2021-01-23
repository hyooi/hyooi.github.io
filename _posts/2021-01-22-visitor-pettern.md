---
layout: post
title:  "디자인패턴: Visitor"
---

# Visitor pattern
> [ Behavioral pattern ]
> 
> 객체의 요소에 대해 수행할 작업을 나타낸다.
> 
> Visitor는 작업이 수행될 클래스를 변경하지 않고도 새로운 오퍼레이션을 정의할 수 있도록 한다. 


<br/>
### 1. UML

![Visitor%2046e2c424c0b0467ab98b97edbfee49de/untitled](/assets/images/designpattern/visitor.png)

### 2. 특징

- 구조 자체를 변경하지 않고도 새로운 기능을 추가할 수 있음
- visitor에서 수행하는 기능과 관련된 코드를 한 곳에 집중시켜 놓을 수 있음
- 클래스의 캡슐화가 깨지므로 캡슐화가 중요하지 않은 경우 사용함
- OCP 적용 방법의 하나

<br/>
### 3. 더블 디스패치

- visitor pattern은 double dispatch를 활용하는 패턴.
- Client에서 element.accept할때, 각 element의 accept메소드에서 visitor.visitorElement(this); 할 때 동적 디스패치가 두번 발생
  - 정적 dispatch : 구현class를 타입으로 한 메소드 호출. 호출할 메소드가 컴파일 시 지정된다.
  - 동적 dispatch : 인터페이스를 타입으로 한 메소드 호출. 호출할 메소드가 런타임 시 지정된다.

<br/>
### 4. 예제

{% highlight java %}
public class Client {
  public static void main(String[] args) {
    List<Element> elements = new ArrayList<>();
    elements.add(new ElementA());
    elements.add(new ElementB());

    Visitor visitor = new Visitor1();

    for (Element element : elements) {
      element.accept(visitor);
    }
  }
}

public abstract class Element {
  public abstract void accept(Visitor visitor);
}

public class ElementA extends Element {
  @Override
  public void accept(Visitor visitor) {
    visitor.visitElementA(this);
  }

  public String operationA() {
    return "Hello World from ElementA.";
  }
}

public class ElementB extends Element {
  @Override
  public void accept(Visitor visitor) {
    visitor.visitElementB(this);
  }

  public String operationB() {
    return "Hello World from ElementB.";
  }
}

public abstract class Visitor {
  public abstract void visitElementA(ElementA e);
  public abstract void visitElementB(ElementB e);
}

public class Visitor1 extends Visitor {
  @Override
  public void visitElementA(ElementA element) {
    System.out.println("Visitor1: Visiting (doing something on) ElementA." + element.operationA());
  }

  @Override
  public void visitElementB(ElementB element) {
    System.out.println("Visitor1: Visiting (doing something on) ElementB." + element.operationB());
  }
}

{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern