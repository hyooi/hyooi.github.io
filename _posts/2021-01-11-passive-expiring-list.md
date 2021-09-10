---
layout: post
title:  "스스로 만료되는 리스트"
categories:
  - Java
tags:
  - 테스트케이스
---
간단하게 구현해본 **Passive expiring list** 

설정한 주기마다 스스로 객체를 만료시키는 list가 필요해 구현했다. (이후 Deque를 활용한 로직으로 변경함)

**Passive expiring map**의 경우 `apache commons4`에서 제공해주고 있는데, list는 구현이 워낙 간단해서 그런지 따로 제공되는 api가 없길래 직접 만들었다.

로직은 간단하다.
**상속보단 조합을 사용하라**던 조슈아 아저씨의 말에 따라 list객체는 외부에서 주입받고, 오브젝트 생성과 동시에 현재 시간을 저장하게된다.
(*이펙티브 자바 인용)

그리고 List인터페이스를 구현했으며, 무조건 모든 메소드 실행 이전에 만료된 객체를 삭제하는 removeAllExpired메소드를 타게 된다.

아래는 해당 클래스의 일부이다.

```java
@Getter
public class PassiveExpiringList<E> implements List<E> {
    private final List<E> list;
    private final long timeToLiveMillis;
    private Instant creation = Instant.now();

public PassiveExpiringList(List<E> list, long procIntvl) {
    this.list = list;
    this.timeToLiveMillis = procIntvl * 1000;
}

@Override
public int size() {
    removeAllExpired();
    return list.size();
}

@Override
public void clear() {
    list.clear();
    this.creation = Instant.now();
}

@Override
public E get(int index) {
    removeAllExpired();
    return list.get(index);
}

@Override
public E set(int index, E element) {
    removeAllExpired();
    return list.set(index, element);
}

protected void removeAllExpired() {
    if (isExpired()) {
      clear();
    }
}

protected boolean isExpired() {
    long timeElapsed = Duration.between(creation, Instant.now()).toMillis();
    return timeElapsed >= this.timeToLiveMillis;
}
}
```

모든 메소드에서 removeAllExpired를 타고 있으며, removeAllExpired에서는 생성 시점의 시간과 현재 시간을 비교해 만료되었을 시 list를 clear한다.
생략된 메소드 또한 같은 방식으로 list의 메소드를 타기 전에 removeAllExpiried메소드를 태워주면 된다.

간단한 로직이긴하지만.. 그럼에도 불구하고 테스트케이스 또한 작성해보았음..

```java
class PassiveExpiringListTest {

	@Test
	@SneakyThrows
	@DisplayName("list초기화 테스트")
	void test_expired_with_constructor1() {
		val tempList = new ArrayList<>();
		tempList.add("temp1");

		val list = new PassiveExpiringList<>(tempList, 3);

		assertThat(list.size()).isEqualTo(1);

		Thread.sleep(3 * 1000);
		assertThat(list.size()).isZero();
	}

	@Test
	@SneakyThrows
	@DisplayName("add후 정상 초기화 테스트")
	void test_expired_after_add1() {
		val tempList = new ArrayList<>();
		tempList.add("temp1");

		val list = new PassiveExpiringList<>(tempList, 3);

		list.add("temp2");
		assertThat(list.size()).isEqualTo(2);

		Thread.sleep(3 * 1000);
		assertThat(list.size()).isZero();
	}
}
```

좀 더 완성도 높은 테스트를 위해서는 테스트케이스가 더 다양해야겠지만...ㅎㅎ

arrayList를 통해 3초간 유지되는 PassiveExpiringList를 선언한다. 그리고 3초가 흐른 후 데이터가 정상적으로 초기화되는지를 확인하는 테스트케이스이다.


성공! :)
