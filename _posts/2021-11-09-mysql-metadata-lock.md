---
layout: post
title:  "Mysql ddl, dml 무한대기 현상"
categories:
- 트러블슈팅
- Database
tags:
- mysql
---

### 1. 개요
별 생각없이 DB테이블을 업데이트했는데, 간단한 컬럼 추가임에도 **무한 대기 현상**에 빠졌다.

다른 테이블은 문제가 없었는데, 특정 테이블은 DML, DDL 모두 작업이 종료되지 않았다.

<br/>

### 2. 원인을 찾자
#### 2.1. 메타데이터 락
mysql은 데이터베이스의 테이블, 스키마, 프로시저 등의 오브젝트에 대한 일관성을 보장하기 위해
[metadata lock](https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html)을 사용한다.

이에 따라 <ins>한 세션의 트랜잭션에서 사용 중인 테이블은, 
해당 트랜잭션이 종료되기 전까지 다른세션에서 ddl을 사용할 수 없다</ins>는 점이 중요한 포인트인데,
예기치않은 프로그램 종료에 의해 트랜잭션이 종료되지 못하면서 무한대기에 빠진 것이었다.

<br/>

#### 2.2. 프로세스 확인
먼저, 락이 잡힌 DB계정에 접속해 프로세스를 확인했더니 
심상치않은 **Waiting for table metadata lock** 상태의 프로세스들을 발견할 수 있었다.

```bash
$ SHOW PROCESSLIST;
```

|Id   | User | Host             | db   |Command|Time|State                          |Info     |
|---  |------|------------------|----- |-------|----|-------------------------------|---------|
|25800|root  |localhost:51282   | mydb |Sleep  |630 |Waiting for table metadata lock|select id, name|
|25803|root  |localhost:51312   | mydb |Sleep  |652 |Waiting for table metadata lock|alter table    |
|25807|root  |localhost:51352   | mydb |Sleep  |7743|                               |NULL           |

<br/>

무수히 많은 select문이 쌓여있었고, 그 중 alter문 하나가 끼어있었다.
다른 세션에서 특정 테이블을 select하는 도중 내가 alter를 실행했는데,
select하는 트랜잭션이 정상 종료되지 않은 것으로 보였다.


해당 락들은 Mysql자체적으로 타임아웃이 있어 자동 종료되기는 하나,
나처럼 별도 설정을 하지 않은 경우, 기본값이 무려 **1년**이다.

```bash
show variables like '%lock_wait_timeout%';
```

|Variable name| Value|
|---          |---   |
|lock_wait_timeout|86400|

> 해당 값을 변경해줄 수도 있지만, 이미 걸린 lock에는 적용되지 않는다.
> ex) set global lock_wait_timeout = 60;

<br/>

### 3. 해결
결국 서버에 접속해 해당 세션들을 직접 kill시켰다.

mysql을 도커로 기동해두었으므로 해당 도커 컨테이너에 접속해 mysqladmin으로 kill해주었다.
process list의 id로 kill하면 된다.
```bash
$ docker exec -it mysql /bin/bash
$ mysqladmin -uroot -padmin123! kill 25800
```
