---
layout: post
title:  "디자인패턴: Adapter"
---

# Adapter pattern
> [ Structural pattern ]
> 
> 한 클래스의 인터페이스를 클라이언트가 사용하고자 하는 **다른 인터페이스로 변환**한다. 
> 호환이 되지 않는 인터페이스를 사용하는 오브젝트들을 함께 작동할 수 있게 한다.


<br/>
### 1. UML

![Adapter%20d39796b404aa4443b5a68602fcd615f6/untitled](/assets/images/designpattern/adapter.png)


### 2. 특징

- 클라이언트는 타겟 인터페이스를 사용하여 메소드를 호출함으로써 어댑터에 요청한다.
- 어댑터에서는 어댑티 인터페이스를 사용하여 그 요청을 어댑티에 대한 메소드 호출로 변환한다.
- 클라이언트에서는 호출 결과를 받긴 하나, 중간에 어떤 어댑터가 있는지는 알 수 없다.
- 어댑터에 새로운 기능을 구현하여 **필요한 기능을 동적으로 추가**할 수 있다.


<br/>
### 3. 장단점

- Advantages (+)
  - 인터페이스가 호환되지 않아 사용할 수 없었던 오브젝트의 기능을 어댑터를 활용하여 재사용할 수 있다.
  - 클래스 어댑터는 타겟 인터페이스와 어댑티 클래스를 상속하여 구현하는데, 이 경우 컴파일 시점에 어댑티가 결정되므로 다른 어댑티를 적용할 수 없다. 반면에 오브젝트 어댑터는 타겟 인터페이스만 상속하고 런타임 시 어댑티 객체에 위임하므로 더 유연하다.
  - 단일책임원칙 및 OCP적용에 유용하다.
  
- Disadvantages (–)
  - 새로운 인터페이스 및 클래스를 도입해야하기 때문에 코드의 전체적인 복잡성이 증가한다.

<br/>

### 4. 예제

{% highlight java %}
class Client {
  public static void main(String[] args) {
    Target objectAdapter = new ObjectAdapter(new Adaptee());
    System.out.println("Object Adapter: " + objectAdapter.operation());
  }
}

interface Target {
  String operation();
}

class ObjectAdapter implements Target {
  private Adaptee adaptee;
  ObjectAdapter(Adaptee adaptee) {
    this.adaptee = adaptee;
  }

  @Override
  public String operation() {
    return adaptee.specificOperation();
  }
}

class Adaptee {
  String specificOperation() {
    return "Hello World from Adaptee";
  }
}


{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern