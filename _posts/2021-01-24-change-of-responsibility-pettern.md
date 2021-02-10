---
layout: post
title:  "디자인패턴: Chain Of Responsibility"
---

# Chain Of Responsibility
> [ Behavioral pattern ]
> 
> 명령 객체와 일련의 처리 객체를 포함하는 디자인 패턴
> if의 객체 지향 버전이라고도 함
> 
> ex. 윈도우 시스템의 ui event처리(마우스 클릭, 키보드 이벤트 등)


<br/>
### 1. 특징
- 수신자/송신자 독립적임.
- 송신자는 특정 수신자에 연결되지 않음
- 둘 이상의 수신자가 요청을 처리할 수 있음
- chain 내부의 객체를 바꾸거나 순서를 바꿔가며 사용할 수 있음
- chain의 수신자를 모두 거쳤음에도 아무런 처리를 하지 않을 수 있음

<br/>
### 2. UML

![Chain%20of%20Responsibility%204afa86429e434294a8ee51587db2c9a2/untitled](/assets/images/designpattern/chain.png)

  
### 3. 예제

{% highlight java %}
public class Sender {
  public static void main(String[] args) { 
    Handler handler = new Receiver1(new Receiver2()); 
    handler.handleRequest(); 
  }
}

public abstract class Handler { 
  private Handler successor;

  public Handler() {} 
  
  public Handler(Handler successor) {
    this.successor = successor; 
  } 
  
  public void handleRequest() {
    if (successor != null) {
      successor.handleRequest(); 
    }
  }
  
  public boolean canHandleRequest() {
    return false; 
  } 
}

public class Receiver1 extends Handler {
  public Receiver1(Handler successor) {
    super(successor); 
  } 

  @Override 
  public void handleRequest() {
    if (canHandleRequest()) {
      System.out.println("Receiver1: Handling the request ...");
    } else { 
      System.out.println("Receiver1: Passing the request along the chain ...");
      super.handleRequest(); 
    } 
  } 
}

public class Receiver2 extends Handler {
  @Override 
  public void handleRequest() {
    System.out.println("Receiver2: Handling the request"); 
  }
}

{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern