---
layout: post
title:  "컨테이너 오케스트레이션"
published: false
---

# 컨테이너 오케스트레이션
- 다수의 컨테이너를 관리해야 하는 분산환경에서는 수동으로 컨테이너 관리 불가
- 따라서 다수의 컨테이너를 쉽게, 자동으로 관리하는 기술을 **컨테이너 오케스트레이션**이라 함
- 쿠버네티스가 가장 대표적이며, 이외에 docker swarm, apache mesos/marathon 존재


## 컨테이너 오케스트레이션의 특징
1. 다수의 서버를 하나의 클러스터처럼 사용
2. 여러개 서버에 컨테이너 배포
3. 서비스 디스커버리로 서비스들을 연결
4. 부하 발생 시 자동 scale out
5. 장애 발생 시 기존 컨테이너 kill후 신규 컨테이너 재생성
6. 컨테이너 health check
7. 컨테이너 간 storage 및 network 관리


## Kubernates
- google에서 개발했으며, 현재 CNCF에 이관(영리목적X)
- 사실상 컨테이너 오케스트레이션의 표준


## Kubernates 특징
### Immutable Infrastructure
- 한번 구축한 인프라는 수정 필요 시 파기 후 재생성
- ex. java version을 올려야 하는 경우, 업데이트된 이미지를 만들어 신규 컨테이너 배포


### Declarative configuration
- 원하는 최종 상태를 명시함
- EX. `컨테이너를 2개 추가하라`가 아닌, `컨테이너 5개로`라는 상태를 명시함


## Self healing
- 장애 발생 시 사람의 개입을 최소화하고 자동 복구
- kubernates는 state를 실시간 감시하여 복구함
