---
layout: post
title:  "Garbage Collector 튜닝"
---

# 튜닝의 목적
- Old generation으로 넘어가는 객체의 수 최소화
  - old영역의 gc가 new영역의 gc보다 시간이 오래 걸리므로 객체의 수를 줄이면 full gc빈도 줄일 수 있음
- Full GC실행시간 최소화

[읽기](https://d2.naver.com/helloworld/37111)