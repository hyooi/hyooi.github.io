---
layout: post
title:  "oracle max(id)+1에서 발생한 duplication error"
categories:
- Database
- 트러블슈팅
tags:
- jdbc
- oracle
---

### 1. 개요
sql developer로 실행 시에는 **정상**이었던 insert query가, 어플리케이션을 통해 실행되면 에러가 발생했다.
```bash
Duplicate entry '0001' for key 'target_pk';
```

너무나 익숙한 에러메시지였다.

<br/>

### 2. 원인
insert sql에 문제가 있는 것이 확실해보였다.
아래는 간소화한 해당 sql이다.
```sql
INSERT INTO TARGET(ID, NAME)
VALUES(
    (SELECT MAX(ID) + 1 FROM TARGET)
    , ?
)
```

<var>SELECT MAX(ID) + 1 FROM TARGET</var>을 통해 최대 ID를 구하고 있었다.

이렇게 가져온 id가 중복이었고, 때문에 **duplication error가 발생**했던 것이다.
어플리케이션의 <ins>insert로직은 batch로 update</ins>하고 있는데, 이 부분이 문제인 것 같아 보였다.

<br/>

### 3. 테스트
#### 3.1. executeUpdate
varchar타입의 ID, NAME컬럼을 가진 테이블을 생성해 간단하게 테스트를 진행했다.

먼저 배치가 아닌 <var>executeUpdate</var>를 이용한 테스트케이스이다.
executeUpdate를 통해 TARGET테이블에 최대ID보다 1을 증가시켜 INSERT처리하고 있다. 

처리한 후에는 TARGET테이블의 전체 데이터를 출력했다.

```java
@Test
  void executeUpdate() throws SQLException {
      try (Connection conn = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:orcl","sa", "password")) {
        PreparedStatement ps = conn.prepareStatement("INSERT INTO TARGET(ID, NAME) VALUES((SELECT MAX(ID) + 1 FROM TARGET), ?)");
        for (int i = 0; i < 10; i++) {
          ps.setString(1, "test" + i);
          ps.executeUpdate();
        }

        ps = conn.prepareStatement("SELECT * FROM TARGET");
        ResultSet rs = ps.executeQuery();
        while (rs.next()) {
          System.out.println(rs.getString(1) + ": " + rs.getString(2));
        }
      }
    }
```

<br/>

의도한대로 id가 하나씩 증가한 것을 확인할 수 있었다.
```bash
1: INIT
2: test0
3: test1
4: test2
5: test3
6: test4
7: test5
8: test6
9: test7
10: test8
11: test9
```

<br/>

#### 3.2. executeBatch
이번에는 에러가 발생한 어플리케이션처럼, 각각의 update sql를 batch로 실행했다.

```java
@Test
  void executeBatch() throws SQLException {
    try (Connection conn = DriverManager.getConnection("jdbc:oracle:thin:@loclhost:1521:orcl", "sa", "password")) {
      PreparedStatement ps = conn.prepareStatement("INSERT INTO TARGET(ID, NAME) VALUES((SELECT MAX(ID) + 1 FROM TARGET), ?)");
      for (int i = 0; i < 10; i++) {
        ps.setString(1, "test" + i);
        ps.addBatch();
      }
      ps.executeBatch();

      ps = conn.prepareStatement("SELECT * FROM TARGET");
      ResultSet rs = ps.executeQuery();
      while (rs.next()) {
        System.out.println(rs.getString(1) + ": " + rs.getString(2));
      }
    }
  }
```

<br/>

직접 insert했던 데이터 외에 테스트코드를 통해 입력한 데이터의 id가 모두 2였다.

문제가 되었던 현상이 재연되는 것을 알 수 있었다.
해당 테이블 또한 id를 pk로 지정했다면 동일하게 <var>duplication error</var>가 발생했을 것이다.

```bash
1: INIT
2: test0
2: test1
2: test2
2: test3
2: test4
2: test5
2: test6
2: test7
2: test8
2: test9
```

<br/>

### 4. 해결
해당 sql은 executeUpdate대신 <var>executeUpdate</var>를 사용해 해결할 수 있었다.

executeBatch는 executeUpdate와는 달리, 성능을 위해 실제DB로의 접근 횟수를 줄인다.

건건이 insert처리를 하는 것이 아닌 <ins>일괄적으로 insert처리</ins>를 하는 것인데,
오라클은 이 경우 <var>select max(id)+1</var>를 모두 같은 값으로 처리하는 것으로 보인다.
> h2로도 동일하게 테스트했는데, h2에서는 순차적으로 id가 증가해 해당 에러가 발생하지 않았다.
