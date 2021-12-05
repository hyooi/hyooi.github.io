---
layout: post
title:  "oracle lob과 temp tablespace"
published: false
categories:
- Database
- 트러블슈팅
tags:
- jdbc
- oracle
---

### 1. 개요
oracle에는 blob과 clob타입이 존재한다.
select * from V$TEMPORARY_LOBS
https://docs.oracle.com/cd/B28359_01/java.111/b31224/oralob.htm#i1060097

https://github.com/spring-projects/spring-framework/issues/10666
https://github.com/spring-projects/spring-framework/issues/10877
ORA-1652: unable to extend temp segment by 128 in tablespace TEMP

```java
TemporaryLobCreator
@Override
public void close() {
    for (Blob blob : this.temporaryBlobs) {
    try {
    blob.free();
    }
    catch (SQLException ex) {
    logger.warn("Could not free BLOB", ex);
    }
    }
    for (Clob clob : this.temporaryClobs) {
    try {
    clob.free();
    }
    catch (SQLException ex) {
    logger.warn("Could not free CLOB", ex);
    }
    }
    }
```
<br/>
