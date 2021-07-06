---
layout: post
title:  "SQL튜닝"
---

# 1. 실행계획?
- 사용자가 sql을 실행하여 데이터를 추출할 때, 옵티마이저가 수립하는 절차
- 실제 실행하지 않는 경우 데이터를 처리하지 않으므로 부하를 주지 않으나, 소요시간 혹은 IO측정 불가
<br><br>


  
## 2. 실행계획 확인 방법
### 2-1. Autotrace
```sql
/* AUTOTRACE */
SET AUTOTRACE ON EXPLAIN; --실제 실행O, 결과 및 실행계획 확인
SET AUTOTRACE ON STATISTICS; -- 실제 실행O, 결과 및 통계(IO정보) 확인
SET AUTOTRACE TRACEONLY; --실제 실행O, 실행계획, 통계 확인
SET AUTOTRACE TRACEONLY EXPLAIN; --실제실행X, 실행계획 확인
SET AUTOTRACE OFF; -- 실행계획 OFF
```

### 2-2. Explain plan
```sql
/* EXPLAIN PLAN */
EXPLAIN PLAN SET STATEMENT_ID = 'A1'
INTO PLAN_TABLE
FOR select * from TEST_BLOB_CLOB;

SELECT OPERATION,
       OPTIONS,
       OBJECT_NAME,
       OBJECT_TYPE,
       ID,
       PARENT_ID,
       POSITION,
       COST,
       CARDINALITY,
       CPU_COST,
       IO_COST
FROM PLAN_TABLE
WHERE STATEMENT_ID = 'A1'
```
<br>

## 3. 실행계획 예
```
SQL> SELECT * FROM SOURCE A, TARGET B WHERE A.ID = B.ID;

----------------------------------------------------------------------------------------------
| Id  | Operation                    | Name          | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT             |               |     1 |   221 |     4   (0)| 00:00:01 |
|   1 |  NESTED LOOPS                |               |       |       |            |          |
|   2 |   NESTED LOOPS               |               |     1 |   221 |     4   (0)| 00:00:01 |
|   3 |    TABLE ACCESS FULL         | SOURCE        |     1 |   115 |     3   (0)| 00:00:01 |
|*  4 |    INDEX UNIQUE SCAN         | SYS_C00326745 |     1 |       |     0   (0)| 00:00:01 |
|   5 |   TABLE ACCESS BY INDEX ROWID| TARGET        |     1 |   106 |     1   (0)| 00:00:01 |
----------------------------------------------------------------------------------------------
```
- 동일 INDENT인 경우 같은 DEPTH의 작업(EX. 3,4혹은 2, 5)
- 동일 INDENT인 경우 위의 작업이 먼저 실행됨
- 따라서 해당 SQL은 SOURCE테이블을 FULL SCAN후, SYS_C00326745인덱스를 NESTED LOOPS로 TARGET테이블과 조인한다.
<br><br>
  

# 4. 옵티마이저란?
- 실행할 SQL을 전달받았을 때, 가장 효율적인 방법으로 SQL을 수행할 최적의 처리 경로를 생성해주는 DBMS의 핵심 엔진
<br><br>
  
## 4-1. SQL 처리과정
![img.png](/assets/images/sql-optimizer.png)
<br>

## 4-2. 옵티마이저의 종류
### 4-2-1. 규칙기반 옵티마이저(= RBO, Rule Based Optimizer)
- oracle8 혹은 그 이전 버전의 기본 옵티마이저
- 15가지 정도의 우선순위를 정해놓고, 우선순위가 앞서는 방식을 채택하도록 함
- 10g이후 버전부터는 공식적으로 지원X
- **장점**
  - 우선순위가 정해져 있으므로 sql의 실행 방식 및 절차를 미리 예측할 수 있음
- **단점**
  - 힌트 사용 불가, 단 rbo환경에서 힌트를 사용하는 경우, 해당sql은 cbo를 사용하여 실행계획을 수립
  - hash join 사용 불가
<br><br>
    

### 4-2-2. 비용기반 옵티마이저(= CBO, Cost Based Optimizer)
- ORACLE을 제외한 DBMS는 CBO만 제공
- 다양한 통계정보(데이터의 건수, 데이터를 보유하고 있는 블록의 수, 데이터의 분포도, CPU, I/O 등)를 참고하여 비용을 계산해, 최소 비용의 실행계획을 수립
- 통계정보의 값이 정확하지 않은 경우 수립된 실행계획 또한 최적화된 결과라고 보기 어려움

```sql
-- 마지막 분석 시간 확인
SELECT TABLE_NAME, NUM_ROWS, LAST_ANALYZED
FROM USER_TABLES
ORDER BY LAST_ANALYZED DESC;

-- 전체 데이터 대상 수동 Anaylyze
-- 대량데이터의 경우 estimate를 통해 일부 데이터로도 anaylze가능
ANALYZE TABLE SOURCE COMPUTE STATISTICS;
```