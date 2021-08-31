---
layout: post
title:  "마이크로서비스 지원역량"
---

- MSA환경에서 필요한 역량들은 현재 이미 spring cloud에서 오픈소스로 지원하고 있음
- 보통 어노테이션만으로도 구성이 가능
- 넷플릭스에서 개발한 오픈소스인 netflix oss와 spring cloud의 오픈소스를 일반적으로 사용함


# Service Discovery
- MSA를 도입하면 서비스 갯수 및 인스턴스 갯수가 급증해 전체 토폴로지가 복잡해짐
- 서비스간 호출이 복잡하게 엮여있게 됨
- 서비스 디스커버리는 중앙에서 시스템의 모든 서비스 정보를 관리
- 인스턴스 생성 시마다 중앙 에이전트에 자신의 정보를 등록하는 방식
- 서비스 간에 물리적인 주소를 사용하지 않아 유연한 관리 가능
- 로드밸런서를 별도로 관리하지 않아도되는 장점이 있으나, 통합관리가 어렵다는 단점이 존재
- ex. Eureka(클라이언트 사이드 서비스 디스커버리)

## Eureka
- 유레카 서버와 클라이언트로 나눠지며, 유레카 서버가 서비스 디스커버리 역할을 함
- 클라이언트는 기동 시 유레카 서버에 정보를 등록
- 주기적으로 heartbeat를 보내며, 장애 발생 시 해당 서비스를 레지스트리에서 삭제


### eureka 특징
- 고가용성: active/active구조로 고가용성 만족
- 피어투피어 구조: 서비스 디스커버리 클러스터의 모든 노드들이 상태를 공유함
- 장애 내성: 사람의 개입없이 인스턴스의 비정상 상태를 감지하고 제외시킴
- 회복성: 서비스 디스커버리의 정보를 클라이언트는 로컬에 캐시
서비스 디스커버리에 장애가 발생해도 로컬 캐시로 접근 가능(ribbon이 해당 역할을 함)
- 부하분산: 클라이언트단 로드밸런싱(ribbon도 해당 기능 수행 가능. 기본적으로 round robbin)


# Config server
- MSA를 도입하면 서비스가 다양해져 설정 관리가 복잡해짐
- 다수의 설정파일 중 잘못되는 경우 장애가 발생하게 되므로, 설정정보를 파일에서 분리함
- 중앙 config server에서 설정 관리(ex. git, db, 파일시스템 등에 저장)
- 기본적으로 app기동 시 config정보를 fetch해오며, 변경된 정보는 서비스 재배포없이 반영 가능
- EX. spring cloud config server, kubernates configmap, consul, etcd, zookeeper


## spring cloud config
- rest api로 호출하면 json으로 응답
- spring boot app에서 사용하는 `application.yml`등의 파일을 git이나 db에 저장하고,
    이를 config server를 통해 로드해올 수 있음


# Service gateway(= api gateway)
- 인증, 인가, 로깅, 필터링 등의 공통 처리 수행
- 각각의 서비스들에 대한 공통 로직을 처리함
- 서비스 마다 언어, 프레임워크가 상이한 msa환경에서 공통 로직 적용하기 용이함
- ex. zuul, spring cloud gateway, kong api gateway, amazone api gateway, gcp cloud endpoints


## netflix zuul
- eureka, ribbon, hystrix와 통합 가능
- eureka를 통해 별도 설정없이도 서비스 디스커버리에서 서비스를 자동 매핑함
- ribbon을 이용해 로드밸런싱되며,
- zuul의 모든 호출은 hystrix를 거쳐 호출되게 됨
- zuul filter를 통해 java로 서비스 호출 전후 실행할 코드를 작성할 수 있음


# SW Defined load balancer
- MSA에서는 인스턴스의 제거, 생성이 빈번함
- 클라이언트에서 Load balancer처리를 함
- 서비스 레지스트리에서 다른 서비스의 정보를 가지고 와 자체적으로 로드밸런싱 처리
- ex. ribbon


# Circuit breaker
- 한 서비스의 장애가 다른 서비스의 장애로 전파되지 않도록 함
- 서비스 사이에 위치해, 특정 서비스에 장애가 발생했다고 판단되면 circuit차단
- circuit차단 시 예외발생 대신 fallback패턴으로 다른 행동을 정의할 수도 있음
- ex. 다른 db호출, retry queue에 쌓거나 하드코딩 결과 응답
- 실패를 빠르게 인지해 시스템 전체에 영향이 가지 않도록 방지하고, 호출 실패 시 대체 경로를 사용하게 함
- ex. hystrix


## hystrix
- circuit breaker
- fallback
- thread isolation: 실제 호출을 새로운 thread가 intercept하여 호출
- timeout: 동일한 정책의 timeout적용 가능 ex. socket, connection, read, jdbc socket..


# Distributed tracing
- MSA는 하나의 api호출이 다양한 서비스를 호출하므로 에러추적이 어려움
- 서비스 간 모든 호출에 추적ID 삽입
- 해당 추적ID를 통해 트랜잭션 파악 가능
- EX. zipkin, jaeger


# Data lake
- MSA는 다양한 DB를 사용함
- 대용량 비정형 데이터를 저장할 수 있음


# Messaging
- MSA는 메시징을 이용한 서비스 간 협력 설계 방식을 권고
- 서비스 간 결합도를 낮출 수 있기 때문!
- spring cloud stream을 통해 다양한 messaging시스템과 event driven architecture구현 가능
- EX. RabbitMQ, ActiveMQ, kafka, AWS kinesis

# Devops
- CI, CD, 자동화된 QA 등이 필요


# 컨테이너 레지스트리
- 컨테이너의 핵심인 이미지의 관리가 필요
- 코드 형상관리와 유사함
- ex. docker hub


# 문서화
- 인터페이스를 통해 서비스 간 통신하므로 문서화 필수
- 웹으로 쉽게 열람 가능해야 함
- EX. Spring REST Docs, swagger
