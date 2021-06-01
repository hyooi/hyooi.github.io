---
layout: post
title:  "JVM Garbage Collectors"
---

# JVM 가비지 컬렉션
## 가비지 컬렉션 단계
- Mark : 사용 중인 메모리와 그렇지 않은 메모리를 식별
- Sweep : mark단계에서 식별된 오브젝트를 삭제하는 단계


## 장점
- 수동으로 메모리 할당/해제하지 않아도 됨
- Dangling pointer를 처리하지 않아도 됨
  (*dangling pointer: 객체에 대한 참조가 수정 없이 삭제되거나 할당 해제되어 
  포인터가 계속 할당 해제된 메모리를 가리킴)
- 자동 memory leak관리


## 단점
- jvm이 object참조의 생성, 삭제를 트래킹한 이후로, 기존 어플리케이션보다 더 많은 cpu를 사용하고,
더 많은 메모리를 필요로 하게됨
- 개발자는 필요하지 않은 object를 해제하는데 사용되는 cpu시간을 제어할 수 없음
- 일부 GC구현은 어플리케이션을 예상치 못하게 중단시킬 수 있음
- 자동화된 메모리 관리는 적절한 수동 메모리 할당/해제보단 효율적이지 않음


## GC구현
### Serial Garbage Collector
- 단일 스레드에서 작동하는 가장 간단한 구현
- 실행 시 전체 어플리케이션의 스레드를 프리징하므로, 멀티 스레드 환경에서는 좋은 방법이 아님
- 적은 중단 시간을 필요로 하지 않는 클라이언트 스타일의 머신에서 선택됨
```
java -XX:+UseSerialGC -jar Application.java
```


### Paralled Garbage Collector
- JVM의 기본 GC. 다중 스레드를 사용해 힙 메모리를 관리
- GC수행하는 동안 다른 어플리케이션 스레드도 프리징됨
- 최대 가비지 컬렉션 스레드와 일시 중지시간, 처리량 및 힙 크기 지정 가능
```
java -XX:+UseParallelGC -jar Application.java
-XX : ParallelGCThreads = <N> : 가비지 컬렉션 스레드 수
-XX : MaxGCPauseMillis = <N> : 최대 일시 중지시간
-XX : GCTimeRatio = <N> : 최대 처리량 목표(가비지컬렉션 소요 시간)
-Xmx <N> : 최대 힙 메모리 양
```

### CMS Garbage Collector
- Concurrent mark sweep이라는 의미. 
- 다수의 가비지 컬렉션 스레드를 사용
- 짧은 가비지 컬렉션 중단을 위해 디자인되었고, 어플리케이션을 사용하는 동안 가비지 컬렉터와
  프로세서 리소스가 공유됨
- 응답이 느리지만 가비지 컬렉션을 위해 응답을 중지하지 않음
- GC가 concurrent하게 동작하므로, System.gc와 같은 명시적인 가비지 컬렉션 요청을 하는 경우
  Concurrent Mode Failure / Interruption발생
- 총 시간의 98% 이상이 CMS가비지 컬렉션에 사용되고, 힙의 2%미만으로 복구되면 OOM발생
  (-XX : -UseGCOverheadLimit으로 비활성화 가능)
- Java9부터 더이상 사용X, Java14부터 미지원
```aidl
java -XX:+UseParNewGC -jar Application.java
```

### G1 Garbage Collector
- Garbage First Garbage Collector
- 대용량 메모리 공간이 있는 다중 프로세서 시스템에서 실행되는 어플리케이션을 위해 설계됨
- Java7부터 지원하며, 효율성이 높아 CMS 가비지 컬렉터를 대체
```aidl
java -XX:+UseG1GC -jar Application.java
```

### Z Garbage Collector