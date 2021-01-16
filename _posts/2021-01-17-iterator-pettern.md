---
layout: post
title:  "디자인패턴: Iterator"
---

# Iterator pattern
> [ Behavioral pattern ]
> 
> 내부 구현 방법을 노출시키지 않으면서도 그 **집합체 안에 있는 모든 항목에 접근**할 수 있게 하는 방법을 제공한다.


<br/>
### 1. 문제

- 어떻게 내부 구현 방법을 노출하지 않으면서 집합체 안의 요소에 접근할 수 있는가?


<br/>
### 2. UML
![Iterator%2073e577e0ef8944489af5242505fc7d26/Untitled.png](/assets/images/designpattern/iterator.png)


### 3. 특징

1. 집합체에 있는 모든 데이터에 대해서 **반복작업을 하는 역할을 컬렉션에서 분리**시킬 수 있다.
2. 다양한 집합체 안의 데이터에 같은 인터페이스를 적용할 수 있기 때문에, 집합체에 있는 객체를 활용하는 코드를 만들 때 다형성을 활용할 수 있다. 
   (**iterator를 지원하기만 하면 어떤 컬렉션에 대해서도 사용 가능**)


<br/>
### 4. 장점
  - 집합체 인터페이스에서 **반복자(iterator) 인터페이스가 분리**된다. (단일책임원칙)
  - 런타임에 Iterator 인스턴스를 교체할 수 있다.


<br/>
### 5. 예제

{% highlight java %}
1  package com.sample.iterator.basic;
2  public class Client {  
3      public static void main(String[] args) {  
4          Aggregate<String> aggregate = new Aggregate1<String>(3);
5          aggregate.add(" ElementA ");
6          aggregate.add(" ElementB ");
7          aggregate.add(" ElementC ");

8         Iterator<String> iterator = aggregate.createIterator();
9         System.out.println("Traversing the aggregate front-to-back:");
10         while (iterator.hasNext()) {
11              System.out.println(iterator.next());
12         }
13      }
14  }

1  package com.sample.iterator.basic;
2  public interface Aggregate<E> {  
3      Iterator<E> createIterator();
4      boolean add(E element);
5  }

1  package com.sample.iterator.basic;
2  public interface Iterator<E> {  
3      E next();
4      boolean hasNext();
5  }

1  package com.sample.iterator.basic;
2  import java.util.NoSuchElementException;
3  public class Aggregate1<E> implements Aggregate<E> {
4      private Object[] elementData;
5      private int idx = 0;
6      private int size;

7      public Aggregate1(int size) {
8          if (size < 0)
9              throw new IllegalArgumentException("size: " + size);
10         this.size = size;
11         elementData = new Object[size];
12     }

13      public boolean add(E element) {
14          if (idx < size) {
15              elementData[idx++] = element;
16              return true;
17          }  else
18              return false;
19      }

20      public int getSize() {
21          return size;
22      }

23      public Iterator<E> createIterator() {
24          return new Iterator1<E>();
25      }

26      private class Iterator1<E> implements Iterator<E> {
27          private int cursor = 0;

28          public boolean hasNext() {
29              return cursor < size;
30          }

31          public E next() {  
32              if (cursor >= size)
33                  throw new NoSuchElementException();
34              return (E) elementData[cursor++];
35          }
36      }
37  }

{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.

  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern