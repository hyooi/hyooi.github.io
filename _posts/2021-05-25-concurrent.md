---
layout: post
title:  "자바 동시성"
---

## Thread클래스와 Executor인터페이스의 차이
- 스레드 생성은 자원을 많이 사용하는 연산
- Executor를 사용하면 스레드 풀을 사용할 수 있음
- 또한 Executor를 통해 스레드 관리가 가능(ex.Executors.newCachedThreadPool)
  - ExecutorService를 리턴하는데, 이를 통해 코드 실행 중단 및 결과반환이 가능  
<br/><br/>
    


## 동시성 코드 테스트
- CountDownLatch사용

{% highlight java %}
@Test
public void waitToComplete() {
  final ExecutorService executor = Executors.newCachedThreadPool();
  final CountDownLatch latch = new CountDownlatch(1);
  executor.execute(new FiniteThreadNamePrinterLatch(latch));
  latch.await(5, TimeUnit.SECONDS);
}

private static class FiniteThreadNamePrinterLatch implements Runnable {
  final CountDownLatch latch;
  
  private FiniteThreadNamePrinterLatch(final CountDownLatch latch) {
    this.latch = latch;
  }
  
  @Override
  public void run() {
    for(int i=0; i<25; i++) {
      System.out.println("Run from thread:" + Thread.currentThread().getName());
    }
  }
}
{% endhighlight %}
<br/>


- 동일 스레드에서 코드 실행하도록 적용 : CountDownLatch보다 오버헤드 낮음
{% highlight java %}
@Test
public void sameThread() {
  final Executor executor = new Executor() {
    
    @Override
    public void execute(final Runnable command) {
      command.run();
    }
  };

    System.out.println("Main thread:" + Thread.currentThread().getName());
    executor.execute(new FiniteThreadNamePrinter());
}

private static class FiniteThreadNamePrinterLatch implements Runnable {

  @Override
  public void run() {
    for(int i=0; i<25; i++) {
      System.out.println("Run from thread:" + Thread.currentThread().getName());
    }
  }
}
{% endhighlight %}
<br/>


## 스레드 간 공유 상태 관리
1. synchronized
  - 한번에 스레드 하나만 접근 가능하도록 함
  - 필요없는 경우엔 최대한 빨리 락 해제해야 함. 성능저하 우려

2. atomic클래스
  - atomic한 연산을 보장
  - incrementAndGet과 같은 메소드는 하나의 연산으로 실행되므로 값을 읽고 쓰는동안 변경되지 않는다는 것을 보장
<br/>
    

## 불변 객체를 사용하는 이유?
- 값이 변경되지 않으므로 스레드 간 락을 사용하지 않아도 됨


