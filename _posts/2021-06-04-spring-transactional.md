---
layout: post
title:  "Spring transactional"
---

# Spring Transaction
- 스프링aop를 이용해 트랜잭션을 적용
- PlatformTransactionManager을 사용해 트랜잭션을 처리


# Read only
- true: 트랜잭션을 읽기 전용으로 설정. 쓰기 작업이 일어나는 경우 예외 발생


# timeout
- 지정한 시간 내에 메소드 수행이 되지 않으면 rollback. (default: -1)


# rollback
- runtime exception발생 시 롤백
- runtime exception이 없었거나, checked exception발생 시 커밋
- checked exception이 커밋인 경우는 리턴 값을 대신해 비즈니스 의미를 담은 경우가 많기 때문
- date access단의 exception은 runtime exception로 던져지므로 해당 exception만 롤백 대상
- rollback-for, rollbackForClassName 등으로 직접 예외를 지정하는 경우 checked exception도 롤백 가능


# Propagation
: 트랜잭션 동작 도중 다른 트랜잭션을 호출한 경우 사용

- required: 기본. 부모 트랜잭션 내에서 실행하며 부모 트랜잭션이 없으면 새로운 트랜잭션 생성
- supports: 이미 시작된 트랜잭션이 있으면 참여하고, 없으면 트랜잭션 없이 진행
- required_new: 부모 트랜잭션을 무시하고 신규 트랜잭션 생성
- mandatory: 부모 트랜잭션이 있으면 참여하며 없으면 예외 발생. 독립적으로 트랜잭션을 진행하면 안되는 경우 사용
- requires_new: 항상 신규 트랜잭션 시작. 이미 진행중인 트랜잭션이 있으면 보류시킴
- not_supported: 트랜잭션 미사용. 트랜잭션이 있으면 보류시킴
- never: 트랜잭션 미사용 강제. 트랜잭션이 있으면 예외 발생
- nested
  - 트랜잭션이 있으면 중첩 트랜잭션 시작(신규 트랜잭션 생성)
  - 부모 트랜잭션의 커밋과 롤백에는 영향을 받으나, 자신의 커밋과 롤백은 부모 트랜잭션에 영향X
  - EX. 작업 중 로그를 저장하는 경우, 로그TX를 nested로 설정


# isolation
- default: db의 isolation level을 따름
- read_uncommitted: 커밋되지 않은 데이터 read가능. 커밋되지 않은 데이터를 읽어 dirty read발생
- read_committed
  - 커밋된 데이터만 read가능
  - dirth read는 방지 가능하나 동일한 트랜잭션 내에서 같은 데이터를 읽을 때 서로 다른 결과를 받을 수 있음
- repeatable_read
  - 트랜잭션 내에서 shared lock이 걸리므로 데이터를 두번 쿼리해도 일관성있는 결과 리턴
  - 트랜잭션이 완료될 때까지 select가 사용하는 모든 데이터에 lock. 다른 사용자는 해당 데이터 수정 불가
- serializable
  - 데이터 일관성을 위해 트랜잭션이 완료될 때까지 select데이터에 shared lock
  - 다른 사용자는 그 영역의 데이터에 수정 및 입력, 읽기 불가
  - 성능 저하 우려