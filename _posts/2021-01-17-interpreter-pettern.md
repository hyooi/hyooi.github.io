---
layout: post
title:  "디자인패턴: Interpreter"
published: false
---

# Interpreter pattern
> [ Behavioral pattern ]
> 
> **문법에 대한 표현**을 정의한다. 또 문장을 해석하기 위해 정의한 표현에 기반하여 인터프리터를 정의한다.


![Interpreter%20095c16183fd64e68896f2d3e18f12321.png](/assets/images/designpattern/Interpreter2.png)

### 1. 문제

1. 특정 유형의 문제가 자주 발생하는 경우, 문제의 예를 **단순한 문장으로 표현**하는 것이 나을 수 있다. 그러면 해당 문장들을 해석하여 문제를 해결하는 인터프리터를 만들 수 있다.
2. 인터프리터가 없는 경우, Expression은 클래스 내에서 직접 연결된다. 따라서  컴파일 및 런타임에 새로운 expression을 지정하거나, 기존 expression을 변경하기가 어렵다.


<br/>
### 2. 솔루션

1. Expression은 별도의 AST(Abstract Syntax Tree)로 표현된다. (하단 AST참조)
2. 런타임에 새로운 AST를 생성하거나, 기존 Expression을 동적으로 변경할 수 있다.


<br/>
### 3. UML

![Interpreter%20095c16183fd64e68896f2d3e18f12321/Untitled.png](/assets/images/designpattern/interpreter-uml.png)


### 4. 특징

1. Terminal Expression : Expression의 해석을 구현. Child expressions를 가지지 않는다. (ex. variable, x, y, z)
2. NonTerminal Expression : Child Expressions에 해석을 요청함 (ex. and, or, not)


<br/>
### 5. 장단점

- Advantages (+)
    - 각 문법 규칙을 클래스로 표현하므로 언어를 쉽게 구현할 수 있음
    - 언어의 변경이나 확장이 쉬움
    - Visitor패턴을 활용하여 기존 Expression구조의 변경 없이 새로운 종류의 해석을 정의할 수 있음

- Disadvantages (–)
    - 문법 규칙의 갯수가 많아지면 복잡해짐


<br/>
### 6. 예제

{% highlight java %}
6  public class Client {  
8      public static void main(String[] args) throws Exception {  
12          List<Product> products = new ArrayList<Product>(); 
13          products.add(new SalesProduct("PC1", "PC", "Product PC 1000", 1000)); 
14          products.add(new SalesProduct("PC2", "PC", "Product PC 2000", 2000)); 
15          products.add(new SalesProduct("PC3", "PC", "Product PC 3000", 3000)); 
17          products.add(new SalesProduct("TV1", "TV", "Product TV 1000", 1000)); 
18          products.add(new SalesProduct("TV2", "TV", "Product TV 2000", 2000)); 
19          products.add(new SalesProduct("TV3", "TV", "Product TV 3000", 3000)); 

25          VarExpr x = new VarExpr("X"); 
26          VarExpr y = new VarExpr("Y"); 
27          VarExpr z = new VarExpr("Z"); 

29          Expression andExpr1 = new AndExpr(); 
30          andExpr1.add(x); 
31          andExpr1.add(y); 

33          Expression andExpr2 = new AndExpr(); 
34          andExpr2.add(y); 
35          andExpr2.add(z); 

36          Expression notExpr = new NotExpr(); 
37          notExpr.add(x); 
38          andExpr2.add(notExpr); 

40          Expression expression = new OrExpr(); 
41          expression.add(andExpr1); 
42          expression.add(andExpr2); 

48          Context context = new Context(); 
49          for (Product p : products) { 
54              context.setVarExpr(x, p.getGroup() == "PC" ? true : false); 
55              context.setVarExpr(y, p.getPrice() > 1000 ? true : false); 
56              context.setVarExpr(z, p.getDescription().contains("TV") ? true : false); 

58              if (expression.evaluate(context)) 
59                  System.out.println("Product found: " + p.getDescription()); 
60          } 
61      } 
62  } 
Product found: Product PC 2000 
Product found: Product PC 3000 
Product found: Product TV 2000 
Product found: Product TV 3000

4  public class Context {  
6      Map<String, Boolean> varExprMap = new HashMap<String, Boolean>(); 

8      public void setVarExpr(VarExpr v, boolean b) {  
9          varExprMap.put(v.getName(), b); 
10     } 

11     public boolean getVarExpr(String name) { 
12         return varExprMap.get(name); 
13     } 
14  }  

4  public abstract class Expression {  
5      public abstract boolean evaluate(Context context); 

8      public boolean add(Expression e) {
9          return false; 
10     } 
11     
		   public Iterator<Expression> iterator() { 
12         return Collections.emptyIterator();
13     } 
14  }  

2  public class VarExpr extends Expression { 
3      private String name; 

4      public VarExpr(String name) {  
5          this.name = name; 
6      }  

8      public boolean evaluate(Context context) {  
9          return context.getVarExpr(name); 
10     } 

11     public String getName() { 
12          return name; 
13     } 
14  }  

5  public abstract class NonTerminalExpression extends Expression {  
6      private List<Expression> expressions = new ArrayList<Expression>(); 

8      public abstract boolean evaluate(Context context); 

10      @Override 
11      public boolean add(Expression e) { 
12          return expressions.add(e); 
13      } 

14      @Override 
15      public Iterator<Expression> iterator() { 
16          return expressions.iterator(); 
17      } 
18  }  

3  public class AndExpr extends NonTerminalExpression {
4      public boolean evaluate(Context context) {  
5          Iterator<Expression> it = iterator(); 
6          while (it.hasNext()) {  
7              if (!it.next().evaluate(context)) 
8                  return false; 
9          } 

10         return true; 
11      } 
12  }  

3  public class OrExpr extends NonTerminalExpression { 
4      public boolean evaluate(Context context) {  
5          Iterator<Expression> it = iterator(); 
6          while (it.hasNext()) {  
7              if (it.next().evaluate(context)) 
8                  return true; 
9          } 

10         return false; 
11      } 
12  }  

3  public class NotExpr extends NonTerminalExpression {  
4      public boolean evaluate(Context context) {  
5          Iterator<Expression> it = iterator(); 
6          while (it.hasNext()) {  
7              if (it.next().evaluate(context)) 
8                  return false; 
9          } 

10         return true; 
11      } 
12  } 

2  public interface Product {  
3      void operation(); 
4      String getId(); 
5      String getGroup(); 
6      String getDescription(); 
7      long getPrice(); 
8  }  

2  public class SalesProduct implements Product {  
3      private String id; 
4      private String group; 
5      private String description; 
6      private long price;

8      public SalesProduct(String id, String group, String description, long price) {  
9          this.id = id; 
10          this.group = group; 
11          this.description = description; 
12          this.price = price; 
13      } 
14      public void operation() { 
15          System.out.println("SalesProduct: Performing an operation ..."); 
16      } 
17      public String getId() { 
18          return id; 
19      } 
20      public String getGroup() { 
21          return group; 
22      } 
23      public String getDescription() { 
24          return description; 
25      } 
26      public long getPrice() { 
27          return price; 
28      } 
29  }
{% endhighlight %}

* [Github]에 전체 디자인패턴 예제 소스가 업로드되어 있습니다.
* 포스팅의 내용은 GOF의 디자인패턴 및 헤드퍼스트 디자인패턴, 그리고 위키백과 등을 참조했습니다.
  
  [Github]: https://github.com/hyooi/TIL/tree/master/til.designpattern
