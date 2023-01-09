---
layout: post
title:  "Mybatis insert 후 생성된 key값 가져오기"
date:   2022-12-29 00:01:01
categories: BACKEND
desc: Mybatis 관련 selectkey memo 복붙
tags: spring mybatis sql
cover: spring.png
---


**1. GeneratedKeys 사용**

AutoIncrement된 key를 자동으로 세팅해준다.

```xml
<insert id="insertStudents" useGeneratedKeys="true"
    keyProperty="id">
  insert into Students (name ,email)
  values (#{name},#{email})
</insert>
```

```java
insertStudents(student: Student);
```

**2. selectKey 사용**

별도의 selectkey 설정을 위한 쿼리를 수행한다.

```xml
<insert id="insertSurveyInfo" parameterType="Board">
    INSERT INTO board(boardID, title, content)
    VALUES(#{boarID}, #{title}, #{content})
    <selectKey resultType="int" keyProperty="iq" order="AFTER">
        SELECT LAST_INSERT_ID()
    </selectKey>        
</insert>
<!--
  이전에 값을 가져오려면 before로 작업
  <selectKey resultType="string" keyProperty="boardID" order="BEFORE">
      SELECT MAX(boardID)+1 FROM board        
  </selectKey>  
-->
```

[ref]: https://mybatis.org/mybatis-3/ko/sqlmap-xml.html