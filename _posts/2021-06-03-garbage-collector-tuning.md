---
layout: post
title:  "Garbage Collector 튜닝"
---

# 튜닝의 목적
- Old generation으로 넘어가는 객체의 수 최소화
  - old영역의 gc가 new영역의 gc보다 시간이 오래 걸리므로 객체의 수를 줄이면 full gc빈도 줄일 수 있음
- Full GC실행시간 최소화
  - full gc실행시간을 줄이기 위해 old영역을 줄이면, full gc횟수가 늘어남.
  반대로 old의 크기를 늘리면 full gc횟수는 줄어들지만 실행시간이 늘어나므로 적절한 설정 필요
    
# GC튜닝절차
- GC 상황 모니터링 후, 수행시간이 1~3초가 넘어가면 튜닝 진행해야 함
- 혹은 메모리가 1,2GB로 지정했는데 OOM이 발생한다면 힙덤프로 원인 파악 후 문제점 제거
- 24시간 이상 데이터를 수집해 GC방식, 메모리 크기를 변경해나가면서 최적의 옵션을 찾아야함

[참조](https://d2.naver.com/helloworld/37111)